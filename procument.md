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
- **Container K1111:** Status = P2-S2 (В пути океан)
- **Accrued Debt:** 43,680 USD (40% of 109,200)
- **Remaining Debt:** 65,520 USD (60% of 109,200)

#### Movement Action:
- Products from K1111 split into:
  - **Truck T-123:** 14,000 kg (50% of container)
  - **Truck T-456:** 14,000 kg (50% of container)

#### After Movement (CORRECT Approach):
- **Container K1111:** 
  - Status frozen at P2-S2
  - Debt recorded: 43,680 USD (permanent record)
  - No further debt accrual

- **Truck T-123:**
  - Inherits: 50% of remaining debt = 32,760 USD (30% of 109,200)
  - New status chain starts: P3-S1, P3-S2, P4-S1, P4-S2, P5-S1, P5-S2
  - When T-123 reaches P3-S1: Debt = 5,460 USD (5% of 109,200)
  - When T-123 reaches P3-S2: Debt = 5,460 USD (5% of 109,200)
  - **Total for T-123 portion:** 43,680 USD (40% of container value)

- **Truck T-456:**
  - Inherits: 50% of remaining debt = 32,760 USD (30% of 109,200)
  - Same status progression as T-123
  - **Total for T-456 portion:** 43,680 USD (40% of container value)

**Total Debt for K1111 (Container + Trucks):** 109,200 USD (100% of container value) ✓

---

## Debt Summary Table

| Transport Unit | Type | Value | Status Progress | Debt Accrued | Remaining |
|----------------|------|-------|----------------|--------------|-----------|
| K1111 | Container | 109,200 | P2-S2 | 43,680 (40%) | 65,520 (60%) |
| T-123 | Truck | 54,600* | P3-S1 | 5,460 (5%) | 49,140 (45%) |
| T-456 | Truck | 54,600* | P3-S1 | 5,460 (5%) | 49,140 (45%) |

*Truck value = 50% of container value (proportional split)

---

## Key Rules

1. **Debt is calculated per container value**, not invoice total
2. **Each container tracks its own status progression** independently
3. **When products move from container to truck:**
   - Container's accrued debt is **locked** (cannot be changed)
   - Remaining debt is **split proportionally** to trucks
   - Trucks continue with remaining status percentages
4. **Status percentages are configured per proforma** (can vary, but total = 100%)
5. **Sub-statuses trigger debt** when marked complete
6. **No double-counting:** Each percentage is only paid once per container

