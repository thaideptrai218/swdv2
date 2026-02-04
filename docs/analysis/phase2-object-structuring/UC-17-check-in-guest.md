# Object Structuring: Check In Guest

**Use Case**: docs/requirements/use-cases/UC-17-check-in-guest.md
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
| CheckInInteraction | «user interaction» | Maps from HostelStaff - handles arrivals list, search, check-in form, bed assignment |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: System displays today's expected arrivals
- Step 5: System displays check-in form with booking details
- Step 10: System displays room layout with available beds
- Step 15: System updates booking status to "Checked-In"
- Step 17: System marks assigned bed/room as occupied
- Preconditions: Confirmed booking, guest arrived
- Postconditions: Booking checked-in, bed/room assigned

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| Booking | Yes | ✅ |
| Guest | Yes (User) | ✅ |
| Room | Yes | ✅ |
| Bed | No - Add | ❌ Hostel-specific |
| Payment | Yes | ✅ |
| IDDocument | No - Add | ❌ ID verification |
| EmergencyContact | No - Add | ❌ Required info |

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
| CheckInInteraction | «user interaction» | Reused for display arrivals, form, confirmation |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Check-in is sequential workflow without state-dependent behavior. Same steps for every check-in: search → verify → collect info → assign bed → complete. No temporal modes or varying responses.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| CheckInCoordinator | «coordinator» | No |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation
- [x] Algorithms
- [ ] Cross-entity validation

| Object | Stereotype | Justification |
|--------|------------|---------------|
| BedAssignmentService | «service» | Algorithm for optimal bed assignment in dorm (layout, availability, conflicts) |
| IDVerificationService | «business logic» | Validates ID documents, extracts information (future: digital scanning) |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| CheckInInteraction | «user interaction» | Boundary | HostelStaff |
| Booking | «entity» | Entity | Booking |
| Guest | «entity» | Entity | User/Guest |
| Room | «entity» | Entity | Room |
| Bed | «entity» | Entity | (New) |
| Payment | «entity» | Entity | Payment |
| IDDocument | «entity» | Entity | (New) |
| EmergencyContact | «entity» | Entity | (New) |
| CheckInCoordinator | «coordinator» | Control | N/A |
| BedAssignmentService | «service» | Logic | N/A |
| IDVerificationService | «business logic» | Logic | N/A |

**Total**: 11 objects (1 boundary, 7 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: HostelStaff
- Boundary: CheckInInteraction
- Control: CheckInCoordinator
- Entity: Booking, Guest, Room, Bed, Payment, IDDocument, EmergencyContact
- Logic: BedAssignmentService, IDVerificationService

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

(No statechart needed - coordinator only)

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (4 existing, 3 new to add)
- [x] Control justified
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified

**Notes**: Need to update Phase 1 to add: Bed, IDDocument, EmergencyContact entities.
