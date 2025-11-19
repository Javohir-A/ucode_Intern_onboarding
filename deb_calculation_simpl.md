# Debt Calculation Flow - Table Operations

## Key Business Rules

1. **Container holds ONE group product OR individual products** (not mixed)
2. **When moving to truck**: Group products are **expanded** into individual products
3. **Partial movement**: Products can be partially moved (e.g., 14,000 kg out of 28,000 kg)
4. **Debt calculation**: Per container value, when STAGE completes (not sub-status)
5. **Status tracking**: Container portion and truck portion track independently

---

## Required Schema Changes

### New Fields in `batch_tu_contents`
- `original_batch_transport_units_id` - Links to original container (NULL if never moved)
- `moved_at` - Timestamp when product was moved (NULL if never moved)
- `original_weight` - Weight when moved (for tracking)
- `original_price` - Price when moved (for tracking)

### New Fields in `procument_debts`
- `batch_transport_units_id` - Which container/truck triggered this debt
- `original_batch_transport_units_id` - Original container (for reporting)
- `proforma_statuses_id` - Which stage triggered this debt
- `container_portion_value` - Value of this portion (for partial movements)

### New Table: `batch_tu_content_movements`
- Tracks when products are moved between transport units
- Records source/target, weight, price, and completed stages at time of move

### New Table: `batch_transport_unit_status_locks`
- Locks container status when products are moved
- Records completed stages and locked debt amount

---

## Scenario 1: Container Created with Group Product

### Step 1: Create Container

**Table: `batch_transport_units`**
- Create new record for Container K1111
- Set `type` = 'container'
- Set `proforma_statuses_id` = NULL (no stage completed yet)
- Set `proforma_sub_statuses_id` = NULL
- Set `total_weight` = 28,000 kg
- Set `total_price` = 109,200 USD
- Link to `batches_id` (Invoice) and `proformas_id`

**Result:** Container K1111 exists, no status, no debt

---

### Step 2: Add Group Product to Container

**Table: `batch_tu_contents`**
- Create new record
- Set `batch_transport_units_id` = K1111 GUID
- Set `group_products_id` = "Compensated" GUID
- Set `products_id` = NULL (it's a group product, not individual)
- Set `weight` = 28,000 kg
- Set `price` = 109,200 USD
- Set `original_batch_transport_units_id` = NULL (not moved yet)
- Set `moved_at` = NULL

**Result:** Container K1111 contains group product "Compensated"

---

## Scenario 2: Container Progresses Through Stages

### Step 3: Container Completes Stage P1

**Table: `batch_transport_units`**
- Update Container K1111 record
- Set `proforma_statuses_id` = P1 stage GUID
- Set `proforma_sub_statuses_id` = Last P1 sub-status GUID

**Table: `procument_debts`**
- Create new debt record
- Set `batch_transport_units_id` = K1111 GUID (container that triggered debt)
- Set `original_batch_transport_units_id` = K1111 GUID (same, not moved yet)
- Set `proforma_statuses_id` = P1 stage GUID
- Set `amount` = 21,840 USD (20% of 109,200)
- Set `container_portion_value` = 109,200 USD (100% of container)

**Result:** Container K1111: Stage P1 complete, Debt = 21,840 USD (20%)

---

### Step 4: Container Completes Stage P2

**Table: `batch_transport_units`**
- Update Container K1111 record
- Set `proforma_statuses_id` = P2 stage GUID
- Set `proforma_sub_statuses_id` = Last P2 sub-status GUID

**Table: `procument_debts`**
- Create new debt record
- Set `batch_transport_units_id` = K1111 GUID
- Set `original_batch_transport_units_id` = K1111 GUID
- Set `proforma_statuses_id` = P2 stage GUID
- Set `amount` = 21,840 USD (20% of 109,200)
- Set `container_portion_value` = 109,200 USD

**Result:** Container K1111: Stage P2 complete, Total Debt = 43,680 USD (40%)

---

## Scenario 3: Partial Movement - Container to Truck

### Step 5: Lock Container Status (Before Movement)

**Table: `batch_transport_unit_status_locks`**
- Create new lock record
- Set `batch_transport_units_id` = K1111 GUID
- Set `locked_stages` = [P1 GUID, P2 GUID] (completed stages)
- Set `locked_debt` = 43,680 USD (total debt accrued)

**Result:** Container K1111 status is locked at P2 with 43,680 USD debt

---

### Step 6: Expand Group Product and Move 50% to Truck

**Expand Group Product:**
- Look up `product_group_items` for "Compensated"
- Calculate 50% of each product:
  - 46 STRIPLOIN: 8,400 kg (50% of 16,800)
  - 67 CUBE ROLL: 2,800 kg (50% of 5,600)
  - 41 TOPSIDE: 1,400 kg (50% of 2,800)
  - 65 BLADE: 1,400 kg (50% of 2,800)
- Total: 14,000 kg (50% of 28,000)

**Table: `batch_transport_units`**
- **Create new record** for Truck T-123
  - Set `type` = 'truck'
  - Set `vehicle_number` = '00 123 000'
  - Set `total_weight` = 14,000 kg
  - Set `total_price` = 54,600 USD (50% of 109,200)
  - Set `proforma_statuses_id` = NULL (starts fresh)
  - Set `proforma_sub_statuses_id` = NULL

- **Update Container K1111 record**
  - Set `total_weight` = 14,000 kg (remaining 50%)
  - Set `total_price` = 54,600 USD (remaining 50%)

**Table: `batch_tu_contents`**
- **Update Container K1111 record** (group product)
  - Set `weight` = 14,000 kg (remaining)
  - Set `price` = 54,600 USD (remaining)

- **Create 4 new records** in Truck T-123 (expanded products):
  1. 46 STRIPLOIN
     - Set `batch_transport_units_id` = T-123 GUID
     - Set `products_id` = 46 STRIPLOIN GUID
     - Set `group_products_id` = NULL (expanded, no longer group)
     - Set `weight` = 8,400 kg
     - Set `price` = 32,760 USD
     - Set `original_batch_transport_units_id` = K1111 GUID (original container)
     - Set `moved_at` = NOW()
     - Set `original_weight` = 8,400 kg
     - Set `original_price` = 32,760 USD

  2. 67 CUBE ROLL (same pattern)
  3. 41 TOPSIDE (same pattern)
  4. 65 BLADE (same pattern)

**Table: `batch_tu_content_movements`**
- Create 4 movement records (one per product)
- Each record links source (K1111) to target (T-123)
- Records `moved_weight`, `moved_price`
- Records `source_completed_stages` = [P1 GUID, P2 GUID]

**Table: `procument_debts`**
- **Update existing P1 debt record**
  - Set `container_portion_value` = 54,600 USD (50% of container)
  - Set `amount` = 10,920 USD (20% of 54,600)

- **Update existing P2 debt record**
  - Set `container_portion_value` = 54,600 USD
  - Set `amount` = 10,920 USD

- **Create 2 new locked debt records** for truck portion:
  1. P1 debt for truck (locked)
     - Set `batch_transport_units_id` = T-123 GUID
     - Set `original_batch_transport_units_id` = K1111 GUID
     - Set `amount` = 10,920 USD (20% of 54,600)
     - Set `container_portion_value` = 54,600 USD

  2. P2 debt for truck (locked) - same pattern

**Result:**
- Container K1111: 14,000 kg remaining, Status locked at P2, Debt = 21,840 USD (for remaining portion)
- Truck T-123: 14,000 kg, Status: None, Debt = 21,840 USD (locked from P1+P2)

---

## Scenario 4: Independent Status Progression

### Step 7: Container Completes Stage P3

**Table: `batch_transport_units`**
- Update Container K1111 record
- Set `proforma_statuses_id` = P3 stage GUID
- Set `proforma_sub_statuses_id` = Last P3 sub-status GUID

**Table: `procument_debts`**
- Create new debt record
- Set `batch_transport_units_id` = K1111 GUID (container)
- Set `original_batch_transport_units_id` = K1111 GUID
- Set `proforma_statuses_id` = P3 stage GUID
- Set `amount` = 10,920 USD (20% of 54,600 - container portion)
- Set `container_portion_value` = 54,600 USD

**Result:** Container K1111: Stage P3 complete, Debt = 32,760 USD (60% of container portion)

---

### Step 8: Truck Completes Stage P3

**Table: `batch_transport_units`**
- Update Truck T-123 record
- Set `proforma_statuses_id` = P3 stage GUID
- Set `proforma_sub_statuses_id` = Last P3 sub-status GUID

**Table: `procument_debts`**
- Create new debt record
- Set `batch_transport_units_id` = T-123 GUID (truck)
- Set `original_batch_transport_units_id` = K1111 GUID (original container)
- Set `proforma_statuses_id` = P3 stage GUID
- Set `amount` = 10,920 USD (20% of 54,600 - truck portion)
- Set `container_portion_value` = 54,600 USD

**Result:**
- Container K1111: Stage P3 complete, Debt = 32,760 USD
- Truck T-123: Stage P3 complete, Debt = 32,760 USD
- **Total for K1111 (both portions):** 65,520 USD (60% of original 109,200)

---

## Scenario 5: Multiple Containers → One Truck

### Setup
- Container K2222: "4HQ" group product, 28,000 kg, 117,600 USD, P1 complete
- Container K3333: "44 SILVER SIDE" product, 28,000 kg, 109,200 USD, P2 complete

### Step 9: Move Products from Multiple Containers to One Truck

**User moves:**
- 50% from K2222 (14,000 kg = 58,800 USD) → Truck T-999
- 75% from K3333 (21,000 kg = 81,900 USD) → Truck T-999

**Table: `batch_transport_units`**
- **Create new record** for Truck T-999
  - Set `type` = 'truck'
  - Set `total_weight` = 35,000 kg (14,000 + 21,000)
  - Set `total_price` = 140,700 USD (58,800 + 81,900)

- **Update Container K2222 record**
  - Set `total_weight` = 14,000 kg (remaining 50%)
  - Set `total_price` = 58,800 USD (remaining 50%)

- **Update Container K3333 record**
  - Set `total_weight` = 7,000 kg (remaining 25%)
  - Set `total_price` = 27,300 USD (remaining 25%)

**Table: `batch_tu_contents`**
- **Update Container K2222 record** (group product)
  - Set `weight` = 14,000 kg
  - Set `price` = 58,800 USD

- **Update Container K3333 record** (product)
  - Set `weight` = 7,000 kg
  - Set `price` = 27,300 USD

- **Create multiple new records** in Truck T-999:
  - Expand K2222's "4HQ" group into individual products (4 products)
  - Each product has:
    - `batch_transport_units_id` = T-999 GUID
    - `products_id` = Individual product GUID
    - `group_products_id` = NULL (expanded)
    - `original_batch_transport_units_id` = K2222 GUID
    - `moved_at` = NOW()
  
  - Add K3333's product:
    - `batch_transport_units_id` = T-999 GUID
    - `products_id` = 44 SILVER SIDE GUID
    - `original_batch_transport_units_id` = K3333 GUID
    - `moved_at` = NOW()

**Table: `batch_tu_content_movements`**
- Create movement records for each product moved
- Link source containers (K2222, K3333) to target truck (T-999)
- Record completed stages at time of move

**Table: `batch_transport_unit_status_locks`**
- **Create lock for K2222**
  - Set `locked_stages` = [P1 GUID]
  - Set `locked_debt` = 11,760 USD (20% of 58,800 - 50% portion)

- **Create lock for K3333**
  - Set `locked_stages` = [P1 GUID, P2 GUID]
  - Set `locked_debt` = 32,760 USD (40% of 81,900 - 75% portion)

**Table: `procument_debts`**
- **Update existing debt records** for K2222 and K3333 to reflect new portion values
- **Create locked debt records** for truck portion from K2222:
  - P1 debt: 11,760 USD (20% of 58,800)
  - `original_batch_transport_units_id` = K2222 GUID

- **Create locked debt records** for truck portion from K3333:
  - P1 debt: 16,380 USD (20% of 81,900)
  - P2 debt: 16,380 USD (20% of 81,900)
  - `original_batch_transport_units_id` = K3333 GUID

**Result:**
- Container K2222: 14,000 kg remaining, Status locked at P1
- Container K3333: 7,000 kg remaining, Status locked at P2
- Truck T-999: 35,000 kg from 2 containers, Status: None, Debt: 44,520 USD (locked)

---

### Step 10: Truck T-999 Completes Stage P3

**Table: `batch_transport_units`**
- Update Truck T-999 record
- Set `proforma_statuses_id` = P3 stage GUID

**Table: `procument_debts`**
- **Create debt for K2222 portion** (50% of K2222 in truck)
  - Set `batch_transport_units_id` = T-999 GUID
  - Set `original_batch_transport_units_id` = K2222 GUID
  - Set `amount` = 11,760 USD (20% of 58,800)
  - Set `container_portion_value` = 58,800 USD

- **Create debt for K3333 portion** (75% of K3333 in truck)
  - Set `batch_transport_units_id` = T-999 GUID
  - Set `original_batch_transport_units_id` = K3333 GUID
  - Set `amount` = 16,380 USD (20% of 81,900)
  - Set `container_portion_value` = 81,900 USD

**Result:**
- Truck T-999: Stage P3 complete
- Total Debt for T-999:
  - From K2222: 23,520 USD (P1 locked + P3)
  - From K3333: 49,140 USD (P1+P2 locked + P3)
  - **Total: 72,660 USD**

---

## Summary Table Operations

### When Container is Created:
1. `batch_transport_units` - Create container record
2. `batch_tu_contents` - Create group product or product record

### When Container Completes a Stage:
1. `batch_transport_units` - Update status fields
2. `procument_debts` - Create debt record with container reference

### When Products are Moved (Partial):
1. `batch_transport_unit_status_locks` - Lock container status
2. `batch_transport_units` - Create truck, update container weight/price
3. `batch_tu_contents` - Update container, create expanded products in truck
4. `batch_tu_content_movements` - Record movement history
5. `procument_debts` - Update existing debts, create locked debts for truck portion

### When Transport Unit Completes a Stage:
1. `batch_transport_units` - Update status fields
2. `procument_debts` - Create debt record
   - For container: `original_batch_transport_units_id` = itself
   - For truck: `original_batch_transport_units_id` = original container GUID

---

## Key Points

1. **Original Container Reference**: Every product in a truck has `original_batch_transport_units_id` pointing back to source container
2. **Group Product Expansion**: When moved to truck, group products become individual products in `batch_tu_contents`
3. **Partial Movements**: Container and truck portions are tracked separately via weight/price in `batch_tu_contents`
4. **Debt Calculation**: Each debt record links to specific transport unit and original container via `batch_transport_units_id` and `original_batch_transport_units_id`
5. **Status Independence**: Container and truck can progress through stages independently, each creating their own debt records

