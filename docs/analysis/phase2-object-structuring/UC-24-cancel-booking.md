# Object Structuring: Cancel Booking

**Use Case**: docs/requirements/use-cases/UC-24-cancel-booking.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Actor**: Traveler
**External Class (Phase 1)**: Traveler («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| CancellationInteraction | «user interaction» | Maps from Traveler - handles cancellation request, policy display, confirmation |

**Secondary Actors**:
- Payment Gateway (refund processing)
- Email Service (notifications)

**External Classes**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| PaymentGatewayProxy | «proxy» | Maps from PaymentGateway - processes refunds to original payment method |
| EmailServiceProxy | «proxy» | Maps from EmailService - sends confirmations and notifications |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 4: System retrieves cancellation policy for booking
- Step 5: System calculates refund amount based on policy and timing
- Step 10: System updates booking status to "Cancelled"
- Step 11: System records cancellation timestamp and reason
- Step 15: System releases bed/room from occupancy
- Step 16: System updates availability
- Preconditions: Booking confirmed
- Postconditions: Booking cancelled, refund processed, inventory released

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| Booking | Yes | ✅ |
| Payment | Yes | ✅ |
| Refund | New from UC-21 | ✅ |
| CancellationPolicy | No - Add | ❌ Policy rules and timing |
| Room | Yes | ✅ |
| Bed | New from UC-17 | ✅ |
| CancellationRecord | No - Add | ❌ Audit trail with reason |

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
| CancellationInteraction | «user interaction» | Reused for display policy, refund breakdown, confirmation |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Cancellation behavior is **STATE-DEPENDENT** based on timing windows:
- Within 24h of booking: Full refund (state: EligibleForFullRefund)
- After 24h: No refund (state: NoRefundEligible)
- After check-in: Cannot cancel (state: CheckedIn)
- Within 24h of check-in: Cancellation fee applies (state: LateCancellation)

Same event (cancel request) produces different responses based on temporal state. System must track time elapsed since booking and time until check-in to determine applicable policy.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| CancellationControl | «state-dependent control» | ⚠️ Yes |

**States for Statechart**:
1. EligibleForFullRefund (< 24h since booking)
2. NoRefundEligible (> 24h since booking, > 24h to check-in)
3. LateCancellation (< 24h to check-in)
4. CheckedIn (cannot cancel)
5. ProcessingRefund
6. RefundCompleted
7. CancellationCompleted

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation
- [x] Algorithms
- [x] Cross-entity validation

| Object | Stereotype | Justification |
|--------|------------|---------------|
| RefundCalculationService | «business logic» | Calculates refund amount based on policy, booking amount, timing, cancellation fees (reused from UC-21) |
| InventoryReleaseService | «service» | Releases bed/room, updates availability, synchronizes with calendar (reused from UC-18) |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| CancellationInteraction | «user interaction» | Boundary | Traveler |
| PaymentGatewayProxy | «proxy» | Boundary | PaymentGateway |
| EmailServiceProxy | «proxy» | Boundary | EmailService |
| Booking | «entity» | Entity | Booking |
| Payment | «entity» | Entity | Payment |
| Refund | «entity» | Entity | Refund |
| CancellationPolicy | «entity» | Entity | (New) |
| Room | «entity» | Entity | Room |
| Bed | «entity» | Entity | Bed |
| CancellationRecord | «entity» | Entity | (New) |
| CancellationControl | «state-dependent control» | Control | N/A |
| RefundCalculationService | «business logic» | Logic | N/A |
| InventoryReleaseService | «service» | Logic | N/A |

**Total**: 13 objects (3 boundary, 7 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: Traveler, PaymentGateway, EmailService
- Boundary: CancellationInteraction, PaymentGatewayProxy, EmailServiceProxy
- Control: CancellationControl
- Entity: Booking, Payment, Refund, CancellationPolicy, Room, Bed, CancellationRecord
- Logic: RefundCalculationService, InventoryReleaseService

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for: CancellationControl**

**Temporal Dependencies**:
- Time since booking (24h window)
- Time until check-in (24h window)
- Check-in status (boolean state)
- Refund processing status

**State Transitions**:
- Booking created → EligibleForFullRefund
- 24h elapsed → NoRefundEligible
- < 24h to check-in → LateCancellation
- Guest checked in → CheckedIn (terminal - no cancel)
- Cancel requested → ProcessingRefund
- Refund confirmed → RefundCompleted → CancellationCompleted

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (5 existing, 2 new to add)
- [x] Control justified (state-dependent)
- [x] State-dependent flagged (YES - Phase 4 required)
- [x] Logic justified

**Notes**: Need to update Phase 1 to add: CancellationPolicy, CancellationRecord entities. This is 9th state-dependent control requiring statechart.
