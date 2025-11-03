# **Project Documentation: Single-Seller Marketplace Platform**

## 1. **Project Overview**

This project is a **single-seller e-commerce platform** (similar to Shopify) designed for small online stores. The platform supports:

* Multiple categories
* Products with **variants** (color, size, etc.)
* Different prices and stock per variant
* Multiple images per product and per variant
* Customer accounts, cart, and checkout
* Orders and payment tracking
* Product reviews

The goal is to provide a **simple, yet flexible system** for managing an online store.

---

## 2. **User Roles**

| Role                    | Description                                            | Permissions                                                                 |
| ----------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------- |
| **Admin / Store Owner** | Manages products, categories, orders, and site content | CRUD products, categories, variants, images; view orders and payments       |
| **Customer**            | Purchases products from the store                      | Browse products, add to cart, checkout, review products, view order history |

---

## 3. **Entities and Data Model**

### 3.1 **User**

Stores both admin and customer accounts.

| Field      | Type      | Description            |
| ---------- | --------- | ---------------------- |
| id         | int       | Primary key            |
| name       | string    | User full name         |
| email      | string    | Unique login email     |
| password   | string    | Hashed password        |
| role       | enum      | `admin` or `customer`  |
| address    | text      | Optional, for shipping |
| created_at | timestamp | Account creation date  |

---

### 3.2 **Category**

Organizes products into hierarchical groups.

| Field     | Type   | Description                 |
| --------- | ------ | --------------------------- |
| id        | int    | Primary key                 |
| name      | string | Category name               |
| slug      | string | URL-friendly name           |
| parent_id | int    | Optional, for subcategories |

---

### 3.3 **Product**

Base product entity (e.g., “T-Shirt”).

| Field       | Type      | Description             |
| ----------- | --------- | ----------------------- |
| id          | int       | Primary key             |
| name        | string    | Product name            |
| description | text      | Product description     |
| category_id | int       | Foreign key to Category |
| status      | enum      | `active` or `inactive`  |
| created_at  | timestamp |                         |
| updated_at  | timestamp |                         |

---

### 3.4 **Attribute**

Defines variant options (e.g., Color, Size).

| Field | Type   | Description    |
| ----- | ------ | -------------- |
| id    | int    | Primary key    |
| name  | string | Attribute name |

---

### 3.5 **AttributeValue**

Stores possible values for each attribute.

| Field        | Type   | Description                    |
| ------------ | ------ | ------------------------------ |
| id           | int    | Primary key                    |
| attribute_id | int    | Foreign key to Attribute       |
| value        | string | Attribute value (e.g., Red, M) |

---

### 3.6 **ProductVariant**

Represents a specific combination of attribute values.

| Field          | Type    | Description                   |
| -------------- | ------- | ----------------------------- |
| id             | int     | Primary key                   |
| product_id     | int     | Foreign key to Product        |
| sku            | string  | Unique identifier for variant |
| price          | decimal | Variant price                 |
| discount_price | decimal | Optional discounted price     |
| stock          | int     | Stock quantity                |

---

### 3.7 **VariantAttributeValue** (junction table)

Connects each variant to its attribute values.

| Field              | Type | Description                   |
| ------------------ | ---- | ----------------------------- |
| id                 | int  | Primary key                   |
| variant_id         | int  | Foreign key to ProductVariant |
| attribute_id       | int  | Foreign key to Attribute      |
| attribute_value_id | int  | Foreign key to AttributeValue |

---

### 3.8 **ProductImage**

Stores images for products or variants.

| Field      | Type    | Description                           |
| ---------- | ------- | ------------------------------------- |
| id         | int     | Primary key                           |
| product_id | int     | Foreign key to Product                |
| variant_id | int     | Optional, for variant-specific images |
| image_url  | string  | Image file URL                        |
| alt_text   | string  | Image description                     |
| is_main    | boolean | Flag main image                       |

---

### 3.9 **Cart**

Tracks products a customer wants to buy.

| Field      | Type | Description                   |
| ---------- | ---- | ----------------------------- |
| id         | int  | Primary key                   |
| user_id    | int  | Foreign key to User           |
| variant_id | int  | Foreign key to ProductVariant |
| quantity   | int  | Quantity selected             |

---

### 3.10 **Order**

Represents a completed checkout.

| Field            | Type      | Description                                            |
| ---------------- | --------- | ------------------------------------------------------ |
| id               | int       | Primary key                                            |
| user_id          | int       | Foreign key to User                                    |
| total_amount     | decimal   | Total order cost                                       |
| status           | enum      | `pending`, `paid`, `shipped`, `completed`, `cancelled` |
| shipping_address | text      | Delivery address                                       |
| created_at       | timestamp |                                                        |
| updated_at       | timestamp |                                                        |

---

### 3.11 **OrderItem**

Links variants to orders.

| Field             | Type    | Description                   |
| ----------------- | ------- | ----------------------------- |
| id                | int     | Primary key                   |
| order_id          | int     | Foreign key to Order          |
| variant_id        | int     | Foreign key to ProductVariant |
| quantity          | int     |                               |
| price_at_purchase | decimal |                               |

---

### 3.12 **Payment**

Tracks payment status.

| Field          | Type    | Description                          |
| -------------- | ------- | ------------------------------------ |
| id             | int     | Primary key                          |
| order_id       | int     | Foreign key to Order                 |
| amount         | decimal | Paid amount                          |
| method         | enum    | `card`, `paypal`, `cash_on_delivery` |
| status         | enum    | `pending`, `completed`, `failed`     |
| transaction_id | string  | Optional gateway ID                  |

---

### 3.13 **Review** (optional)

Stores customer feedback on products.

| Field      | Type      | Description            |
| ---------- | --------- | ---------------------- |
| id         | int       | Primary key            |
| user_id    | int       | Foreign key to User    |
| product_id | int       | Foreign key to Product |
| rating     | int       | 1–5 stars              |
| comment    | text      | Optional review text   |
| created_at | timestamp |                        |

---

## 4. **Key Features**

1. **Product Management**

   * Add/edit/delete products, variants, and images
   * Set price, stock, and attributes per variant
   * Assign images to specific variants or general product

2. **Category Management**

   * Create hierarchical categories
   * Assign products to categories

3. **Customer Features**

   * Registration/login
   * Browse products with filters (e.g., Color = Red, Size = M)
   * Add to cart, checkout
   * Order history and tracking
   * Leave product reviews

4. **Admin Dashboard**

   * View orders, payments, and stock
   * Update products, variants, and categories
   * Simple analytics (optional)

---

## 5. **Filtering Example**

To find products with **Color = Red** and **Size = M**, query:

```sql
SELECT DISTINCT p.*
FROM product p
JOIN product_variant v ON v.product_id = p.id
JOIN variant_attribute_value vav1 ON vav1.variant_id = v.id
JOIN attribute_value av1 ON av1.id = vav1.attribute_value_id
JOIN variant_attribute_value vav2 ON vav2.variant_id = v.id
JOIN attribute_value av2 ON av2.id = vav2.attribute_value_id
WHERE av1.value = 'Red'
  AND av2.value = 'M';
```

---

## 6. **Benefits of This Design**

* Fully normalized for fast filtering and queries
* Supports multiple images per variant
* Flexible attribute system (add new colors, sizes, materials easily)
* SKU system allows inventory tracking
* Ready for expansion to coupons, shipping, or analytics
