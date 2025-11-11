# Multi-Merchant E-Commerce & Inventory Management System

## 📋 Project Overview

A platform where **multiple merchants** can manage their products, inventory, and shops in one centralized system. Each merchant can have warehouses and multiple retail shops with real-time inventory tracking.

---

## 👥 Who Will Use This?

1. **System Administrators** - Manage the entire platform
2. **Merchants** - Business owners who sell products
3. **Shop Managers** - Manage individual retail locations

---

## 🎯 Main Features

### 1. User Management
**What it does:** Control who can access the system and what they can do

- **Admin**: Full control over everything
- **Merchant**: Manage their own products, warehouses, and shops

### 2. Product Catalog
**What it does:** Organize and categorize all products

**Categories & Subcategories:**
- Main categories (e.g., Electronics, Clothing, Food)
- Subcategories (e.g., Electronics → Laptops, Phones, Tablets)
- Easy navigation and product organization

**Merchant Products:**
- Each merchant can add their products
- Product information: Title, Photos, SKU code
- Track how many products each merchant has

**Product Variations:**
- Same product in different options
- Example: T-shirt in Small/Medium/Large, Red/Blue/Green
- Each variation has: Photos, SKU, Size, Color
- Track stock (astatka) for each variation

### 3. Warehouse Management
**What it does:** Track inventory in central warehouses

**Each Warehouse:**
- Has a name/title
- Belongs to a merchant
- Stores products before distribution

**Incoming Stock (Prixod):**
- Record when products arrive at warehouse
- Document which products and how many
- Example: "Received 10 units of Product A, Variation 1"

**Outgoing Stock (Uxod):**
- Record when products leave warehouse
- Track where they're going (to shops)
- Example: "Sent 9 units of Product A, Variation 2 to Shop #1"

### 4. Shop Management
**What it does:** Manage retail locations where customers buy products

**Multiple Shops per Merchant:**
- Each merchant can have multiple retail stores
- Shop information: Name, Logo, Contact details
- Track how many products each shop sells

**Shop Inventory:**
- Products available in each shop
- Real-time stock levels (astatka)

**Shop Pricing:**
- Set base price for each product variation
- Apply discounts:
  - Percentage discount (e.g., 20% off)
  - Fixed amount discount (e.g., $5 off)
- Final price calculated automatically:
  - `Final Price = Base Price - (Base Price × Discount %) - Fixed Discount`

**Shop Stock Movements:**
- **Incoming (Prixod):** Products received from warehouse
- **Outgoing (Uxod):** Products sold to customers or returned

---

## 🔄 How It Works

### Scenario: Merchant Adds New Product

1. **Merchant logs in** to the system
2. **Creates product** under appropriate category
   - Adds: Title, Photos, SKU
3. **Adds variations** (if product has options)
   - Example: Shirt - Small/Red, Medium/Blue, Large/Green
4. **Product appears** in their catalog

### Scenario: Restocking a Shop

1. **Products arrive** at warehouse
2. **Warehouse manager records** incoming stock (Prixod)
   - "Received 100 units of Product X"
3. **Transfer to shop** when needed
4. **Warehouse records** outgoing stock (Uxod)
   - "Sent 50 units to Shop A"
5. **Shop records** incoming stock (Prixod)
   - "Received 50 units"
6. **Stock levels update** automatically

### Scenario: Customer Purchase

1. **Customer buys product** at Shop A
2. **Shop records** outgoing stock (Uxod)
   - "Sold 1 unit of Product X, Red/Medium"
3. **Stock level decreases** automatically
4. **System shows** updated inventory

---

## 📊 Key Information Tracked

### For Merchants:
- Total products listed
- Total inventory across all warehouses
- Number of shops operated
- Product performance

### For Products:
- Current stock in each location
- Product variations and their availability
- Pricing at different shops
- Movement history (incoming/outgoing)

### For Warehouses:
- Total inventory stored
- Incoming shipments
- Outgoing transfers to shops

### For Shops:
- Available products and stock
- Pricing and discounts
- Sales and transfers
- Contact information for customers

---

## 🎨 Example Structure

```
Merchant: "Fashion Store LLC"
├── Products (100 products)
│   ├── T-Shirt Classic
│   │   ├── Variation: Small/Red (SKU: TS-001-S-R)
│   │   ├── Variation: Medium/Blue (SKU: TS-001-M-B)
│   │   └── Variation: Large/Green (SKU: TS-001-L-G)
│   └── Jeans Denim
│       ├── Variation: 30/Black
│       └── Variation: 32/Blue
│
├── Warehouse: "Main Warehouse"
│   ├── T-Shirt Small/Red: 50 units
│   ├── T-Shirt Medium/Blue: 30 units
│   └── Jeans 30/Black: 20 units
│
└── Shops (3 locations)
    ├── Shop 1: "Downtown Store"
    │   ├── T-Shirt Small/Red: 10 units ($20, 10% off = $18)
    │   └── Jeans 30/Black: 5 units ($50, $5 off = $45)
    │
    ├── Shop 2: "Mall Location"
    │   └── T-Shirt Medium/Blue: 15 units ($20, no discount)
    │
    └── Shop 3: "Airport Branch"
        └── T-Shirt Small/Red: 8 units ($25, 0% off)
```

---

## ✅ What Makes This System Useful?

1. **Centralized Management**
   - One place to manage everything
   - No need for multiple spreadsheets

2. **Real-Time Tracking**
   - Always know what's in stock
   - Prevent overselling

3. **Multi-Location Support**
   - Manage multiple shops easily
   - Transfer stock between locations

4. **Flexible Pricing**
   - Different prices for different shops
   - Easy discount management

5. **Inventory Control**
   - Track every product movement
   - Know where products are (warehouse or shop)

6. **Product Variations**
   - Handle complex products (sizes, colors, etc.)
   - Individual stock tracking per variation

---

## 🚀 Success Criteria

The system is successful when:

- ✅ Merchants can add and manage their products
- ✅ Warehouses can track incoming and outgoing stock
- ✅ Shops have accurate, real-time inventory
- ✅ Pricing and discounts calculate correctly
- ✅ All stock movements are documented
- ✅ Multiple merchants can work independently
- ✅ Admins can oversee the entire platform

---

## 📱 User Experience

### Merchant Dashboard Shows:
- Total products
- Total inventory value
- Low stock alerts
- Recent stock movements
- Shop performance

### Warehouse View Shows:
- Current inventory list
- Pending incoming shipments
- Pending outgoing transfers
- Stock level warnings

### Shop View Shows:
- Products available for sale
- Current prices (with discounts)
- Today's sales
- Stock that needs replenishment

---

## 🔑 Key Terms Explained

| Term | Meaning |
|------|---------|
| **Merchant** | Business owner selling products |
| **SKU** | Stock Keeping Unit - unique product code |
| **Variation** | Different version of same product (size, color, etc.) |
| **Astatka** | Stock/Inventory amount (quantity available) |
| **Prixod** | Incoming stock (products received) |
| **Uxod** | Outgoing stock (products leaving/sold) |
| **Base Price** | Starting price before discounts |
| **Final Price** | Price customer pays after discounts |

---

## 🎯 Implementation in Ucode

This system will be built using **Ucode** platform, which provides:

- **Database management** for all products and inventory
- **User roles** (Admin, Merchant) with appropriate permissions
- **REST API** for integration with other systems
- **Admin interface** for easy management
- **Real-time updates** for inventory tracking

**No coding required for basic setup** - Ucode handles the technical infrastructure!

---

## 📈 Future Enhancements (Optional)

- Customer accounts and order history
- Online shopping integration
- Barcode scanning for stock management
- Analytics and reports
- Notifications for low stock
- Multi-currency support
- Supplier management

---

**Platform:** Ucode (Low-Code Backend as a Service)  
**Complexity:** Medium  
**Timeline:** Configurable in Ucode with minimal custom development  
**Target Users:** Multi-merchant retail businesses

---

## Quick Summary

> **In one sentence:** A system where multiple merchants can manage their products across warehouses and retail shops, with real-time inventory tracking and flexible pricing.

> **Main benefit:** Complete visibility and control over inventory from warehouse to retail shop, preventing stockouts and enabling efficient multi-location management.

