# Shopizer Codebase Analysis

## Project Overview

**Shopizer** is a Java-based open-source headless e-commerce platform providing REST APIs for building online stores.

- **Version**: 3.2.5 (README mentions 3.2.7)
- **Java Version**: 11+ (tested with Java 11, 17)
- **License**: Apache License 2.0
- **Framework**: Spring Boot 2.5.12

---

## Project Structure

Shopizer follows a **multi-module Maven architecture** with clear separation of concerns:

```
shopizer/
├── sm-core-model/          # Domain entities and data models
├── sm-core-modules/        # Core business modules (payment, shipping, etc.)
├── sm-core/                # Business logic and services
├── sm-shop-model/          # API DTOs and facade interfaces
└── sm-shop/                # REST API controllers and main application
```

### Module Breakdown

#### 1. **sm-core-model** (Domain Layer)
Contains JPA entities representing the business domain:
- `catalog/` - Products, categories, manufacturers
- `customer/` - Customer entities and attributes
- `order/` - Orders, order items, payments
- `merchant/` - Store configuration
- `user/` - Admin users and permissions
- `shoppingcart/` - Cart and cart items
- `shipping/` - Shipping configuration
- `tax/` - Tax classes and rates
- `content/` - CMS content
- `payments/` - Payment transactions
- `reference/` - Countries, zones, languages, currencies

#### 2. **sm-core-modules** (Integration Layer)
External integrations and modules:
- Payment gateways (PayPal, Stripe, Braintree)
- Shipping providers
- Email services
- File storage (AWS S3, Google Cloud Storage)

#### 3. **sm-core** (Service Layer)
Business logic and service implementations:
- Repository interfaces (Spring Data JPA)
- Service layer implementations
- Business rules and validations

#### 4. **sm-shop-model** (API Contract Layer)
API models and facade interfaces:
- REST API request/response DTOs
- Facade interfaces for controllers
- Mappers (using MapStruct)

#### 5. **sm-shop** (Presentation Layer)
REST API controllers and application entry point:
- REST API endpoints (v0, v1, v2)
- Security configuration
- Swagger documentation
- Main Spring Boot application

---

## Architecture

### Layered Architecture

```
┌─────────────────────────────────────┐
│   REST API Controllers (sm-shop)    │  ← HTTP/JSON
├─────────────────────────────────────┤
│   Facades (sm-shop-model)           │  ← Business orchestration
├─────────────────────────────────────┤
│   Services (sm-core)                │  ← Business logic
├─────────────────────────────────────┤
│   Repositories (sm-core)            │  ← Data access
├─────────────────────────────────────┤
│   Entities (sm-core-model)          │  ← Domain models
└─────────────────────────────────────┘
```

### Key Design Patterns
- **Facade Pattern**: Facades abstract complex service interactions
- **Repository Pattern**: Spring Data JPA repositories
- **DTO Pattern**: Separate API models from domain entities
- **Dependency Injection**: Spring IoC container

---

## Main Dependencies

### Core Framework
- **Spring Boot 2.5.12** - Application framework
- **Spring Data JPA** - Data persistence
- **Spring Security** - Authentication & authorization
- **Spring Boot Actuator** - Monitoring and health checks

### Database
- **H2 Database** - Default embedded database (development)
- **MySQL 8.0.21** - Production database (optional)
- **PostgreSQL 42.2.18** - Alternative database (optional)
- **Oracle 18.3.0.0** - Alternative database (optional)

### Payment Gateways
- **PayPal SDK 2.6.109**
- **Stripe Java 19.5.0**
- **Braintree Java 2.73.0**

### Cloud Storage
- **AWS S3 SDK 1.11.640** - Amazon S3 storage
- **Google Cloud Storage 1.74.0** - GCP storage

### Search & Caching
- **Elasticsearch 7.5.2** - Product search
- **Infinispan 9.4.18** - Distributed caching
- **Ehcache** - Local caching

### Business Rules
- **Drools 7.32.0** - Rules engine for pricing, promotions

### API Documentation
- **Swagger 2.9.2** (Springfox) - API documentation

### Security
- **JWT 0.8.0** - Token-based authentication
- **OWASP AntiSamy 1.6.7** - XSS protection
- **Passay 1.6.0** - Password validation

### Utilities
- **MapStruct 1.3.0** - Object mapping
- **Jackson 2.13.4** - JSON processing
- **Apache Commons** - Utilities (Lang3, Collections4, IO, Validator)
- **Guava 27.1** - Google utilities
- **MaxMind GeoIP2 2.7.0** - Geolocation

---

## How to Build

### Prerequisites
- Java 11 or higher
- Maven 3.6+ (or use included Maven wrapper)

### Build Commands

#### 1. Build entire project
```bash
cd shopizer
./mvnw clean install
```

#### 2. Run the application
```bash
cd sm-shop
./mvnw spring-boot:run
```

#### 3. Build Docker image
```bash
cd sm-shop
docker build -t shopizer:latest .
```

#### 4. Run with Docker
```bash
docker run -p 8080:8080 shopizerecomm/shopizer:latest
```

### Access Points
- **Swagger UI**: http://localhost:8080/swagger-ui.html
- **API Base**: http://localhost:8080/api/v1
- **Actuator**: http://localhost:8080/actuator

---

## API Endpoints Overview

All APIs are versioned under `/api/v1` (and some under `/api/v2`).

### Authentication & Security

#### User Authentication (Admin)
- `POST /api/v1/user/login` - Admin user login
- `POST /api/v1/user/password/reset/request` - Request password reset
- `GET /api/v1/user/{store}/reset/{token}` - Validate reset token

#### Customer Authentication
- `POST /api/v1/customer/register` - Customer registration
- `POST /api/v1/customer/login` - Customer login
- `POST /api/v1/customer/password/reset/request` - Request password reset
- `GET /api/v1/customer/{store}/reset/{token}` - Validate reset token

#### Security & Permissions
- `GET /api/v1/sec/private/{group}/permissions` - Get group permissions
- `GET /api/v1/sec/private/permissions` - List all permissions

---

### Store Management

#### Merchant Store
- `GET /api/v1/store/{code}` - Get store by code (public)
- `GET /api/v1/private/store/{code}` - Get store details (admin)
- `POST /api/v1/private/store` - Create store
- `PUT /api/v1/private/store/{code}` - Update store
- `DELETE /api/v1/private/store/{code}` - Delete store

#### Marketplace
- `GET /api/v1/private/marketplace/{store}` - Get marketplace config
- `POST /api/v1/store/signup` - Store signup

---

### Catalog Management

#### Products (v1)
- `POST /api/v1/private/product` - Create product
- `PUT /api/v1/private/product/{id}` - Update product
- `DELETE /api/v1/private/product/{id}` - Delete product
- `GET /api/v1/products` - List products (with filters)
- `GET /api/v1/product/{id}` - Get product by ID
- `GET /api/v1/product/{friendlyUrl}` - Get product by URL
- `GET /api/v1/private/product/unique` - Check product uniqueness
- `POST /api/v1/private/product/{id}/category/{categoryId}` - Add to category
- `DELETE /api/v1/private/product/{id}/category/{categoryId}` - Remove from category

#### Product Images
- `POST /api/v1/private/product/{id}/image` - Upload product image
- `DELETE /api/v1/private/product/image/{id}` - Delete product image

#### Product Inventory
- `POST /api/v1/private/product/{productId}/inventory` - Create inventory
- `PUT /api/v1/private/product/{productId}/inventory/{id}` - Update inventory
- `DELETE /api/v1/private/product/{productId}/inventory/{id}` - Delete inventory

#### Product Pricing
- `POST /api/v1/private/product/{sku}/price` - Create price
- `PUT /api/v1/private/product/{sku}/price/{id}` - Update price
- `DELETE /api/v1/private/product/{sku}/price/{id}` - Delete price

#### Product Reviews
- `POST /api/v1/product/{id}/review` - Create review
- `GET /api/v1/product/{id}/reviews` - List reviews

#### Product Types
- `GET /api/v1/private/product/types` - List product types
- `GET /api/v1/private/product/type/{id}` - Get product type

#### Product Options & Attributes
- `POST /api/v1/private/product/option` - Create product option
- `PUT /api/v1/private/product/option/{id}` - Update product option
- `DELETE /api/v1/private/product/option/{id}` - Delete product option
- `GET /api/v1/private/product/option/unique` - Check option uniqueness

#### Product Groups
- `POST /api/v1/private/products/group` - Create product group
- `GET /api/v1/private/product/groups` - List product groups

#### Product Variants (v2)
- `POST /api/v2/private/product/variant` - Create variant
- `GET /api/v2/private/product/{id}/variants` - List variants
- `POST /api/v2/private/product/variant/group` - Create variant group

#### Manufacturers
- `POST /api/v1/private/manufacturer` - Create manufacturer
- `PUT /api/v1/private/manufacturer/{id}` - Update manufacturer
- `DELETE /api/v1/private/manufacturer/{id}` - Delete manufacturer
- `GET /api/v1/manufacturer/{id}` - Get manufacturer
- `GET /api/v1/manufacturers` - List manufacturers

---

### Categories

- `GET /api/v1/category/{id}` - Get category by ID
- `GET /api/v1/category/{friendlyUrl}` - Get category by URL
- `POST /api/v1/private/category` - Create category
- `PUT /api/v1/private/category/{id}` - Update category
- `DELETE /api/v1/private/category/{id}` - Delete category
- `GET /api/v1/categories` - List categories (hierarchical)

---

### Shopping Cart

- `POST /api/v1/cart` - Create cart
- `PUT /api/v1/cart/{code}` - Update cart (add items)
- `GET /api/v1/cart/{code}` - Get cart by code
- `POST /api/v1/cart/{code}/multi` - Add multiple items
- `POST /api/v1/cart/{code}/promo/{promo}` - Apply promo code
- `DELETE /api/v1/cart/{code}/product/{sku}` - Remove item from cart
- `GET /api/v1/auth/customer/cart` - Get authenticated customer's cart

---

### Orders

#### Order Creation
- `POST /api/v1/cart/{code}/checkout` - Checkout (guest)
- `POST /api/v1/auth/cart/{code}/checkout` - Checkout (authenticated)

#### Order Management
- `GET /api/v1/private/orders` - List all orders (admin)
- `GET /api/v1/private/orders/{id}` - Get order by ID (admin)
- `GET /api/v1/auth/orders` - List customer orders (authenticated)
- `GET /api/v1/auth/orders/{id}` - Get customer order (authenticated)
- `GET /api/v1/private/orders/customers/{id}` - List orders by customer
- `PUT /api/v1/private/orders/{id}/status` - Update order status
- `PATCH /api/v1/private/orders/{id}/customer` - Update order customer

#### Order Payment
- `POST /api/v1/cart/{code}/payment/init` - Initialize payment (guest)
- `POST /api/v1/auth/cart/{code}/payment/init` - Initialize payment (authenticated)

#### Order Shipping
- `GET /api/v1/private/orders/{id}/shipping` - Get shipping info
- `POST /api/v1/private/orders/{id}/shipping` - Update shipping

---

### Customers

#### Customer Management
- `POST /api/v1/private/customer` - Create customer (admin)
- `PUT /api/v1/private/customer/{id}` - Update customer (admin)
- `DELETE /api/v1/private/customer/{id}` - Delete customer (admin)
- `GET /api/v1/private/customer/{id}` - Get customer (admin)
- `GET /api/v1/private/customers` - List customers (admin)

#### Customer Profile (Authenticated)
- `GET /api/v1/auth/customer/profile` - Get own profile
- `PATCH /api/v1/auth/customer` - Update own profile
- `PATCH /api/v1/auth/customer/address` - Update own address
- `DELETE /api/v1/auth/customer` - Delete own account

#### Customer Reviews
- `POST /api/v1/private/customers/{id}/reviews` - Create review (admin)
- `GET /api/v1/customers/{id}/reviews` - List customer reviews

#### Newsletter
- `POST /api/v1/newsletter` - Subscribe to newsletter
- `PUT /api/v1/newsletter/{email}` - Update subscription

---

### Shipping

#### Shipping Configuration
- `GET /api/v1/private/shipping/origin` - Get shipping origin
- `POST /api/v1/private/shipping/origin` - Set shipping origin
- `GET /api/v1/private/shipping/expedition` - Get shipping methods
- `GET /api/v1/shipping/country` - Get shipping countries

---

### Tax

#### Tax Rates
- `POST /api/v1/private/tax/rate` - Create tax rate
- `PUT /api/v1/private/tax/rate/{id}` - Update tax rate
- `DELETE /api/v1/private/tax/rate/{id}` - Delete tax rate
- `GET /api/v1/private/tax/rate/unique` - Check uniqueness

#### Tax Classes
- `POST /api/v1/private/tax/class` - Create tax class
- `PUT /api/v1/private/tax/class/{id}` - Update tax class
- `DELETE /api/v1/private/tax/class/{id}` - Delete tax class
- `GET /api/v1/private/tax/class/unique` - Check uniqueness

---

### Content Management

#### Content Pages
- `GET /api/v1/content/pages` - List content pages (public)
- `GET /api/v1/private/content/pages` - List content pages (admin)
- `GET /api/v1/content/summary` - Get content summary
- `POST /api/v1/private/content` - Create content
- `PUT /api/v1/private/content/{id}` - Update content
- `DELETE /api/v1/private/content/{id}` - Delete content

#### Content Administration
- `GET /api/v1/private/content/list` - List content items
- `GET /api/v1/private/content/folder` - List content folders
- `POST /api/v1/private/content/folder` - Create folder

---

### Search

- `GET /api/v1/search` - Search products
- `POST /api/v1/search` - Advanced search
- `GET /api/v1/private/search/index` - Reindex products (admin)

---

### System & Configuration

#### References
- `GET /api/v1/languages` - List languages
- `GET /api/v1/country` - List countries
- `GET /api/v1/zones` - List zones
- `GET /api/v1/currencies` - List currencies

#### Modules
- `GET /api/v1/private/modules/payment` - List payment modules
- `POST /api/v1/private/modules/payment` - Configure payment module
- `GET /api/v1/private/modules/shipping` - List shipping modules

#### Configurations
- `POST /api/v1/private/configurations/payment` - Save payment config
- `GET /api/v1/private/configurations/payment` - Get payment config

#### System Tools
- `POST /api/v1/private/search/tools/reindex` - Reindex search
- `GET /api/v1/private/cache` - Cache management

#### Contact
- `POST /api/v1/contact` - Send contact message

#### Opt-in
- `POST /api/v1/optin` - Opt-in to communications

---

### Users (Admin)

- `GET /api/v1/private/users/{id}` - Get user
- `POST /api/v1/private/user` - Create user
- `PUT /api/v1/private/user/{id}` - Update user
- `DELETE /api/v1/private/user/{id}` - Delete user
- `GET /api/v1/private/users` - List users

---

## API Access Patterns

### Public APIs
No authentication required:
- Product browsing
- Category listing
- Store information
- Content pages
- Search

### Authenticated Customer APIs (`/auth/*`)
Requires customer JWT token:
- Customer profile management
- Order history
- Shopping cart (customer-specific)
- Checkout

### Admin APIs (`/private/*`)
Requires admin JWT token:
- Store management
- Product management
- Order management
- Customer management
- System configuration

---

## Key Features

### E-commerce Core
- Multi-store support
- Multi-language & multi-currency
- Product catalog with variants
- Category hierarchy
- Shopping cart & checkout
- Order management
- Customer accounts
- Product reviews & ratings

### Payment Integration
- PayPal
- Stripe
- Braintree
- Extensible payment module system

### Shipping
- Multiple shipping methods
- Shipping zones
- Real-time shipping quotes

### Content Management
- CMS for pages and content
- File management
- Image handling

### Search
- Elasticsearch integration
- Full-text product search
- Faceted search

### Business Rules
- Drools rules engine
- Pricing rules
- Promotions & discounts

### Security
- JWT-based authentication
- Role-based access control (RBAC)
- XSS protection
- Password policies

---

## Configuration

### Database Configuration
Default: H2 embedded database (in-memory)

For MySQL, configure in `application.properties`:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/shopizer
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

### Email Configuration
Configure SMTP or AWS SES in application properties.

### File Storage
- Local filesystem (default)
- AWS S3
- Google Cloud Storage

---

## Testing

Run tests:
```bash
./mvnw test
```

---

## Documentation

- **Swagger UI**: http://localhost:8080/swagger-ui.html
- **Official Docs**: https://shopizer-ecommerce.github.io/documentation/
- **Slack Community**: https://shopizer.slack.com

---

## Summary

Shopizer is a well-architected, modular e-commerce platform built with Spring Boot. It provides:

- **Headless architecture** - REST APIs for any frontend
- **Multi-module design** - Clear separation of concerns
- **Extensible** - Plugin architecture for payments, shipping, etc.
- **Production-ready** - Security, caching, monitoring built-in
- **Open source** - Apache 2.0 license

The codebase follows enterprise Java best practices with layered architecture, dependency injection, and comprehensive API coverage for building modern e-commerce applications.
