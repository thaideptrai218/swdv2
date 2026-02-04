# Object Structuring: Communicate With Guest

**Use Case**: docs/requirements/use-cases/UC-16-communicate-with-guest.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Actor**: Hostel Owner
**External Class (Phase 1)**: HostelOwner («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| MessagingInteraction | «user interaction» | Maps from HostelOwner external user - handles inbox, compose, send message |

**Secondary Actor**: Email Service
**External Class (Phase 1)**: EmailService («external system»)

| Object | Stereotype | Justification |
|--------|------------|---------------|
| EmailServiceProxy | «proxy» | Maps from EmailService - delivers messages to guests |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: System retrieves message threads for property
- Step 5: System displays message thread view with conversation history
- Step 5.2: Associated booking details sidebar
- Step 5.3: Guest profile snippet
- Step 9: System saves message to thread
- Preconditions: Booking exists with guest information
- Postconditions: Message sent, conversation stored

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| Message | No - Add | ❌ Need to add |
| MessageThread | No - Add | ❌ Need to add |
| Booking | Yes | ✅ |
| Guest | Yes (User specialization) | ✅ |
| Property | Yes | ✅ |
| MessageTemplate | No - Add | ❌ Quick reply templates |
| Attachment | No - Add | ❌ File attachments |

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
| MessagingInteraction | «user interaction» | Reused for display inbox, threads, sent confirmations |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Messaging is stateless coordination - no temporal behavior that depends on modes. Same sequence regardless of context: retrieve → display → compose → send → notify.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| MessagingCoordinator | «coordinator» | No |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation
- [ ] Algorithms
- [ ] Cross-entity validation

| Object | Stereotype | Justification |
|--------|------------|---------------|
| MessageValidationService | «business logic» | Validates non-empty messages, file size limits, format checks |
| BulkMessagingService | «service» | Handles bulk messaging to multiple guests with personalization |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| MessagingInteraction | «user interaction» | Boundary | HostelOwner |
| EmailServiceProxy | «proxy» | Boundary | EmailService |
| Message | «entity» | Entity | (New) |
| MessageThread | «entity» | Entity | (New) |
| Booking | «entity» | Entity | Booking |
| Guest | «entity» | Entity | User/Guest |
| Property | «entity» | Entity | Property |
| MessageTemplate | «entity» | Entity | (New) |
| Attachment | «entity» | Entity | (New) |
| MessagingCoordinator | «coordinator» | Control | N/A |
| MessageValidationService | «business logic» | Logic | N/A |
| BulkMessagingService | «service» | Logic | N/A |

**Total**: 12 objects (2 boundary, 7 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: HostelOwner, EmailService
- Boundary: MessagingInteraction, EmailServiceProxy
- Control: MessagingCoordinator
- Entity: MessageThread, Message, Booking, Guest, Property, MessageTemplate, Attachment
- Logic: MessageValidationService, BulkMessagingService

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

(No statechart needed - coordinator only)

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (5 existing, 4 new to add)
- [x] Control justified
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified

**Notes**: Need to update Phase 1 to add: Message, MessageThread, MessageTemplate, Attachment entities.
