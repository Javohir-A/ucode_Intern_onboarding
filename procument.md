# Status Hierarchy and Debt Calculation Example

## Glossary

| Abbreviation | Full Name | Description |
|--------------|-----------|-------------|
| P | Proforma | Main procurement document |
| I | Invoice / Batch | Specific shipment document |
| T | Truck | Land transport unit |
| C | Container | Ocean transport unit |
| TS | Template Status | Reusable status template |
| PS | Proforma Status | One of 5 fixed statuses in proforma |
| PSP | Proforma Status Percentage | Percentage value for each PS |
| PSS | Proforma Sub Status | Sub-status within a PS (payment checkpoint) |

---

## Example: Proforma P-210

### Proforma Details
- **Proforma Number:** P-210
- **Total Value:** 430,500 USD
- **Total Containers:** 13
- **Currency:** USD

### Proforma Statuses (5 Fixed Statuses with Percentages)

| PS | Status Name | Percentage | Description |
|----|-------------|------------|-------------|
| P1 | Продукция готова на заводе | 20% | Product ready at factory |
| P2 | В пути на воде | 20% | On the way by water |
| P3 | В пути на суше | 20% | On the way by land |
| P4 | Пересёк границу с Узбекистаном | 20% | Crossed border with Uzbekistan |
| P5 | Прибыл на склад получателя | 20% | Arrived at recipient's warehouse |

**Total:** 100%

---

## Status Hierarchy with Sub-Statuses

**Important:** Sub-statuses track progress within a stage, but **debt is calculated when the entire stage (PS) is complete**, not per sub-status.

### P1: Продукция готова на заводе (20%)

| PSS Code | Sub-Status Name | Purpose | Debt Trigger |
|----------|----------------|---------|--------------|
| P1-S1 | Отгружено с заводов | Track progress | None (tracking only) |
| P1-S2 | Готово к отправке | Track progress | None (tracking only) |

**Debt Calculation:** When **P1 stage is complete** → Debt = 20% of container value

### P2: В пути на воде (20%)

| PSS Code | Sub-Status Name | Purpose | Debt Trigger |
|----------|----------------|---------|--------------|
| P2-S1 | В порту Индии | Track progress | None (tracking only) |
| P2-S2 | В пути (океан) | Track progress | None (tracking only) |
| P2-S3 | Прибыло в порт назначения | Track progress | None (tracking only) |

**Debt Calculation:** When **P2 stage is complete** → Debt = 20% of container value

### P3: В пути на суше (20%)

| PSS Code | Sub-Status Name | Purpose | Debt Trigger |
|----------|----------------|---------|--------------|
| P3-S1 | Отгружено с порта | Track progress | None (tracking only) |
| P3-S2 | В пути (суша) | Track progress | None (tracking only) |

**Debt Calculation:** When **P3 stage is complete** → Debt = 20% of container value

### P4: Пересёк границу с Узбекистаном (20%)

| PSS Code | Sub-Status Name | Purpose | Debt Trigger |
|----------|----------------|---------|--------------|
| P4-S1 | Прибыло в порт Дубая | Track progress | None (tracking only) |
| P4-S2 | На таможне | Track progress | None (tracking only) |
| P4-S3 | Прошло таможню | Track progress | None (tracking only) |

**Debt Calculation:** When **P4 stage is complete** → Debt = 20% of container value

### P5: Прибыл на склад получателя (20%)

| PSS Code | Sub-Status Name | Purpose | Debt Trigger |
|----------|----------------|---------|--------------|
| P5-S1 | Прибыло на таможню | Track progress | None (tracking only) |
| P5-S2 | Принято на складе | Track progress | None (tracking only) |

**Debt Calculation:** When **P5 stage is complete** → Debt = 20% of container value

---

## Invoice I-001 (Example)

### Invoice Details
- **Invoice Number:** I-001
- **Proforma:** P-210
- **Batch Number:** P5.1
- **Total Value:** 680,400 USD
- **Containers:** 6 containers

### Container Breakdown

| Container | Product | Volume (kg) | Price/kg | Container Value | Current Stage | Sub-Status Progress |
|-----------|---------|-------------|----------|-----------------|---------------|---------------------|
| K1111 | Compensated | 28,000 | 3.90 | 109,200 | P2 (complete) | P1-S1 ✓, P1-S2 ✓, P2-S1 ✓, P2-S2 ✓ |
| K2222 | Compensated | 28,000 | 3.90 | 109,200 | P2 (complete) | P1-S1 ✓, P1-S2 ✓, P2-S1 ✓, P2-S2 ✓ |
| K3333 | Compensated | 28,000 | 3.90 | 109,200 | P1 (complete) | P1-S1 ✓, P1-S2 ✓ |
| K4444 | 4HQ | 28,000 | 4.20 | 117,600 | P1 (complete) | P1-S1 ✓, P1-S2 ✓ |
| K5555 | 4HQ | 28,000 | 4.20 | 117,600 | P1 (complete) | P1-S1 ✓, P1-S2 ✓ |
| K6666 | 4HQ | 28,000 | 4.20 | 117,600 | P1 (complete) | P1-S1 ✓, P1-S2 ✓ |

**Total Invoice Value:** 680,400 USD

---

## Debt Calculation Per Container (Current vs Correct)

### Container K1111 (Value: 109,200 USD)

#### Current (INCORRECT) Behavior:
- When K1111 **stage P1 completes**: Debt = **680,400 USD** (entire invoice) ❌
- When K1111 **stage P2 completes**: Debt = **680,400 USD** (entire invoice again) ❌
- **Problem:** Uses invoice total instead of container value

#### Correct Behavior:
- When K1111 **stage P1 completes** (all P1 sub-statuses done): Debt = **21,840 USD** (20% of 109,200) ✓
- When K1111 **stage P2 completes** (all P2 sub-statuses done): Debt = **21,840 USD** (20% of 109,200) ✓
- **Total for K1111 after P2:** 43,680 USD (40% of container value)
- **Note:** Sub-status changes (P1-S1, P1-S2, P2-S1, P2-S2) do NOT trigger debt - only stage completion does

### Container K3333 (Value: 109,200 USD)

#### Correct Behavior:
- When K3333 **stage P1 completes**: Debt = **21,840 USD** (20% of 109,200) ✓
- **Total for K3333 after P1:** 21,840 USD (20% of container value)
- **Note:** K3333 is still in P1, so only P1 debt has been accrued

---

## Product Movement: Container → Truck (Partial Movement)

### Scenario: Container K1111 - Partial product movement to trucks

**Important:** Products can be **partially moved** from a container. The remaining products stay in the container and can continue progressing through stages independently.

#### Before Movement:
- **Container K1111:** 
  - Total: 28,000 kg
  - Total Value: 109,200 USD
  - Stage P2 complete (all P2 sub-statuses done)
  - **Accrued Debt:** 43,680 USD (40% of 109,200 = P1: 20% + P2: 20%)
  - **Remaining Debt:** 65,520 USD (60% of 109,200 = P3: 20% + P4: 20% + P5: 20%)

#### Movement Action (Partial):
- **14,000 kg moved to Truck T-123** (50% of container = 54,600 USD value)
- **14,000 kg remains in Container K1111** (50% of container = 54,600 USD value)

#### After Movement (CORRECT Approach - Complex Debt Calculation):

**The container is split into two independent tracking units:**

1. **Container K1111 (Remaining 50% = 14,000 kg, 54,600 USD value):**
   - **Locked debt from completed stages:** 21,840 USD (40% of 54,600 = P1: 10,920 + P2: 10,920)
   - **Can continue progressing** through remaining stages: P3, P4, P5
   - When K1111 **stage P3 completes**: Debt = 10,920 USD (20% of 54,600)
   - When K1111 **stage P4 completes**: Debt = 10,920 USD (20% of 54,600)
   - When K1111 **stage P5 completes**: Debt = 10,920 USD (20% of 54,600)
   - **Total for container portion:** 54,600 USD (100% of 54,600)

2. **Truck T-123 (Moved 50% = 14,000 kg, 54,600 USD value):**
   - **Locked debt from completed stages:** 21,840 USD (40% of 54,600 = P1: 10,920 + P2: 10,920)
   - **New status chain starts:** P3, P4, P5 stages (truck stages)
   - When T-123 **stage P3 completes**: Debt = 10,920 USD (20% of 54,600)
   - When T-123 **stage P4 completes**: Debt = 10,920 USD (20% of 54,600)
   - When T-123 **stage P5 completes**: Debt = 10,920 USD (20% of 54,600)
   - **Total for truck portion:** 54,600 USD (100% of 54,600)

**Total Debt for K1111 (Container portion + Truck portion):** 109,200 USD (100% of original container value) ✓
- Container K1111 (remaining): 54,600 USD (50%)
- Truck T-123 (moved): 54,600 USD (50%)

**Key Complexity:**
- Both the container and truck can progress through stages **independently**
- Debt is calculated separately for each portion based on its own stage progression
- The container portion and truck portion are tracked separately but both reference the original container K1111

---

## Product Movement: Multiple Containers → Single Truck (Complex Scenario)

### Scenario: Products from different containers (different proformas/invoices) moved to one truck

#### Setup:
- **Proforma P-210** (Invoice I-001):
  - Container K1111: 109,200 USD, Stage P2 complete (40% debt accrued)
  - Container K2222: 109,200 USD, Stage P1 complete (20% debt accrued)
  
- **Proforma P-211** (Invoice I-002):
  - Container K7777: 150,000 USD, Stage P2 complete (40% debt accrued)
  - Container K8888: 120,000 USD, Stage P1 complete (20% debt accrued)

#### Movement Action (Partial from each container):
- **Truck T-999** receives products from:
  - K1111: 14,000 kg moved (50% of container = 54,600 USD value) - **14,000 kg remains in K1111**
  - K2222: 21,000 kg moved (75% of container = 81,900 USD value) - **7,000 kg remains in K2222**
  - K7777: 20,000 kg moved (40% of container = 60,000 USD value) - **30,000 kg remains in K7777**
  - K8888: 15,000 kg moved (50% of container = 60,000 USD value) - **15,000 kg remains in K8888**

**Total in Truck T-999:** 256,500 USD (from 4 different containers, 2 different proformas, 2 different invoices)
**Remaining in containers:** Products still remain in each original container and can continue progressing independently

#### After Movement (CORRECT Approach):

**Each product maintains its original references:**
- Product from K1111 → Truck T-999: References (P-210, I-001, K1111)
- Product from K2222 → Truck T-999: References (P-210, I-001, K2222)
- Product from K7777 → Truck T-999: References (P-211, I-002, K7777)
- Product from K8888 → Truck T-999: References (P-211, I-002, K8888)

**Debt Calculation Per Original Container (Split Tracking):**

1. **Container K1111:**
   - **Remaining in container (50% = 54,600 USD):**
     - Locked debt: 21,840 USD (P1: 10,920 + P2: 10,920)
     - Can continue: P3, P4, P5 stages
     - When container **stage P3 completes**: Debt = 10,920 USD (20% of 54,600)
     - When container **stage P4 completes**: Debt = 10,920 USD (20% of 54,600)
     - When container **stage P5 completes**: Debt = 10,920 USD (20% of 54,600)
   
   - **Moved to T-999 (50% = 54,600 USD):**
     - Locked debt: 21,840 USD (P1: 10,920 + P2: 10,920)
     - Continues on truck: P3, P4, P5 stages
     - When T-999 **stage P3 completes**: Debt = 10,920 USD (20% of 54,600)
     - When T-999 **stage P4 completes**: Debt = 10,920 USD (20% of 54,600)
     - When T-999 **stage P5 completes**: Debt = 10,920 USD (20% of 54,600)
   
   **Total for K1111:** 109,200 USD (container portion + truck portion)

2. **Container K2222:**
   - **Remaining in container (25% = 27,300 USD):**
     - Locked debt: 5,460 USD (P1: 5,460)
     - Can continue: P2, P3, P4, P5 stages
   
   - **Moved to T-999 (75% = 81,900 USD):**
     - Locked debt: 16,380 USD (P1: 16,380)
     - Continues on truck: P2, P3, P4, P5 stages
     - When T-999 **stage P2 completes**: Debt = 16,380 USD (20% of 81,900)
     - When T-999 **stage P3 completes**: Debt = 16,380 USD (20% of 81,900)
     - When T-999 **stage P4 completes**: Debt = 16,380 USD (20% of 81,900)
     - When T-999 **stage P5 completes**: Debt = 16,380 USD (20% of 81,900)
   
   **Total for K2222:** 109,200 USD (container portion + truck portion)

3. **Container K7777:**
   - **Remaining in container (60% = 90,000 USD):**
     - Locked debt: 36,000 USD (P1: 18,000 + P2: 18,000)
     - Can continue: P3, P4, P5 stages
   
   - **Moved to T-999 (40% = 60,000 USD):**
     - Locked debt: 24,000 USD (P1: 12,000 + P2: 12,000)
     - Continues on truck: P3, P4, P5 stages
     - When T-999 **stage P3 completes**: Debt = 12,000 USD (20% of 60,000)
     - When T-999 **stage P4 completes**: Debt = 12,000 USD (20% of 60,000)
     - When T-999 **stage P5 completes**: Debt = 12,000 USD (20% of 60,000)
   
   **Total for K7777:** 150,000 USD (container portion + truck portion)

4. **Container K8888:**
   - **Remaining in container (50% = 60,000 USD):**
     - Locked debt: 12,000 USD (P1: 12,000)
     - Can continue: P2, P3, P4, P5 stages
   
   - **Moved to T-999 (50% = 60,000 USD):**
     - Locked debt: 12,000 USD (P1: 12,000)
     - Continues on truck: P2, P3, P4, P5 stages
     - When T-999 **stage P2 completes**: Debt = 12,000 USD (20% of 60,000)
     - When T-999 **stage P3 completes**: Debt = 12,000 USD (20% of 60,000)
     - When T-999 **stage P4 completes**: Debt = 12,000 USD (20% of 60,000)
     - When T-999 **stage P5 completes**: Debt = 12,000 USD (20% of 60,000)
   
   **Total for K8888:** 120,000 USD (container portion + truck portion)

**When Truck T-999 completes stage P3:**
- Total debt accrued = 10,920 (K1111) + 16,380 (K2222) + 12,000 (K7777) + 12,000 (K8888) = **51,300 USD**
- Each amount is calculated based on its **original container's value and remaining stages**

**Key Point:** Even though all products are in the same truck, debt is calculated **per original container**, maintaining the link to the original proforma/invoice/container.

---

## Debt Summary Table

| Transport Unit | Type | Value | Current Stage | Completed Stages | Debt Accrued | Remaining |
|----------------|------|-------|---------------|------------------|--------------|-----------|
| K1111 | Container | 109,200 | P2 (complete) | P1, P2 | 43,680 (40%) | 65,520 (60%) |
| T-123 | Truck | 54,600* | P3 (in progress) | None yet | 0 (0%) | 32,760 (30%) |
| T-456 | Truck | 54,600* | P3 (in progress) | None yet | 0 (0%) | 32,760 (30%) |

*Truck value = 50% of container value (proportional split)
**Note:** Trucks will accrue debt when their stages (P3, P4, P5) complete, not when sub-statuses change

---

## Key Rules

1. **Debt is calculated per container value**, not invoice total
2. **Each container tracks its own status progression** independently
3. **Debt accrues when a STAGE (PS) completes**, not when sub-statuses change
   - Sub-statuses (PSS) are for tracking progress only
   - Debt = Stage Percentage × Container Value (when stage is complete)
4. **When products move from container to truck (PARTIAL MOVEMENT - Complex):**
   - Products can be **partially moved** (e.g., 14,000 kg out of 28,000 kg)
   - **Remaining products stay in the container** and can continue progressing independently
   - **Both portions track debt separately:**
     - **Container portion:** Continues with container's stage progression
     - **Truck portion:** Continues with truck's stage progression
   - **Debt calculation complexity:**
     - Each portion has its own locked debt from completed stages (proportional to amount)
     - Each portion can progress through remaining stages independently
     - Debt is calculated separately when each portion completes a stage
     - Example: If 50% moved to truck, container portion = 50% of original value, truck portion = 50% of original value
     - Both portions reference the original container for tracking

5. **Multiple containers can move products to the same truck:**
   - Products from different containers, invoices, and proformas can be in one truck
   - **Each product maintains its original references** (proforma, invoice, container)
   - **Debt is calculated per original container portion**, not per truck
   - When truck completes a stage, debt is calculated separately for each product portion based on:
     - Its original container's value (proportional to amount moved)
     - Its original container's completed stages (locked debt for that portion)
     - Remaining stages applicable to that product portion
   - **The container's remaining portion** can also progress independently and accrue debt separately
6. **Status percentages are configured per proforma** (can vary, but total = 100%)
7. **No double-counting:** Each stage percentage is only paid once per container (but split across portions)
8. **Stage completion criteria:** All sub-statuses within a stage must be complete for the stage to be considered complete

---

## Implementation Challenge

**The complexity of partial product movement makes debt calculation challenging:**

1. **Tracking Requirements:**
   - Need to track what percentage/amount of each container's products are in which transport unit
   - Need to track stage progression separately for:
     - Container's remaining portion
     - Each truck that received products from that container
   - Need to maintain references to original container/invoice/proforma for each product portion

2. **Debt Calculation Requirements:**
   - Calculate debt based on original container value (proportional to amount moved)
   - Lock debt from completed stages (proportional to amount)
   - Calculate debt separately when each portion (container or truck) completes a stage
   - Ensure total debt for original container = 100% of container value (across all portions)

3. **Data Model Considerations:**
   - How to represent partial product movement in database?
   - How to link truck products back to original container portions?
   - How to track stage progression for split portions?
   - How to prevent double-counting when calculating total debt?

