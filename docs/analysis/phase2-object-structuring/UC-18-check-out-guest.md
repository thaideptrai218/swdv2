# Object Structuring: Check Out Guest

**Use Case**: docs/requirements/use-cases/UC-18-check-out-guest.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Actor**: Hostel Staff
**External Class (Phase 1)**: HostelStaff («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| CheckOutInteraction | «user interaction» | Maps from HostelStaff - handles departures list, search, check-out form, charge collection |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: System displays today's expected departures
- Step 5: System displays check-out form with stay details
- Step 8: Staff inspects room/bed, notes damages
- Step 9: System processes outstanding charges
- Step 15: System updates booking status to "Completed"
- Step 17: System releases bed/room, marks "Needs Cleaning"
- Postconditions: Booking completed, charges settled, inventory released

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| Booking | Yes | ✅ |
| Guest | Yes (User) | ✅ |
| Room | Yes | ✅ |
| Bed | New from UC-17 | ✅ |
| Payment | Yes | ✅ |
| Charge | No - Add | ❌ Additional charges |
| GuestFeedback | No - Add | ❌ Feedback notes |
| LostProperty | No - Add | ❌ Lost items tracking |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen
- [x] Printer (receipt)
- [ ] External device
- [x] Same as input

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| CheckOutInteraction | «user interaction» | Reused for display departures, form, receipt |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Check-out is sequential workflow without state modes. Same steps: search → collect charges → settle payment → release inventory → complete. No temporal dependencies or varying behavior based on state.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| CheckOutCoordinator | «coordinator» | No |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation
- [x] Algorithms
- [ ] Cross-entity validation

| Object | Stereotype | Justification |
|--------|------------|---------------|
| ChargeCalculationService | «business logic» | Calculates late check-out fees, damage charges, service charges based on property policies |
| InventoryReleaseService | «service» | Manages bed/room release, cleaning status updates, availability synchronization |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| CheckOutInteraction | «user interaction» | Boundary | HostelStaff |
| Booking | «entity» | Entity | Booking |
| Guest | «entity» | Entity | User/Guest |
| Room | «entity» | Entity | Room |
| Bed | «entity» | Entity | Bed |
| Payment | «entity» | Entity | Payment |
| Charge | «entity» | Entity | (New) |
| GuestFeedback | «entity» | Entity | (New) |
| LostProperty | «entity» | Entity | (New) |
| CheckOutCoordinator | «coordinator» | Control | N/A |
| ChargeCalculationService | «business logic» | Logic | N/A |
| InventoryReleaseService | «service» | Logic | N/A |

**Total**: 12 objects (1 boundary, 8 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: HostelStaff
- Boundary: CheckOutInteraction
- Control: CheckOutCoordinator
- Entity: Booking, Guest, Room, Bed, Payment, Charge, GuestFeedback, LostProperty
- Logic: ChargeCalculationService, InventoryReleaseService

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

(No statechart needed - coordinator only)

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (5 existing, 3 new to add)
- [x] Control justified
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified

**Notes**: Need to update Phase 1 to add: Charge, GuestFeedback, LostProperty entities.
