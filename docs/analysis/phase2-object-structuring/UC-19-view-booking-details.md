# Object Structuring: View Booking Details

**Use Case**: docs/requirements/use-cases/UC-19-view-booking-details.md
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
| BookingDetailsInteraction | «user interaction» | Maps from HostelStaff - handles search, details display, quick actions |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 3: System retrieves matching bookings
- Step 6-13: System displays comprehensive booking details (guest info, booking details, room assignment, payment, communication, timeline, staff notes)
- All read-only operations - no data modification
- Preconditions: Booking exists
- Postconditions: No changes (read-only view)

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| Booking | Yes | ✅ |
| Guest | Yes (User) | ✅ |
| Room | Yes | ✅ |
| Bed | New from UC-17 | ✅ |
| Payment | Yes | ✅ |
| Message | New from UC-16 | ✅ |
| MessageThread | New from UC-16 | ✅ |
| StaffNote | No - Add | ❌ Internal notes |
| ActivityLog | No - Add | ❌ Audit timeline |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen
- [x] Printer (booking summary)
- [ ] External device
- [x] Same as input

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingDetailsInteraction | «user interaction» | Reused for display search results, details, printable view |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Read-only view with no control logic needed. Pure data retrieval and display. No workflow coordination required - just fetch and show.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| (None needed) | N/A | No |

---

## 5. Application Logic

**Complex Rules**:
- [ ] Multi-condition validation
- [ ] Algorithms
- [ ] Cross-entity validation

**No application logic objects needed** - pure read/display operation.

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| BookingDetailsInteraction | «user interaction» | Boundary | HostelStaff |
| Booking | «entity» | Entity | Booking |
| Guest | «entity» | Entity | User/Guest |
| Room | «entity» | Entity | Room |
| Bed | «entity» | Entity | Bed |
| Payment | «entity» | Entity | Payment |
| Message | «entity» | Entity | Message |
| MessageThread | «entity» | Entity | MessageThread |
| StaffNote | «entity» | Entity | (New) |
| ActivityLog | «entity» | Entity | (New) |

**Total**: 10 objects (1 boundary, 9 entity, 0 control, 0 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: HostelStaff
- Boundary: BookingDetailsInteraction
- Control: (None - simple read)
- Entity: Booking, Guest, Room, Bed, Payment, Message, MessageThread, StaffNote, ActivityLog
- Logic: (None)

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

(No statechart needed - read-only view)

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (7 existing, 2 new to add)
- [x] Control justified (none needed for read-only)
- [x] State-dependent flagged (N/A - no control)
- [x] Logic justified (none needed)

**Notes**: Need to update Phase 1 to add: StaffNote, ActivityLog entities. This is simplest use case - pure information display.
