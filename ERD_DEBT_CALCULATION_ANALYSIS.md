# ERD Analysis: Debt Calculation Requirements

## Requirements Summary

Based on our discussion, the system needs to handle:

1. **Partial product movement**: Products can be partially moved from containers to trucks (e.g., 14,000 kg out of 28,000 kg)
2. **Original container reference**: Products in trucks must maintain reference to their original container
3. **Independent status tracking**: Container portion and truck portion track status progression independently
4. **Per-container debt calculation**: Debt is calculated per container value (not invoice total), split across portions
5. **Stage-based debt**: Debt accrues when a STAGE completes (not per sub-status)
6. **Multiple containers in one truck**: One truck can have products from multiple containers/proformas/invoices

---

## Current ERD Analysis

### ✅ **What Works Well**

#### 1. **Status Tracking Structure** (`normal_erd.sql`)
```sql
Table proforma_statuses {
  main_status static_statuses  // The 5 fixed statuses
  percentage int               // Percentage per status
  proformas_id UUID
}

Table proforma_sub_statuses {
  status_templates_id UUID
  proforma_statuses_id UUID    // Links to parent status
  proformas_id UUID
}

Table batch_transport_units {
  proforma_statuses_id UUID     // Current status stage
  proforma_sub_statuses_id UUID // Current sub-status
  proformas_id UUID
  batches_id UUID
}
```
**✅ GOOD**: Each transport unit (container/truck) can track its own status progression independently.

#### 2. **Product Tracking** (`normal_erd.sql`)
```sql
Table batch_tu_contents {
  batch_transport_units_id UUID  // Current transport unit
  batch_items_id UUID            // Links to batch item
  products_id UUID
  weight FLOAT
  price FLOAT
  unit_price FLOAT
}
```
**✅ GOOD**: Products are linked to transport units and batch items.

#### 3. **Debt Tracking** (`normal_erd.sql`)
```sql
Table procument_debts {
  proformas_id UUID
  batches_id UUID
  partners_id UUID
  branches_id UUID
  amount FLOAT
  transaction_type procument_debt_transaction_type
  created_date timestamp
}
```
**✅ GOOD**: Debt table exists with necessary references.

---

## ❌ **Critical Missing Features**

### **Problem 1: No Original Container Reference**

**Current Schema:**
```sql
Table batch_tu_contents {
  batch_transport_units_id UUID  // Only current transport unit
  batch_items_id UUID
  products_id UUID
  weight FLOAT
  price FLOAT
}
```

**Missing:**
- ❌ No `original_batch_transport_units_id` to track which container the product came from
- ❌ No way to know if this product was moved from a container to a truck
- ❌ Cannot link truck products back to their original container for debt calculation

**Impact:**
- When products move from Container K1111 to Truck T-123, we lose the connection to K1111
- Cannot calculate debt based on original container's value and completed stages
- Cannot track what percentage of container was moved vs remained

### **Problem 2: No Partial Movement Tracking**

**Current Schema:**
- `batch_tu_contents` only tracks products in their current transport unit
- No way to track:
  - What percentage/amount of original container was moved
  - What percentage/amount remains in container
  - The relationship between container portion and truck portion

**Impact:**
- Cannot split debt calculation: "50% of container K1111 is in truck, 50% remains in container"
- Cannot track independent status progression for each portion

### **Problem 3: Debt Calculation at Wrong Level**

**Current Schema:**
```sql
Table procument_debts {
  proformas_id UUID
  batches_id UUID        // Invoice level
  amount FLOAT
}
```

**Missing:**
- ❌ No `batch_transport_units_id` to link debt to specific container
- ❌ Debt is tracked at batch/invoice level, not container level
- ❌ Cannot track debt per container portion (container vs truck)

**Current Implementation (from code):**
- Debt is calculated when any container reaches a stage → uses **entire invoice total**
- Should be: Debt calculated per container value when that container's stage completes

### **Problem 4: No Movement History**

**Missing Table:**
- No table to track product movements between transport units
- No record of:
  - When products were moved
  - From which container
  - To which truck
  - How much was moved
  - What stages were completed at time of movement

---

## 🔧 **Required Schema Changes**

### **Solution 1: Add Original Container Reference**

**Option A: Add field to `batch_tu_contents`**
```sql
Table batch_tu_contents {
  // ... existing fields ...
  original_batch_transport_units_id UUID [ref:> batch_transport_units.guid]  // NEW
  moved_from_container_at TIMESTAMP  // NEW
  original_weight FLOAT               // NEW: weight when moved
  original_price FLOAT                // NEW: price when moved
}
```

**Option B: Create movement tracking table**
```sql
Table batch_tu_content_movements {
  guid UUID [pk]
  source_batch_transport_units_id UUID [ref:> batch_transport_units.guid]
  target_batch_transport_units_id UUID [ref:> batch_transport_units.guid]
  batch_tu_contents_id UUID [ref:> batch_tu_contents.guid]
  moved_weight FLOAT
  moved_price FLOAT
  moved_at TIMESTAMP
  source_status_at_move UUID [ref:> proforma_statuses.guid]  // Status when moved
  source_sub_status_at_move UUID [ref:> proforma_sub_statuses.guid]
}
```

### **Solution 2: Track Container Portions**

**Add to `batch_transport_units`:**
```sql
Table batch_transport_units {
  // ... existing fields ...
  parent_batch_transport_units_id UUID [ref:> batch_transport_units.guid]  // NEW: if this is a truck, link to original container
  is_partial_movement BOOL          // NEW: true if products were partially moved
  original_total_weight FLOAT        // NEW: original container total weight
  current_weight FLOAT               // NEW: current weight in this unit
  moved_weight FLOAT                 // NEW: weight moved to other units (if container)
}
```

### **Solution 3: Fix Debt Calculation**

**Update `procument_debts`:**
```sql
Table procument_debts {
  // ... existing fields ...
  batch_transport_units_id UUID [ref:> batch_transport_units.guid]  // NEW: link to specific container/truck
  original_batch_transport_units_id UUID [ref:> batch_transport_units.guid]  // NEW: original container if moved
  container_portion_percentage FLOAT  // NEW: what % of container this debt represents
  proforma_statuses_id UUID [ref:> proforma_statuses.guid]  // NEW: which stage triggered this debt
  // Remove or keep batches_id depending on reporting needs
}
```

### **Solution 4: Track Status Per Portion**

**Create status tracking per portion:**
```sql
Table batch_transport_unit_status_history {
  guid UUID [pk]
  batch_transport_units_id UUID [ref:> batch_transport_units.guid]
  proforma_statuses_id UUID [ref:> proforma_statuses.guid]
  proforma_sub_statuses_id UUID [ref:> proforma_sub_statuses.guid]
  completed_at TIMESTAMP
  debt_accrued FLOAT
  is_locked BOOL  // true if this portion's status is frozen (products moved)
}
```

---

## 📊 **Recommended Approach**

### **Hybrid Solution: Track Original + Movement History**

1. **Add to `batch_tu_contents`:**
   ```sql
   original_batch_transport_units_id UUID  // Original container
   moved_at TIMESTAMP                      // When moved (NULL if never moved)
   ```

2. **Create movement tracking table:**
   ```sql
   Table batch_tu_content_movements {
     guid UUID [pk]
     source_batch_transport_units_id UUID
     target_batch_transport_units_id UUID
     batch_tu_contents_id UUID
     moved_weight FLOAT
     moved_price FLOAT
     moved_at TIMESTAMP
     source_completed_stages UUID[]  // Array of completed stage IDs at time of move
   }
   ```

3. **Update `procument_debts`:**
   ```sql
   batch_transport_units_id UUID      // Container/truck that triggered debt
   original_batch_transport_units_id UUID  // Original container (for reporting)
   proforma_statuses_id UUID          // Stage that triggered debt
   container_portion_value FLOAT       // Value of this portion
   ```

4. **Add status locking mechanism:**
   ```sql
   Table batch_transport_unit_status_locks {
     guid UUID [pk]
     batch_transport_units_id UUID
     locked_at TIMESTAMP
     locked_stages UUID[]  // Stages completed and locked
     locked_debt FLOAT     // Debt accrued up to lock point
   }
   ```

---

## ✅ **Can Current ERD Handle It?**

### **Answer: PARTIALLY - Needs Modifications**

**What Works:**
- ✅ Status tracking structure (proforma_statuses, proforma_sub_statuses)
- ✅ Transport unit tracking (batch_transport_units)
- ✅ Product tracking (batch_tu_contents)
- ✅ Debt table exists (procument_debts)

**What's Missing:**
- ❌ Original container reference in `batch_tu_contents`
- ❌ Movement history tracking
- ❌ Container portion tracking (partial movements)
- ❌ Debt linked to specific containers (currently at batch level)
- ❌ Status locking mechanism for moved portions

**Recommendation:**
The ERD has a good foundation but needs **4-5 additional fields/tables** to fully support the complex debt calculation requirements. The changes are **additive** (won't break existing functionality) and can be implemented incrementally.

---

## 🎯 **Implementation Priority**

1. **HIGH PRIORITY:**
   - Add `original_batch_transport_units_id` to `batch_tu_contents`
   - Add `batch_transport_units_id` to `procument_debts`
   - Add `proforma_statuses_id` to `procument_debts`

2. **MEDIUM PRIORITY:**
   - Create `batch_tu_content_movements` table
   - Add movement tracking fields

3. **LOW PRIORITY:**
   - Status locking mechanism (can be handled in application logic initially)
   - Detailed portion tracking (can calculate from movements)

