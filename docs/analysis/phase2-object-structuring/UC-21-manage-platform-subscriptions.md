# Object Structuring: Manage Platform Subscriptions

**Use Case**: docs/requirements/use-cases/UC-21-manage-platform-subscriptions.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Actor**: Platform Administrator
**External Class (Phase 1)**: PlatformAdministrator («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| AdminInteraction | «user interaction» | Maps from PlatformAdministrator - handles subscription list, search, admin actions |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: System displays subscription list with filters
- Step 4: System displays subscription details (plan, billing, payment history, status)
- Step 5: Admin performs actions (upgrade/downgrade, extend trial, refund, cancel, waive fees)
- Step 6: System updates subscription and feature access
- Postconditions: Subscription updated, features adjusted, changes logged

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| Subscription | Yes | ✅ |
| Property | Yes | ✅ |
| Payment | Yes | ✅ |
| SubscriptionPlan | Yes | ✅ |
| Refund | No - Add | ❌ Refund tracking |
| AdminAuditLog | New from UC-20 | ✅ |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen
- [ ] Printer
- [ ] External device
- [x] Same as input

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| AdminInteraction | «user interaction» | Reused for display subscription list, details, confirmations |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Admin subscription actions are discrete operations. Each action is independent without temporal dependencies. No state-based behavior changes.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| SubscriptionAdminCoordinator | «coordinator» | No |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation
- [x] Algorithms
- [x] Cross-entity validation

| Object | Stereotype | Justification |
|--------|------------|---------------|
| RefundCalculationService | «business logic» | Calculates pro-rated refunds based on billing cycle, usage, and refund policies |
| FeatureAccessService | «service» | Updates feature access flags when plan changes (immediate effect) |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| AdminInteraction | «user interaction» | Boundary | PlatformAdministrator |
| Subscription | «entity» | Entity | Subscription |
| Property | «entity» | Entity | Property |
| Payment | «entity» | Entity | Payment |
| SubscriptionPlan | «entity» | Entity | SubscriptionPlan |
| Refund | «entity» | Entity | (New) |
| AdminAuditLog | «entity» | Entity | AdminAuditLog |
| SubscriptionAdminCoordinator | «coordinator» | Control | N/A |
| RefundCalculationService | «business logic» | Logic | N/A |
| FeatureAccessService | «service» | Logic | N/A |

**Total**: 10 objects (1 boundary, 6 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: PlatformAdministrator
- Boundary: AdminInteraction
- Control: SubscriptionAdminCoordinator
- Entity: Subscription, Property, Payment, SubscriptionPlan, Refund, AdminAuditLog
- Logic: RefundCalculationService, FeatureAccessService

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

(No statechart needed - coordinator only)

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (5 existing, 1 new to add)
- [x] Control justified
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified

**Notes**: Need to update Phase 1 to add: Refund entity. Reuses AdminAuditLog from UC-20.
