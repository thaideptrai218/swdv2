# Object Structuring: Process Booking

**Use Case**: docs/requirements/use-cases/UC-12-process-booking.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Primary Actor**: Hostel Owner
**External Class (Phase 1)**: HostelOwner («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Analysis**: Owner initiates by accessing bookings dashboard, views bookings organized by status tabs, selects specific booking, confirms/declines, assigns beds, adds notes, communicates with guests through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingProcessingUI | «user interaction» | Maps from HostelOwner external user. Handles bookings dashboard with tabbed views (New/Confirmed/Checked-In/Completed/Cancelled), booking details display, bed assignment interface with room layout visualization, message compose form, status update actions. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2-3: "Retrieves all bookings for property" → Booking entity
- Step 4: "Summary card" → Booking, User (guest), RoomType entities
- Step 6: "Comprehensive booking details" → Booking, User, RoomType, Payment entities
- Step 7: "Confirms booking" → Booking entity (status update)
- Step 8: "Assigns beds" → Bed entity, Booking entity (bed assignment)
- Step 9: "Internal notes" → Booking entity (notes attribute) or separate Note entity
- Step 10: "Communicates with guest" → Notification entity (message history)
- Step 7.4-7.5: "Sends notifications" → Notification entity
- Alternative: "Decline booking, refund" → Booking entity (status), Payment entity (refund)

**Referenced Data**:
- **Booking**: Primary entity being processed
  - Attributes: bookingId, checkInDate, checkOutDate, numberOfGuests, totalPrice, status, confirmationCode, specialRequests, internalNotes
- **User**: Guest information (traveler)
  - Attributes: userId, fullName, email, phoneNumber, nationality
- **RoomType**: Room details
  - Attributes: roomTypeId, name, bedConfiguration
- **Bed**: Bed assignment (dorm bookings)
  - Attributes: bedId, bedNumber, status
- **Payment**: Payment information
  - Attributes: paymentId, amount, paymentMethod, transactionId, status
- **Notification**: Email/SMS communications
  - Attributes: notificationId, recipientId, type, content, sentDate
- **Hostel**: Property reference
  - Attributes: hostelId, ownerId

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Booking | Yes | ✅ | Primary entity processed (status updates, bed assignment, notes) |
| User | Yes | ✅ | Guest information display |
| RoomType | Yes | ✅ | Room details display |
| Bed | Yes | ✅ | Bed assignment for dorms (step 8) |
| Payment | Yes | ✅ | Payment information display, refunds (decline flow) |
| Notification | Yes | ✅ | Email/SMS confirmations, communications (steps 7.4-7.5, 10) |
| Hostel | Yes | ✅ | Property reference for owner's bookings |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (dashboard, details, confirmation messages)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing BookingProcessingUI)

**Analysis**:
- Steps 3-6, 16: Display dashboard, booking details, success messages
- Steps 7.4-7.5, 10: Send email/SMS via Email Service and SMS Gateway
- Alternative: Refund via Payment Gateway

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingProcessingUI | «user interaction» | Reused for output - displays tabbed dashboard, booking details, bed assignment interface (color-coded: green=available, red=occupied, blue=selected), validation errors, success messages, message history |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends confirmation emails (step 7.4), booking notifications (step 15), guest messages (step 10.4). Reused from multiple UCs. |
| SMSGatewayProxy | «proxy» | Maps from SMSGateway external system. Sends urgent SMS notifications (step 7.5): new bookings, cancellations, check-in within 24h (BR-044). Real-time alerts for owner. |
| PaymentGatewayProxy | «proxy» | Maps from PaymentGateway external system. Processes refunds when owner declines booking (alternative step 7). Handles payment reversal. Reused from UC-05. |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Booking processing is stateless view and update operations:
- Tabbed organization is UI filtering by Booking.status attribute, not workflow states
- Status updates (New → Confirmed → Checked-In → Completed) are data changes, not control flow
- Confirmation, bed assignment, notes are independent operations
- Each action updates booking data without sequential workflow dependency
- No complex state transitions or timeout handling required

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| BookingProcessingCoordinator | «coordinator» | No | Coordinates booking management: retrieves bookings → filters by status → displays organized dashboard → handles confirmation/decline → assigns beds → sends notifications. Stateless orchestration of booking operations. |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (bed availability, booking conflicts)
- [x] Algorithms (bed assignment, status organization, urgency sorting)
- [x] Cross-entity validation (room availability revalidation)

**Identified Logic**:
- **Booking organization**: Filter and sort by status (New/Confirmed/Checked-In/Completed/Cancelled), urgency (check-in proximity)
- **Room availability revalidation**: Recheck when confirming to prevent double-booking race conditions
- **Bed assignment**: Validate bed availability, prevent conflicts, update Bed.status
- **Notification rules**: Email + SMS for specific events (BR-044: urgent events trigger SMS)
- **Refund processing**: Calculate refund amount, initiate via Payment Gateway
- **24-hour confirmation**: Track time since booking creation (BR-042)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingOrganizer | «business logic» | Organizes bookings by status and urgency: Filters by status (New/Confirmed/Checked-In/Completed/Cancelled). Sorts by check-in proximity (urgent bookings first). Reused from UC-06 with owner-specific filtering. |
| AvailabilityRevalidator | «business logic» | Revalidates room availability when confirming booking (step 7.2). Prevents double-booking race conditions. Checks RoomType.availableRooms and Bed.status. Critical for booking integrity. Similar to UC-05 AvailabilityValidator. |
| BedAssignmentManager | «service» | Manages bed assignment for dorm bookings (step 8): Displays room layout with color-coded beds (green=available, red=occupied, blue=selected). Validates bed availability (prevents race conditions). Updates Bed.status = Occupied. Saves bed assignment to Booking. Required before check-in (BR-046). |
| BookingNotificationService | «service» | Manages booking notifications: Confirmation emails/SMS (steps 7.4-7.5) within 5 minutes (BR-043). Urgent SMS for new bookings, cancellations, check-in within 24h (BR-044). Message delivery tracking. Coordinates EmailServiceProxy and SMSGatewayProxy. |
| RefundProcessor | «service» | Processes refunds when owner declines booking (alternative step 7): Validates decline reason required (BR-045). Coordinates with PaymentGatewayProxy for full refund. Updates Booking.status = Cancelled. Sends cancellation notification with refund details. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| BookingProcessingUI | «user interaction» | Boundary (I/O) | HostelOwner | Booking processing interface |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Email notifications |
| SMSGatewayProxy | «proxy» | Boundary (Output) | SMSGateway | Urgent SMS alerts |
| PaymentGatewayProxy | «proxy» | Boundary (Output) | PaymentGateway | Refund processing |
| BookingProcessingCoordinator | «coordinator» | Control | N/A | Booking operations orchestration (stateless) |
| Booking | «entity» | Entity | Booking | Booking records |
| User | «entity» | Entity | User | Guest information |
| RoomType | «entity» | Entity | RoomType | Room details |
| Bed | «entity» | Entity | Bed | Bed assignment |
| Payment | «entity» | Entity | Payment | Payment info and refunds |
| Notification | «entity» | Entity | Notification | Communications log |
| Hostel | «entity» | Entity | Hostel | Property reference |
| BookingOrganizer | «business logic» | Application Logic | N/A | Status organization and urgency sorting |
| AvailabilityRevalidator | «business logic» | Application Logic | N/A | Room availability recheck |
| BedAssignmentManager | «service» | Application Logic | N/A | Dorm bed assignment |
| BookingNotificationService | «service» | Application Logic | N/A | Email/SMS notifications |
| RefundProcessor | «service» | Application Logic | N/A | Decline and refund processing |

**Total**: 16 objects (4 boundary, 7 entity, 1 control, 4 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actors**: EmailService, SMSGateway, PaymentGateway
- **Boundary**: BookingProcessingUI, EmailServiceProxy, SMSGatewayProxy, PaymentGatewayProxy
- **Control**: BookingProcessingCoordinator
- **Entity**: Booking, User, RoomType, Bed, Payment, Notification, Hostel
- **Logic**: BookingOrganizer, AvailabilityRevalidator, BedAssignmentManager, BookingNotificationService, RefundProcessor

**Key Interactions (Confirm Booking)**:
1. HostelOwner → BookingProcessingUI: Navigate to bookings dashboard
2. BookingProcessingUI → BookingProcessingCoordinator: Request bookings
3. BookingProcessingCoordinator → Booking: Query owner's property bookings
4. BookingProcessingCoordinator → BookingOrganizer: Organize by status
5. BookingOrganizer → Booking: Filter by status, sort by urgency
6. BookingProcessingCoordinator → User: Get guest info
7. BookingProcessingCoordinator → RoomType: Get room details
8. BookingProcessingCoordinator → BookingProcessingUI: Display dashboard
9. HostelOwner → BookingProcessingUI: Select booking, click "Confirm"
10. BookingProcessingUI → BookingProcessingCoordinator: Process confirmation
11. BookingProcessingCoordinator → AvailabilityRevalidator: Validate availability
12. AvailabilityRevalidator → RoomType: Check availableRooms
13. BookingProcessingCoordinator → Booking: Update status = "Confirmed"
14. BookingProcessingCoordinator → BookingNotificationService: Send confirmations
15. BookingNotificationService → Notification: Create email record
16. BookingNotificationService → EmailServiceProxy: Send confirmation email
17. EmailServiceProxy → EmailService: Deliver email
18. BookingNotificationService → Notification: Create SMS record
19. BookingNotificationService → SMSGatewayProxy: Send SMS
20. SMSGatewayProxy → SMSGateway: Deliver SMS
21. BookingProcessingCoordinator → BookingProcessingUI: Display success

**Bed Assignment Flow** (Dorm bookings):
1. HostelOwner → BookingProcessingUI: View booking, access bed assignment
2. BookingProcessingUI → BedAssignmentManager: Load bed interface
3. BedAssignmentManager → Bed: Query available beds for roomType
4. BedAssignmentManager → BookingProcessingUI: Display room layout (color-coded)
5. HostelOwner → BookingProcessingUI: Select bed(s)
6. BookingProcessingUI → BedAssignmentManager: Save assignment
7. BedAssignmentManager → Bed: Validate availability (prevent race condition)
8. BedAssignmentManager → Bed: Update status = Occupied
9. BedAssignmentManager → Booking: Save bed assignment (bed numbers)

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - BookingProcessingCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (4 identified)
- [x] Boundary → External mapping (1:1):
  - BookingProcessingUI ← HostelOwner
  - EmailServiceProxy ← EmailService
  - SMSGatewayProxy ← SMSGateway
  - PaymentGatewayProxy ← PaymentGateway
- [x] Entities in Phase 1:
  - Booking ✅
  - User ✅
  - RoomType ✅
  - Bed ✅
  - Payment ✅
  - Notification ✅
  - Hostel ✅
- [x] Control justified (stateless coordinator for booking operations)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (organization, availability recheck, bed assignment, notifications, refunds)

---

## Alternative Flows Mapping

**Step 7 Alternative** (Owner declines booking):
- HostelOwner clicks "Decline Booking"
- BookingProcessingCoordinator → RefundProcessor: Process decline
- RefundProcessor validates decline reason required (BR-045)
- RefundProcessor → Booking: Update status = Cancelled
- RefundProcessor → PaymentGatewayProxy: Initiate full refund
- PaymentGatewayProxy → PaymentGateway: Process refund
- RefundProcessor → Payment: Update status = Refunded
- RefundProcessor → BookingNotificationService: Send cancellation notification
- Notification includes reason and refund details

**Step 7.2 Alternative** (Room no longer available):
- AvailabilityRevalidator → RoomType: Check availableRooms
- Detects double-booking (race condition)
- BookingProcessingCoordinator receives error
- BookingProcessingUI displays "Room no longer available"
- Owner must decline booking with refund

**Step 8.4 Alternative** (Bed already assigned):
- BedAssignmentManager → Bed: Validate selection
- Detects bed.status = Occupied (race condition)
- BookingProcessingUI displays "Bed no longer available"
- Highlights remaining available beds
- Owner selects different bed

**No-Show Flow**:
- Check-in time passes without guest arrival
- HostelOwner marks booking as "No-Show"
- BookingProcessingCoordinator → Booking: Update status
- BookingNotificationService sends no-show notification
- No refund (per policy)
- BedAssignmentManager → Bed: Update status = Available (release bed)

**Early Check-In Request** / **Modification Request**: Handled via messaging (step 10)

**Urgent Notification Flow** (BR-044):
- For bookings within 24h: BookingNotificationService sends SMS reminder
- For new bookings: Immediate SMS alert
- For cancellations: Urgent SMS notification
- SMSGatewayProxy delivers within 30 seconds

---

## Business Rules

- **BR-042**: Confirmation required within 24 hours (BookingOrganizer urgency sorting)
- **BR-043**: Confirmation notification within 5 minutes (BookingNotificationService SLA)
- **BR-044**: SMS for urgent events (BookingNotificationService logic)
- **BR-045**: Decline requires reason and full refund (RefundProcessor enforces)
- **BR-046**: Bed assignment required before check-in (BedAssignmentManager enforces)
- **BR-047**: Internal notes private to staff (Booking.internalNotes attribute, not visible to guests)

---

## Notes

- **Stateless design**: Each action independent, no workflow states
- **Tabbed organization**: UI filtering by Booking.status attribute
- **Multi-channel notifications**: Email + SMS ensures owner awareness
- **Bed assignment**: Unique hostel feature (dorm-level granularity)
- **Race condition prevention**: AvailabilityRevalidator and BedAssignmentManager prevent double-booking
- **Internal notes**: Staff coordination for special requests
- **Real-time sync**: Prevents conflicts across multiple staff devices
- **Urgency sorting**: Near check-in dates prioritized
- **Communication history**: Reduces misunderstandings
- **Refund automation**: RefundProcessor handles decline workflow
