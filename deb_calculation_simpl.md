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

**Process:**
1. User selects group product "Compensated" and provides price per kg (e.g., 3.90 USD/kg)
2. System looks up `product_group_items` for "Compensated" to get individual products and their weights
3. System expands group product into individual products immediately
4. All products in group use the same unit price (from user input)

**From `product_group_items` for "Compensated":**
- 46 STRIPLOIN: 16,800 kg (60% of 28,000)
- 67 CUBE ROLL: 5,600 kg (20% of 28,000)
- 41 TOPSIDE: 2,800 kg (10% of 28,000)
- 65 BLADE: 2,800 kg (10% of 28,000)

**Table: `batch_tu_contents`**
- **Create 4 individual product records** (one per product in group):

  1. **46 STRIPLOIN:**
     - Set `batch_transport_units_id` = K1111 GUID
     - Set `products_id` = 46 STRIPLOIN GUID
     - Set `group_products_id` = NULL (expanded, stored as individual)
     - Set `weight` = 16,800 kg
     - Set `unit_price` = 3.90 USD/kg (from user input)
     - Set `price` = 65,520 USD (16,800 × 3.90)
     - Set `original_batch_transport_units_id` = NULL (not moved yet)
     - Set `moved_at` = NULL

  2. **67 CUBE ROLL:**
     - Set `batch_transport_units_id` = K1111 GUID
     - Set `products_id` = 67 CUBE ROLL GUID
     - Set `group_products_id` = NULL
     - Set `weight` = 5,600 kg
     - Set `unit_price` = 3.90 USD/kg
     - Set `price` = 21,840 USD (5,600 × 3.90)
     - Set `original_batch_transport_units_id` = NULL
     - Set `moved_at` = NULL

  3. **41 TOPSIDE:**
     - Set `batch_transport_units_id` = K1111 GUID
     - Set `products_id` = 41 TOPSIDE GUID
     - Set `group_products_id` = NULL
     - Set `weight` = 2,800 kg
     - Set `unit_price` = 3.90 USD/kg
     - Set `price` = 10,920 USD (2,800 × 3.90)
     - Set `original_batch_transport_units_id` = NULL
     - Set `moved_at` = NULL

  4. **65 BLADE:**
     - Set `batch_transport_units_id` = K1111 GUID
     - Set `products_id` = 65 BLADE GUID
     - Set `group_products_id` = NULL
     - Set `weight` = 2,800 kg
     - Set `unit_price` = 3.90 USD/kg
     - Set `price` = 10,920 USD (2,800 × 3.90)
     - Set `original_batch_transport_units_id` = NULL
     - Set `moved_at` = NULL

**Total:** 28,000 kg, 109,200 USD (all products use same unit price: 3.90 USD/kg)

**Result:** Container K1111 contains 4 individual products (expanded from "Compensated" group)

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

### Step 6: Move 50% of Products to Truck

**Process:**
- Container K1111 already has 4 individual products (from Step 2)
- User moves 50% of each product's weight to truck
- Calculate 50% of each product in container:

**Current products in Container K1111:**
- 46 STRIPLOIN: 16,800 kg → Move 8,400 kg (50%)
- 67 CUBE ROLL: 5,600 kg → Move 2,800 kg (50%)
- 41 TOPSIDE: 2,800 kg → Move 1,400 kg (50%)
- 65 BLADE: 2,800 kg → Move 1,400 kg (50%)
- **Total to move: 14,000 kg (50% of 28,000)**

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
- **Update 4 existing records** in Container K1111 (reduce by 50%):
  1. **46 STRIPLOIN:**
     - Update weight: 16,800 kg → 8,400 kg (remaining 50%)
     - Update price: 65,520 USD → 32,760 USD (remaining 50%)
     - Keep `unit_price` = 3.90 USD/kg (unchanged)

  2. **67 CUBE ROLL:**
     - Update weight: 5,600 kg → 2,800 kg
     - Update price: 21,840 USD → 10,920 USD

  3. **41 TOPSIDE:**
     - Update weight: 2,800 kg → 1,400 kg
     - Update price: 10,920 USD → 5,460 USD

  4. **65 BLADE:**
     - Update weight: 2,800 kg → 1,400 kg
     - Update price: 10,920 USD → 5,460 USD

- **Create 4 new records** in Truck T-123 (moved products):
  1. **46 STRIPLOIN:**
     - Set `batch_transport_units_id` = T-123 GUID
     - Set `products_id` = 46 STRIPLOIN GUID
     - Set `group_products_id` = NULL
     - Set `weight` = 8,400 kg (moved amount)
     - Set `unit_price` = 3.90 USD/kg (same as original)
     - Set `price` = 32,760 USD (8,400 × 3.90)
     - Set `original_batch_transport_units_id` = K1111 GUID (original container)
     - Set `moved_at` = NOW()
     - Set `original_weight` = 8,400 kg
     - Set `original_price` = 32,760 USD

  2. **67 CUBE ROLL:**
     - Set `batch_transport_units_id` = T-123 GUID
     - Set `products_id` = 67 CUBE ROLL GUID
     - Set `weight` = 2,800 kg
     - Set `unit_price` = 3.90 USD/kg
     - Set `price` = 10,920 USD
     - Set `original_batch_transport_units_id` = K1111 GUID
     - Set `moved_at` = NOW()
     - Set `original_weight` = 2,800 kg
     - Set `original_price` = 10,920 USD

  3. **41 TOPSIDE:**
     - Set `batch_transport_units_id` = T-123 GUID
     - Set `products_id` = 41 TOPSIDE GUID
     - Set `weight` = 1,400 kg
     - Set `unit_price` = 3.90 USD/kg
     - Set `price` = 5,460 USD
     - Set `original_batch_transport_units_id` = K1111 GUID
     - Set `moved_at` = NOW()
     - Set `original_weight` = 1,400 kg
     - Set `original_price` = 5,460 USD

  4. **65 BLADE:**
     - Set `batch_transport_units_id` = T-123 GUID
     - Set `products_id` = 65 BLADE GUID
     - Set `weight` = 1,400 kg
     - Set `unit_price` = 3.90 USD/kg
     - Set `price` = 5,460 USD
     - Set `original_batch_transport_units_id` = K1111 GUID
     - Set `moved_at` = NOW()
     - Set `original_weight` = 1,400 kg
     - Set `original_price` = 5,460 USD

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
- Container K2222: Contains individual products (expanded from "4HQ" group), 28,000 kg, 117,600 USD, P1 complete
  - Products: 45 RUMP STEAK, 41 TOPSIDE, 44 SILVER SIDE, 42 KNUCKLE (from product_group_items)
- Container K3333: Contains "44 SILVER SIDE" product, 28,000 kg, 109,200 USD, P2 complete

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
- **Update existing records** in Container K2222 (reduce each product by 50%):
  - For each product in K2222, update:
    - `weight` = 50% of original
    - `price` = 50% of original
    - Keep `unit_price` unchanged

- **Update existing record** in Container K3333 (reduce by 75%):
  - 44 SILVER SIDE:
    - Update `weight`: 28,000 kg → 7,000 kg (remaining 25%)
    - Update `price`: 109,200 USD → 27,300 USD (remaining 25%)
    - Keep `unit_price` unchanged

- **Create new records** in Truck T-999:
  - **From K2222 (50% of each product):**
    - For each product in K2222, create new record:
      - Set `batch_transport_units_id` = T-999 GUID
      - Set `products_id` = Product GUID
      - Set `weight` = 50% of original weight
      - Set `unit_price` = Same as original
      - Set `price` = weight × unit_price
      - Set `original_batch_transport_units_id` = K2222 GUID
      - Set `moved_at` = NOW()
      - Set `original_weight` = Moved weight
      - Set `original_price` = Moved price
  
  - **From K3333 (75% of product):**
    - 44 SILVER SIDE:
      - Set `batch_transport_units_id` = T-999 GUID
      - Set `products_id` = 44 SILVER SIDE GUID
      - Set `weight` = 21,000 kg (75% of 28,000)
      - Set `unit_price` = Same as original
      - Set `price` = 81,900 USD (75% of 109,200)
      - Set `original_batch_transport_units_id` = K3333 GUID
      - Set `moved_at` = NOW()
      - Set `original_weight` = 21,000 kg
      - Set `original_price` = 81,900 USD

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
2. `batch_tu_contents` - Create individual product records
   - If group product selected: Look up `product_group_items`, expand into individual products
   - All products use same `unit_price` (from user input)
   - Each product gets its own record in `batch_tu_contents`

### When Container Completes a Stage:
1. `batch_transport_units` - Update status fields (`proforma_statuses_id`, `proforma_sub_statuses_id`)
2. `procument_debts` - Create debt record with container reference

### When Products are Moved (Partial):
1. `batch_transport_unit_status_locks` - Lock container status (record completed stages and locked debt)
2. `batch_transport_units` - Create truck record, update container weight/price totals
3. `batch_tu_contents` - Update container products (reduce weight/price), create new records in truck
4. `batch_tu_content_movements` - Record movement history for each product
5. `procument_debts` - Update existing debts (adjust `container_portion_value`), create locked debts for truck portion

### When Transport Unit Completes a Stage:
1. `batch_transport_units` - Update status fields
2. `procument_debts` - Create debt record
   - For container: `original_batch_transport_units_id` = itself
   - For truck: `original_batch_transport_units_id` = original container GUID

---

## Key Points

1. **Group Product Expansion**: Group products are expanded into individual products **immediately when added to container** (not when moved to truck). All products in group use same `unit_price` from user input.

2. **Original Container Reference**: Every product in a truck has `original_batch_transport_units_id` pointing back to source container

3. **Partial Movements**: Container and truck portions are tracked separately via weight/price in `batch_tu_contents`. Each product record is updated (container) or created (truck).

4. **Debt Calculation**: Each debt record links to specific transport unit and original container via `batch_transport_units_id` and `original_batch_transport_units_id`

5. **Status Independence**: Container and truck can progress through stages independently, each creating their own debt records

