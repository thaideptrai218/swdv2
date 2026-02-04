# Object Structuring: Book Accommodation

**Use Case**: docs/requirements/use-cases/UC-05-book-accommodation.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Primary Actor**: Traveler
**External Class (Phase 1)**: Traveler («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Analysis**: Traveler initiates booking via "Book Now" button, enters guest information, selects payment method through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingUI | «user interaction» | Maps from Traveler external user. Handles booking form display, guest information input, payment method selection, confirmation display. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 1: "System verifies traveler authentication" → User entity
- Step 3: "Selected hostel, room type, dates" → Hostel, RoomType entities
- Step 5: "Guest information" → User entity (profile data) + Booking entity (guest details)
- Step 8: "Check room availability" → RoomType, Booking entities
- Step 9: "Creates pending booking" → Booking entity
- Step 13: "System confirms booking" → Booking entity (status update)
- Step 14-15: "Send confirmation emails" → Notification entity
- Step 12: "Payment confirmation" → Payment entity
- Precondition: "Traveler authenticated" → User entity
- Postcondition: "Booking confirmed and saved" → Booking entity
- For dorm rooms: Bed assignment (Bed entity)

**Referenced Data**:
- **User**: Authentication and guest data
  - Attributes: userId, email, fullName
- **Hostel**: Property being booked
  - Attributes: hostelId, name, address
- **RoomType**: Room being booked
  - Attributes: roomTypeId, name, pricePerNight, availableRooms
- **Bed**: Individual bed assignment (dorm rooms only)
  - Attributes: bedId, bedNumber, status
- **Booking**: Reservation record
  - Attributes: bookingId, checkInDate, checkOutDate, numberOfGuests, totalPrice, status, confirmationCode
- **Payment**: Transaction record
  - Attributes: paymentId, amount, paymentMethod, transactionId, status
- **Notification**: Email confirmations
  - Attributes: recipientId, type, subject, content

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| User | Yes | ✅ | Authentication check (step 1), guest data source (step 5), booking owner |
| Hostel | Yes | ✅ | Property reference (step 3), notification recipient (step 15) |
| RoomType | Yes | ✅ | Room reference (step 3), availability check (step 8), pricing (step 3.2) |
| Bed | Yes | ✅ | Bed assignment for dorm bookings (implicit in room allocation) |
| Booking | Yes | ✅ | Created (step 9), confirmed (step 13), core transaction entity |
| Payment | Yes | ✅ | Created and processed (step 11-12), linked to booking |
| Notification | Yes | ✅ | Confirmation emails (steps 14-15) |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (booking form, confirmation page)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing BookingUI)

**Analysis**:
- Step 10: System redirects to Payment Gateway (external system)
- Steps 14-15: System sends emails via Email Service
- Step 16: System displays confirmation page
- Includes UC-02 if authentication needed

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingUI | «user interaction» | Reused for output - displays booking form, price breakdown, error messages, payment redirect, confirmation page with booking reference |
| PaymentGatewayProxy | «proxy» | Maps from PaymentGateway external system. Handles payment redirect (step 10), payment processing coordination (step 11), payment confirmation receipt (step 12) |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends confirmation email to traveler (step 14) and booking notification to owner (step 15) |

**Note on UC-02 Include**:
- Step 2: "Include: UC-02 (Log In) if not authenticated"
- This is handled by AuthenticationControl from UC-02
- Not a separate boundary object here, but referenced in interactions

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Booking process has clear state-dependent behavior:
- **PreBooking** state: Verifying authentication
- **FormEntry** state: Collecting guest information
- **PendingPayment** state: Awaiting payment completion (10-minute timeout)
- **PaymentProcessing** state: Payment being processed
- **Confirmed** state: Booking successful
- **Failed** state: Payment failed or availability lost
- State transitions based on payment outcome (success/failure/timeout)
- Different behavior based on availability status (step 8 alternative)
- Timeout handling (10-minute payment window)

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| BookingControl | «state-dependent control» | ⚠️ Yes | Orchestrates booking workflow through states: authentication → form entry → availability validation → pending booking → payment processing → confirmation/failure. Handles 10-minute payment timeout. Manages state transitions based on payment gateway responses and availability changes. |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (guest info format, phone validation)
- [x] Algorithms (price calculation with taxes/fees)
- [x] Cross-entity validation (real-time availability check, race condition prevention)

**Identified Logic**:
- **Guest information validation**: Phone format, name validation, arrival time format
- **Availability validation**: Real-time check to prevent double-booking race conditions
- **Price calculation**: (pricePerNight × nights) + taxes + fees, breakdown display
- **Booking reference generation**: Unique confirmation code with date/property identifiers
- **Payment coordination**: Redirect to gateway, status tracking, timeout handling
- **Bed assignment**: For dorm rooms, assign specific beds

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| GuestInfoValidator | «business logic» | Validates guest information: phone number format, name validation, arrival time format. Ensures data quality before booking creation. |
| AvailabilityValidator | «business logic» | Real-time availability check at step 8 to prevent race conditions. Validates room still available between form display and payment. Critical for booking integrity. Reused from UC-03 AvailabilityCalculator. |
| PriceCalculator | «business logic» | Calculates total price with breakdown: room rate × nights, taxes, payment processing fees. Ensures price transparency (BR-024). Reused from UC-03. |
| BookingReferenceGenerator | «service» | Generates unique confirmation codes including date/property identifiers (BR-025). Non-guessable format for security. |
| PaymentCoordinator | «service» | Manages payment gateway interaction: redirect URL generation, payment status polling, timeout handling (10 minutes), confirmation processing. Ensures idempotent payment (prevents duplicate charges). |
| BedAssignmentService | «service» | For dorm room bookings, assigns specific beds. Updates Bed entity status (Available → Occupied). Handles bed preferences if specified. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| BookingUI | «user interaction» | Boundary (I/O) | Traveler | Booking form and confirmation interface |
| PaymentGatewayProxy | «proxy» | Boundary (Output) | PaymentGateway | Payment processing integration |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Confirmation email delivery |
| BookingControl | «state-dependent control» | Control | N/A | Booking workflow orchestration with state management |
| User | «entity» | Entity | User | Authentication and guest data |
| Hostel | «entity» | Entity | Hostel | Property reference |
| RoomType | «entity» | Entity | RoomType | Room reference and availability |
| Bed | «entity» | Entity | Bed | Bed assignment (dorm rooms) |
| Booking | «entity» | Entity | Booking | Reservation record |
| Payment | «entity» | Entity | Payment | Transaction record |
| Notification | «entity» | Entity | Notification | Email confirmations |
| GuestInfoValidator | «business logic» | Application Logic | N/A | Guest data validation |
| AvailabilityValidator | «business logic» | Application Logic | N/A | Real-time availability check |
| PriceCalculator | «business logic» | Application Logic | N/A | Price calculation with breakdown |
| BookingReferenceGenerator | «service» | Application Logic | N/A | Unique confirmation code generation |
| PaymentCoordinator | «service» | Application Logic | N/A | Payment gateway coordination |
| BedAssignmentService | «service» | Application Logic | N/A | Bed assignment for dorm bookings |

**Total**: 17 objects (3 boundary, 7 entity, 1 control, 6 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: Traveler
- **Secondary Actors**: PaymentGateway, EmailService
- **Boundary**: BookingUI, PaymentGatewayProxy, EmailServiceProxy
- **Control**: BookingControl (+ AuthenticationControl from UC-02 if needed)
- **Entity**: User, Hostel, RoomType, Bed, Booking, Payment, Notification
- **Logic**: GuestInfoValidator, AvailabilityValidator, PriceCalculator, BookingReferenceGenerator, PaymentCoordinator, BedAssignmentService

**Key Interactions (Main Flow)**:
1. Traveler → BookingUI: Click "Book Now"
2. BookingUI → BookingControl: Initiate booking
3. BookingControl → User: Verify authentication
4. [If not authenticated: Include UC-02 login flow]
5. BookingControl → PriceCalculator: Calculate total price
6. PriceCalculator → RoomType: Get price per night
7. BookingControl → BookingUI: Display booking form with price breakdown
8. Traveler → BookingUI: Enter guest information
9. BookingUI → BookingControl: Submit guest details
10. BookingControl → GuestInfoValidator: Validate guest information
11. BookingControl → AvailabilityValidator: Revalidate availability
12. AvailabilityValidator → RoomType: Check availableRooms
13. AvailabilityValidator → Booking: Check date conflicts
14. BookingControl → BookingReferenceGenerator: Generate confirmation code
15. BookingControl → Booking: Create pending booking
16. [If dorm room: BookingControl → BedAssignmentService → Bed: Assign beds]
17. BookingControl → PaymentCoordinator: Initiate payment
18. PaymentCoordinator → PaymentGatewayProxy: Create payment session
19. PaymentGatewayProxy → PaymentGateway: Redirect for payment
20. Traveler → PaymentGateway: Complete payment
21. PaymentGateway → PaymentGatewayProxy: Payment confirmation
22. PaymentGatewayProxy → PaymentCoordinator: Relay confirmation
23. PaymentCoordinator → Payment: Create payment record
24. BookingControl → Booking: Confirm booking (status = Confirmed)
25. BookingControl → RoomType: Update availableRooms
26. BookingControl → Notification: Create traveler confirmation email
27. BookingControl → EmailServiceProxy: Send traveler email
28. EmailServiceProxy → EmailService: Deliver email
29. BookingControl → Notification: Create owner notification email
30. BookingControl → EmailServiceProxy: Send owner email
31. BookingControl → BookingUI: Display confirmation page

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: BookingControl

**States Identified**:
1. **VerifyingAuth**: Checking traveler authentication
2. **DisplayingForm**: Showing booking form with pre-filled data
3. **CollectingGuestInfo**: Awaiting guest information submission
4. **ValidatingInfo**: Validating guest information format
5. **CheckingAvailability**: Real-time availability validation
6. **CreatingPendingBooking**: Creating temporary booking record
7. **PendingPayment**: Waiting for payment completion (10-minute timeout)
8. **ProcessingPayment**: Payment being processed by gateway
9. **ConfirmingBooking**: Booking confirmed, sending notifications
10. **Completed**: Booking successful, confirmation displayed
11. **AvailabilityLost**: Room no longer available (race condition)
12. **PaymentFailed**: Payment declined or failed
13. **PaymentTimeout**: Payment took longer than 10 minutes
14. **ValidationFailed**: Guest information invalid

**Events**:
- initiateBooking
- authenticationVerified / authenticationRequired (→ UC-02)
- guestInfoSubmitted
- validationSuccess / validationFailed
- availabilityConfirmed / availabilityLost
- pendingBookingCreated
- paymentRedirect
- paymentSuccess / paymentFailed / paymentTimeout
- bookingConfirmed
- emailsSent
- displayConfirmation

---

## Validation

- [x] At least 1 boundary object (3 identified)
- [x] Boundary → External mapping (1:1):
  - BookingUI ← Traveler
  - PaymentGatewayProxy ← PaymentGateway
  - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
  - User ✅
  - Hostel ✅
  - RoomType ✅
  - Bed ✅
  - Booking ✅
  - Payment ✅
  - Notification ✅
- [x] Control justified (state-dependent with payment states, timeout, availability states)
- [x] State-dependent flagged (BookingControl)
- [x] Logic justified (complex validation, availability, payment coordination, bed assignment)

---

## Alternative Flows Mapping

**Step 1 Alternative** (Not authenticated):
- BookingControl checks User authentication
- If not authenticated: Include UC-02 (Log In)
- AuthenticationControl from UC-02 handles login flow
- After successful login, returns to BookingControl
- BookingControl resumes at DisplayingForm state

**Step 7 Alternative** (Invalid guest info):
- GuestInfoValidator detects format errors (phone, name)
- BookingControl → ValidationFailed state
- BookingUI displays specific error messages
- Returns to CollectingGuestInfo state

**Step 8 Alternative** (Room unavailable):
- AvailabilityValidator finds room booked by another traveler
- BookingControl → AvailabilityLost state
- BookingControl deletes pending booking
- BookingUI displays "Room no longer available" with alternatives
- Offers different dates or room types

**Step 12 Alternative** (Payment failed):
- PaymentGateway returns failure (insufficient funds, declined)
- PaymentCoordinator receives failure event
- BookingControl → PaymentFailed state
- BookingControl deletes pending booking
- BookingUI displays error with retry option
- Can choose different payment method

**Step 12 Alternative** (Payment timeout):
- If payment processing > 10 minutes
- BookingControl → PaymentTimeout state
- BookingUI displays "Payment pending" status
- When payment eventually confirms: BookingControl → ConfirmingBooking
- Sends confirmation email when confirmed

**Step 14 Alternative** (Email delivery fails):
- EmailServiceProxy receives failure from EmailService
- BookingControl logs error but continues (booking still confirmed)
- BookingUI displays warning on confirmation page
- Offers "Resend email" button

---

## Include Relationship: UC-02 (Log In)

**Integration Point**: Step 1-2
- BookingControl checks authentication
- If User not authenticated:
  - BookingControl delegates to AuthenticationControl (UC-02)
  - AuthenticationControl handles complete login flow
  - On success: Returns to BookingControl
  - On failure: Booking cancelled
- Booking form preserves selected hostel, room, dates during login

---

## Security Considerations

- **Authentication required**: Prevents anonymous bookings, reduces fraud
- **PCI DSS compliance**: Payment data handled exclusively by PaymentGateway
- **HTTPS only**: All booking data encrypted in transit
- **Unique booking references**: Non-guessable confirmation codes
- **Idempotent payment**: PaymentCoordinator prevents duplicate charges
- **Audit trail**: All booking state transitions logged
- **Race condition prevention**: Real-time availability check at step 8
- **Sensitive data**: Guest phone, email encrypted at rest

---

## Business Rules

- **BR-021**: Full prepayment required (enforced by BookingControl)
- **BR-022**: Confirmation email within 5 minutes (EmailServiceProxy SLA)
- **BR-023**: 24-hour cancellation window (UC-24 Cancel Booking)
- **BR-024**: Payment fees disclosed (PriceCalculator breakdown)
- **BR-025**: Unique booking reference (BookingReferenceGenerator format)

---

## Performance Considerations

- **Booking form load**: ≤ 2 seconds
- **Payment redirect**: ≤ 3 seconds
- **Confirmation page**: ≤ 5 seconds after payment
- **Email delivery**: Initiated within 10 seconds
- **Confirmation email**: Delivered within 5 minutes
- **Payment timeout**: 10-minute window
- **Transaction logging**: All state transitions logged for audit

---

## Notes

- **Authentication inclusion**: UC-02 included if traveler not logged in
- **Payment Gateway integration**: PCI compliance without handling card data
- **Real-time availability**: Step 8 prevents double-booking race conditions
- **Confirmation emails**: Critical for guest confidence and owner coordination
- **State management**: BookingControl maintains complex workflow states
- **Reusability**: AvailabilityValidator and PriceCalculator reused from UC-03
- **Bed assignment**: BedAssignmentService handles dorm-specific logic
- **Revenue-generating**: Primary conversion point, highly optimized
- **Cancellation policy**: 24-hour window (BR-023) extends to UC-24
