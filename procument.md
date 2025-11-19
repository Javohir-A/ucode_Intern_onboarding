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

## Product Movement: Container → Truck

### Scenario: Container K1111 arrives, products moved to trucks

#### Before Movement:
- **Container K1111:** Stage P2 complete (all P2 sub-statuses done)
- **Accrued Debt:** 43,680 USD (40% of 109,200 = P1: 20% + P2: 20%)
- **Remaining Debt:** 65,520 USD (60% of 109,200 = P3: 20% + P4: 20% + P5: 20%)

#### Movement Action:
- Products from K1111 split into:
  - **Truck T-123:** 14,000 kg (50% of container = 54,600 USD value)
  - **Truck T-456:** 14,000 kg (50% of container = 54,600 USD value)

#### After Movement (CORRECT Approach):
- **Container K1111:** 
  - Stage frozen at P2 (complete)
  - Debt recorded: 43,680 USD (permanent record - P1: 21,840 + P2: 21,840)
  - No further debt accrual

- **Truck T-123:**
  - Inherits: 50% of remaining stages = 32,760 USD (30% of 109,200 = P3: 10,920 + P4: 10,920 + P5: 10,920)
  - New status chain starts: P3, P4, P5 stages
  - When T-123 **stage P3 completes**: Debt = 10,920 USD (20% of 109,200)
  - When T-123 **stage P4 completes**: Debt = 10,920 USD (20% of 109,200)
  - When T-123 **stage P5 completes**: Debt = 10,920 USD (20% of 109,200)
  - **Total for T-123 portion:** 32,760 USD (30% of container value)

- **Truck T-456:**
  - Inherits: 50% of remaining stages = 32,760 USD (30% of 109,200)
  - Same status progression as T-123
  - **Total for T-456 portion:** 32,760 USD (30% of container value)

**Total Debt for K1111 (Container + Trucks):** 109,200 USD (100% of container value) ✓
- Container: 43,680 USD (40%)
- Truck T-123: 32,760 USD (30%)
- Truck T-456: 32,760 USD (30%)

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

#### Movement Action:
- **Truck T-999** receives products from:
  - K1111: 14,000 kg (50% of container = 54,600 USD value)
  - K2222: 21,000 kg (75% of container = 81,900 USD value)
  - K7777: 20,000 kg (40% of container = 60,000 USD value)
  - K8888: 15,000 kg (50% of container = 60,000 USD value)

**Total in Truck T-999:** 256,500 USD (from 4 different containers, 2 different proformas, 2 different invoices)

#### After Movement (CORRECT Approach):

**Each product maintains its original references:**
- Product from K1111 → Truck T-999: References (P-210, I-001, K1111)
- Product from K2222 → Truck T-999: References (P-210, I-001, K2222)
- Product from K7777 → Truck T-999: References (P-211, I-002, K7777)
- Product from K8888 → Truck T-999: References (P-211, I-002, K8888)

**Debt Calculation Per Original Container:**

1. **Container K1111 (50% moved to T-999):**
   - Container debt locked: 21,840 USD (P1: 10,920 + P2: 10,920) for 50% portion
   - Remaining in T-999: 32,760 USD (P3: 10,920 + P4: 10,920 + P5: 10,920)
   - When T-999 **stage P3 completes**: Debt = 10,920 USD (20% of 54,600)
   - When T-999 **stage P4 completes**: Debt = 10,920 USD (20% of 54,600)
   - When T-999 **stage P5 completes**: Debt = 10,920 USD (20% of 54,600)

2. **Container K2222 (75% moved to T-999):**
   - Container debt locked: 16,380 USD (P1: 16,380) for 75% portion
   - Remaining in T-999: 65,520 USD (P2: 16,380 + P3: 16,380 + P4: 16,380 + P5: 16,380)
   - When T-999 **stage P2 completes**: Debt = 16,380 USD (20% of 81,900)
   - When T-999 **stage P3 completes**: Debt = 16,380 USD (20% of 81,900)
   - When T-999 **stage P4 completes**: Debt = 16,380 USD (20% of 81,900)
   - When T-999 **stage P5 completes**: Debt = 16,380 USD (20% of 81,900)

3. **Container K7777 (40% moved to T-999):**
   - Container debt locked: 24,000 USD (P1: 12,000 + P2: 12,000) for 40% portion
   - Remaining in T-999: 36,000 USD (P3: 12,000 + P4: 12,000 + P5: 12,000)
   - When T-999 **stage P3 completes**: Debt = 12,000 USD (20% of 60,000)
   - When T-999 **stage P4 completes**: Debt = 12,000 USD (20% of 60,000)
   - When T-999 **stage P5 completes**: Debt = 12,000 USD (20% of 60,000)

4. **Container K8888 (50% moved to T-999):**
   - Container debt locked: 12,000 USD (P1: 12,000) for 50% portion
   - Remaining in T-999: 48,000 USD (P2: 12,000 + P3: 12,000 + P4: 12,000 + P5: 12,000)
   - When T-999 **stage P2 completes**: Debt = 12,000 USD (20% of 60,000)
   - When T-999 **stage P3 completes**: Debt = 12,000 USD (20% of 60,000)
   - When T-999 **stage P4 completes**: Debt = 12,000 USD (20% of 60,000)
   - When T-999 **stage P5 completes**: Debt = 12,000 USD (20% of 60,000)

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
4. **When products move from container to truck:**
   - Container's accrued debt is **locked** (cannot be changed)
   - Remaining stages (and their percentages) are **split proportionally** to trucks
   - Trucks continue with remaining stages (P3, P4, P5)
5. **Multiple containers can move products to the same truck:**
   - Products from different containers, invoices, and proformas can be in one truck
   - **Each product maintains its original references** (proforma, invoice, container)
   - **Debt is calculated per original container**, not per truck
   - When truck completes a stage, debt is calculated separately for each product based on:
     - Its original container's value (proportional to amount moved)
     - Its original container's completed stages (locked debt)
     - Remaining stages applicable to that product
6. **Status percentages are configured per proforma** (can vary, but total = 100%)
7. **No double-counting:** Each stage percentage is only paid once per container
8. **Stage completion criteria:** All sub-statuses within a stage must be complete for the stage to be considered complete

