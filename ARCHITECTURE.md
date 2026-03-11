# Shopizer Architecture Diagram

## High-Level System Architecture

```mermaid
graph TB
    subgraph Clients["Clients"]
        BROWSER[Browser / React Shop]
        ADMIN[Admin UI]
        MOBILE[Mobile App]
    end

    subgraph Shopizer["Shopizer Backend (Spring Boot)"]
        subgraph sm-shop["sm-shop (Presentation Layer)"]
            FILTER[Filters\nCorsFilter · XssFilter]
            SEC[Security\nJWT · Spring Security]
            API_V1[REST API v1\n/api/v1/...]
            API_V2[REST API v2\n/api/v2/...]
            SWAGGER[Swagger UI\n:8080/swagger-ui.html]
        end

        subgraph sm-shop-model["sm-shop-model (API Contract Layer)"]
            FACADES[Facade Interfaces\nProductFacade · OrderFacade\nCustomerFacade · CategoryFacade\nShoppingCartFacade · StoreFacade\nShippingFacade · TaxFacade\nContentFacade · UserFacade]
            DTOS[API DTOs\nReadable* · Persistable*]
        end

        subgraph sm-core["sm-core (Business Layer)"]
            SERVICES[Business Services\nProductService · OrderService\nCustomerService · CategoryService\nShoppingCartService · MerchantService\nShippingService · TaxService\nContentService · UserService]
            REPOS[Repositories\nSpring Data JPA]
            MODULES[Business Modules\nPayment · Shipping · Email\nSearch · Storage · Rules]
        end

        subgraph sm-core-model["sm-core-model (Domain Layer)"]
            ENTITIES[JPA Entities\nProduct · Order · Customer\nCategory · MerchantStore\nShoppingCart · User · Tax\nContent · Shipping]
        end

        subgraph sm-core-modules["sm-core-modules (Integration Layer)"]
            PAYMENT_MOD[Payment Modules\nPayPal · Stripe · Braintree]
            SHIPPING_MOD[Shipping Modules\nCanada Post · Custom]
            STORAGE_MOD[Storage Modules\nLocal · AWS S3 · GCS]
            EMAIL_MOD[Email Modules\nSMTP · AWS SES]
            SEARCH_MOD[Search Module\nElasticsearch]
        end
    end

    subgraph Infrastructure["Infrastructure"]
        DB[(Database\nH2 / MySQL\nPostgreSQL)]
        CACHE[(Cache\nInfinispan\nEhcache)]
        ES[(Elasticsearch)]
        S3[(AWS S3 /\nGCS Bucket)]
        SMTP_SRV[SMTP /\nAWS SES]
    end

    Clients -->|HTTP/JSON| FILTER
    FILTER --> SEC
    SEC --> API_V1
    SEC --> API_V2
    API_V1 --> FACADES
    API_V2 --> FACADES
    FACADES --> DTOS
    FACADES --> SERVICES
    SERVICES --> REPOS
    SERVICES --> MODULES
    REPOS --> ENTITIES
    ENTITIES --> DB
    SERVICES --> CACHE
    MODULES --> PAYMENT_MOD
    MODULES --> SHIPPING_MOD
    MODULES --> STORAGE_MOD
    MODULES --> EMAIL_MOD
    MODULES --> SEARCH_MOD
    SEARCH_MOD --> ES
    STORAGE_MOD --> S3
    EMAIL_MOD --> SMTP_SRV
```

---

## Module Dependency Graph

```mermaid
graph LR
    SM_SHOP[sm-shop\nMain App] --> SM_SHOP_MODEL[sm-shop-model\nAPI Contracts]
    SM_SHOP --> SM_CORE[sm-core\nBusiness Logic]
    SM_SHOP --> SM_CORE_MODEL[sm-core-model\nDomain Entities]
    SM_SHOP_MODEL --> SM_CORE_MODEL
    SM_CORE --> SM_CORE_MODEL
    SM_CORE --> SM_CORE_MODULES[sm-core-modules\nIntegrations]
    SM_CORE_MODULES --> SM_CORE_MODEL
```

---

## Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant Filter as XSS/CORS Filter
    participant JWT as JWT Auth Filter
    participant API as REST Controller
    participant Facade
    participant Service
    participant Repo as Repository
    participant DB

    Client->>Filter: HTTP Request
    Filter->>JWT: Sanitized Request
    JWT->>API: Authenticated Request
    API->>Facade: Call Facade Method
    Facade->>Service: Business Logic
    Service->>Repo: Data Access
    Repo->>DB: SQL Query
    DB-->>Repo: Result Set
    Repo-->>Service: Domain Entity
    Service-->>Facade: Processed Entity
    Facade-->>API: DTO (Readable*)
    API-->>Client: JSON Response
```

---

## Security Architecture

```mermaid
graph TD
    REQ[Incoming Request] --> CORS[CorsFilter]
    CORS --> XSS[XssFilter]
    XSS --> JWT_FILTER[AuthenticationTokenFilter\nJWT Validation]

    JWT_FILTER --> PUBLIC{Public\nEndpoint?}
    PUBLIC -->|Yes| CONTROLLER[REST Controller]
    PUBLIC -->|No| AUTH{Authenticated?}

    AUTH -->|No| REJECT[401 Unauthorized]
    AUTH -->|Yes| ROLE{Role Check}

    ROLE -->|/private/* → ADMIN| ADMIN_CHECK{Is Admin?}
    ROLE -->|/auth/* → CUSTOMER| CUST_CHECK{Is Customer?}
    ROLE -->|Public| CONTROLLER

    ADMIN_CHECK -->|Yes| CONTROLLER
    ADMIN_CHECK -->|No| FORBIDDEN[403 Forbidden]
    CUST_CHECK -->|Yes| CONTROLLER
    CUST_CHECK -->|No| FORBIDDEN
```

---

## Domain Model

```mermaid
erDiagram
    MerchantStore ||--o{ Product : "has"
    MerchantStore ||--o{ Category : "has"
    MerchantStore ||--o{ Customer : "has"
    MerchantStore ||--o{ Order : "has"
    MerchantStore ||--o{ User : "has"

    Product ||--o{ ProductImage : "has"
    Product ||--o{ ProductAttribute : "has"
    Product ||--o{ ProductInventory : "has"
    Product }o--o{ Category : "belongs to"
    Product }o--|| Manufacturer : "made by"

    ShoppingCart ||--o{ ShoppingCartItem : "contains"
    ShoppingCartItem }o--|| Product : "references"
    ShoppingCart }o--|| Customer : "owned by"

    Order ||--o{ OrderProduct : "contains"
    Order }o--|| Customer : "placed by"
    Order ||--|| Payment : "paid via"
    Order ||--o{ OrderStatusHistory : "has"

    Customer ||--|| Billing : "has"
    Customer ||--|| Delivery : "has"
    Customer ||--o{ CustomerReview : "writes"
```

---

## API Structure

```mermaid
graph TD
    BASE["/api/v1"] --> STORE["/store\nMerchant Store"]
    BASE --> CATALOG["/products · /product\nCatalog"]
    BASE --> CATEGORY["/category · /categories\nCategories"]
    BASE --> CART["/cart\nShopping Cart"]
    BASE --> ORDER["/cart/{code}/checkout\nOrders"]
    BASE --> CUSTOMER["/customer\nCustomers"]
    BASE --> USER["/user\nAdmin Users"]
    BASE --> SHIPPING["/shipping\nShipping"]
    BASE --> TAX["/tax\nTax"]
    BASE --> CONTENT["/content\nCMS"]
    BASE --> SEARCH["/search\nSearch"]
    BASE --> REF["/languages · /country · /zones\nReferences"]
    BASE --> SYS["/modules · /configurations\nSystem"]

    CATALOG --> PUB_CAT["Public\nGET /products\nGET /product/{id}"]
    CATALOG --> PRIV_CAT["Admin /private\nPOST /product\nPUT /product/{id}\nDELETE /product/{id}"]

    ORDER --> GUEST_CO["Guest\nPOST /cart/{code}/checkout"]
    ORDER --> AUTH_CO["Authenticated\nPOST /auth/cart/{code}/checkout"]

    BASE2["/api/v2"] --> VARIANTS["/product/variant\nProduct Variants"]
    BASE2 --> VARIANT_GRP["/product/variant/group\nVariant Groups"]
```

---

## Infrastructure & Integrations

```mermaid
graph LR
    subgraph App["Shopizer App"]
        CORE[sm-core-modules]
    end

    subgraph Payments["Payment Gateways"]
        PP[PayPal SDK]
        ST[Stripe]
        BT[Braintree]
    end

    subgraph Storage["File Storage"]
        LOCAL[Local Filesystem]
        S3[AWS S3]
        GCS[Google Cloud Storage]
    end

    subgraph Email["Email Services"]
        SMTP[SMTP Server]
        SES[AWS SES]
    end

    subgraph Search["Search"]
        ES[Elasticsearch 7.5]
    end

    subgraph DB["Databases"]
        H2[H2 Embedded\nDev/Default]
        MYSQL[MySQL 8]
        PG[PostgreSQL]
    end

    subgraph Cache["Caching"]
        INF[Infinispan\nDistributed Cache]
        EHC[Ehcache\nLocal Cache]
    end

    CORE --> PP
    CORE --> ST
    CORE --> BT
    CORE --> LOCAL
    CORE --> S3
    CORE --> GCS
    CORE --> SMTP
    CORE --> SES
    CORE --> ES
    App --> H2
    App --> MYSQL
    App --> PG
    App --> INF
    App --> EHC
```
