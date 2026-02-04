# Object Structuring: Manage Bookings

**Use Case**: docs/requirements/use-cases/UC-06-manage-bookings.md
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

**Analysis**: Traveler initiates by navigating to bookings page, browses booking list, selects specific booking, performs actions (download PDF, contact hostel).

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingManagementUI | «user interaction» | Maps from Traveler external user. Handles bookings list display (tabbed: Upcoming/Past/Cancelled), booking details view, action buttons (download, contact, cancel, review). |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: "System retrieves all bookings for traveler" → Booking entity
- Step 3-4: "Organized by tabs, shows summary cards" → Booking entity (status, dates)
- Step 4.1: "Hostel name and thumbnail" → Hostel, Photo entities
- Step 6: "Full booking details" → Booking, Hostel, RoomType entities
- Step 6.4: "Guest information" → User, UserProfile entities (or Booking guest data)
- Step 7.1: "Download confirmation PDF" → Booking data
- Precondition: "Traveler has at least one booking" → Booking entity
- Extends: UC-24 (Cancel Booking), UC-07 (Submit Review)

**Referenced Data**:
- **Booking**: Primary entity managed
  - Attributes: bookingId, checkInDate, checkOutDate, numberOfGuests, totalPrice, status, confirmationCode, bookingDate, guestInfo, specialRequests
- **Hostel**: Property reference
  - Attributes: hostelId, name, address, phone, email
- **RoomType**: Room details
  - Attributes: name, bedConfiguration
- **Photo**: Hostel thumbnail
  - Attributes: url
- **User**: Traveler (booking owner)
  - Attributes: userId, email, fullName
- **Review**: Check if reviewed (for showing review action)
  - Attributes: reviewId, bookingId

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Booking | Yes | ✅ | Primary entity retrieved (step 2), displayed (steps 3-6), actions performed on |
| Hostel | Yes | ✅ | Property information (steps 4.1, 6.1) |
| RoomType | Yes | ✅ | Room details (step 6.3) |
| Photo | Yes | ✅ | Hostel thumbnail (step 4.1) |
| User | Yes | ✅ | Booking owner, guest information |
| Review | Yes | ✅ | Check if booking already reviewed (step 7.5 condition) |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (bookings list, details, confirmation page)
- [x] Printer (PDF generation)
- [ ] External device
- [x] Same as input (reusing BookingManagementUI)

**Analysis**:
- Steps 3-6: Display bookings list and details
- Step 7.1: Generate PDF confirmation for download
- Step 7.2: Open map (could be Maps API or browser native)
- Step 7.3: Contact hostel (email/phone)
- Extends UC-24 and UC-07 (separate use cases)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingManagementUI | «user interaction» | Reused for output - displays booking list (tabbed), booking details, action results, empty states, error messages |
| PDFGenerator | «output» | Generates booking confirmation PDF (step 7.1) with QR code, booking details, hostel info |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Booking management is stateless display and retrieval:
- No sequential workflow states
- Actions are independent operations (download, contact, navigate to other UCs)
- Booking status is data attribute, not workflow state
- Tabbed organization is UI filtering, not state transitions
- Each action executes independently

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| BookingManagementCoordinator | «coordinator» | No | Coordinates booking retrieval: query bookings → filter by status → display organized list. Handles action dispatch: PDF generation, contact initiation, extension to UC-24/UC-07. Stateless coordination. |

---

## 5. Application Logic

**Complex Rules**:
- [ ] Multi-condition validation
- [x] Algorithms (booking organization, filtering, action availability)
- [ ] Cross-entity validation

**Identified Logic**:
- **Booking organization**: Filter by status (Upcoming: checkInDate > today, Past: checkOutDate ≤ today, Cancelled: status = Cancelled)
- **Sorting logic**: Upcoming (nearest first), Past (most recent first)
- **Action availability**: Cancel (within 24 hours), Review (after checkout and not reviewed)
- **PDF generation**: Booking confirmation with QR code for check-in
- **Timeline calculation**: Booking events (booked, payment, check-in, check-out)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BookingOrganizer | «business logic» | Organizes bookings by status and date: Upcoming (checkInDate > today, sorted nearest first), Past (checkOutDate ≤ today, sorted recent first), Cancelled (status filter). Handles empty state detection per tab. |
| ActionAvailabilityChecker | «business logic» | Determines which actions available for each booking: Cancel (within 24-hour window from booking date), Review (checkOutDate < today AND not reviewed), Contact (always available). Business rules: BR-027, BR-028. |
| BookingPDFService | «service» | Generates booking confirmation PDF with: booking details, hostel info, QR code (BR-029), price breakdown, cancellation policy. Handles PDF generation errors with retry. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| BookingManagementUI | «user interaction» | Boundary (I/O) | Traveler | Bookings list and details interface |
| PDFGenerator | «output» | Boundary (Output) | N/A | PDF confirmation generation |
| BookingManagementCoordinator | «coordinator» | Control | N/A | Booking retrieval orchestration (stateless) |
| Booking | «entity» | Entity | Booking | Reservation records |
| Hostel | «entity» | Entity | Hostel | Property information |
| RoomType | «entity» | Entity | RoomType | Room details |
| Photo | «entity» | Entity | Photo | Hostel thumbnails |
| User | «entity» | Entity | User | Traveler/guest data |
| Review | «entity» | Entity | Review | Review existence check |
| BookingOrganizer | «business logic» | Application Logic | N/A | Status-based organization and sorting |
| ActionAvailabilityChecker | «business logic» | Application Logic | N/A | Action availability rules |
| BookingPDFService | «service» | Application Logic | N/A | PDF generation |

**Total**: 12 objects (2 boundary, 6 entity, 1 control, 3 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: Traveler
- **Secondary Actors**: None (extends to UC-24, UC-07)
- **Boundary**: BookingManagementUI, PDFGenerator
- **Control**: BookingManagementCoordinator
- **Entity**: Booking, Hostel, RoomType, Photo, User, Review
- **Logic**: BookingOrganizer, ActionAvailabilityChecker, BookingPDFService

**Key Interactions**:
1. Traveler → BookingManagementUI: Navigate to bookings page
2. BookingManagementUI → BookingManagementCoordinator: Request bookings
3. BookingManagementCoordinator → Booking: Query all traveler bookings
4. BookingManagementCoordinator → BookingOrganizer: Organize by status
5. BookingOrganizer → Booking: Filter Upcoming (checkInDate > today)
6. BookingOrganizer → Booking: Filter Past (checkOutDate ≤ today)
7. BookingOrganizer → Booking: Filter Cancelled (status = Cancelled)
8. BookingManagementCoordinator → Hostel: Get hostel details per booking
9. BookingManagementCoordinator → Photo: Get hostel thumbnails
10. BookingManagementCoordinator → BookingManagementUI: Display organized tabs
11. Traveler → BookingManagementUI: Select specific booking
12. BookingManagementUI → BookingManagementCoordinator: Request booking details
13. BookingManagementCoordinator → Booking: Get full details
14. BookingManagementCoordinator → RoomType: Get room info
15. BookingManagementCoordinator → ActionAvailabilityChecker: Check available actions
16. ActionAvailabilityChecker → Booking: Check dates and status
17. ActionAvailabilityChecker → Review: Check if reviewed
18. BookingManagementCoordinator → BookingManagementUI: Display details with actions
19. Traveler → BookingManagementUI: Click "Download PDF"
20. BookingManagementUI → BookingManagementCoordinator: Generate PDF
21. BookingManagementCoordinator → BookingPDFService: Create PDF
22. BookingPDFService → Booking: Get booking data
23. BookingPDFService → Hostel: Get property info
24. BookingPDFService → PDFGenerator: Render PDF with QR code
25. PDFGenerator → Traveler: Download PDF file

**Extension Points**:
- Step 7.4: Cancel booking → UC-24 (if within cancellation window)
- Step 7.5: Leave review → UC-07 (if past checkout and not reviewed)

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - BookingManagementCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
  - BookingManagementUI ← Traveler
  - PDFGenerator ← N/A (output device)
- [x] Entities in Phase 1:
  - Booking ✅
  - Hostel ✅
  - RoomType ✅
  - Photo ✅
  - User ✅
  - Review ✅
- [x] Control justified (stateless coordinator for retrieval and display)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (organization rules, action availability, PDF generation)

---

## Alternative Flows Mapping

**Step 2 Alternative** (No bookings):
- BookingOrganizer finds zero bookings
- BookingManagementUI displays empty state "No bookings yet" with search link

**Step 3 Alternative** (Empty tab):
- BookingOrganizer filters produce zero results for specific tab
- BookingManagementUI displays tab-specific empty message

**Step 7.1 Alternative** (PDF generation fails):
- BookingPDFService encounters error
- BookingManagementCoordinator receives error
- BookingManagementUI displays error with retry option or email alternative

**Step 7.4 Alternative** (Cancellation window expired):
- ActionAvailabilityChecker detects booking date + 24 hours < now
- BookingManagementUI hides cancel button, displays "Non-refundable"

**Step 7.5 Alternative** (Already reviewed):
- ActionAvailabilityChecker queries Review entity
- Review found for booking
- BookingManagementUI displays "You reviewed this booking" with link

**Step 7.5 Alternative** (Checkout date future):
- ActionAvailabilityChecker detects checkOutDate > today
- BookingManagementUI hides review option, shows "Review available after check-out"

---

## Extension Relationships

**UC-24: Cancel Booking** (extends at step 7.4):
- Condition: ActionAvailabilityChecker confirms within 24-hour window
- Trigger: Traveler clicks "Cancel Booking" button
- BookingManagementCoordinator delegates to UC-24 CancellationControl
- After cancellation: Returns to BookingManagementUI with updated status

**UC-07: Submit Review** (extends at step 7.5):
- Condition: ActionAvailabilityChecker confirms checkout passed and not reviewed
- Trigger: Traveler clicks "Leave Review" button
- BookingManagementCoordinator navigates to UC-07 review form
- After review submission: Returns to BookingManagementUI

---

## Performance Considerations

- **List load**: ≤ 2 seconds
- **Details view**: ≤ 1 second
- **PDF generation**: ≤ 5 seconds
- **Sorting/filtering**: Client-side for small lists (<100), server-side for large
- **Mobile pull-to-refresh**: Real-time status updates
- **Pagination**: If traveler has >50 bookings

---

## Notes

- **Stateless design**: Each view/action independent, no workflow states
- **Tabbed organization**: Improves UX for frequent travelers
- **Action availability**: Dynamic based on business rules (BR-027, BR-028, BR-029)
- **PDF with QR code**: Facilitates quick check-in (BR-029)
- **Extension points**: Clean delegation to UC-24 and UC-07
- **Historical data**: Bookings retained indefinitely (BR-026)
- **Empty states**: Helpful guidance when no bookings in tabs
- **Contact tracking**: Communication timestamp recorded (BR-030)
