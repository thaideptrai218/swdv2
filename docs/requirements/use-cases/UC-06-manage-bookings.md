# UC-06: Manage Bookings

| Field | Description |
|-------|-------------|
| **Use Case Name** | Manage Bookings |
| **Use Case ID** | UC-06 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Traveler views list of all bookings (upcoming, past, cancelled), accesses booking details, downloads confirmation, and contacts hostel. System retrieves and displays booking information organized by status. |
| **Dependency** | Include: None. Extend: UC-24 (Cancel Booking) at step 7.4, UC-07 (Submit Review) at step 7.5 |
| **Actors** | Primary: A3: Traveler. Secondary: None |
| **Preconditions** | Traveler authenticated. Traveler has at least one booking in system. System operational. |
| **Trigger** | Traveler clicks "My Bookings" link in navigation menu or user profile dropdown |
| **Main Sequence** | 1. Traveler navigates to bookings page<br>2. System retrieves all bookings for traveler<br>3. System displays bookings organized by tabs<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Tab: Upcoming Bookings (future check-in dates)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Tab: Past Bookings (past check-out dates)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Tab: Cancelled Bookings<br>4. For each booking, System shows summary card<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Hostel name and thumbnail photo<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Check-in and check-out dates<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Room type and number of guests<br>&nbsp;&nbsp;&nbsp;&nbsp;4.5. Total amount paid<br>&nbsp;&nbsp;&nbsp;&nbsp;4.6. Booking status (Confirmed, Checked-In, Completed, Cancelled)<br>5. Traveler selects specific booking to view details<br>6. System displays full booking details page<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Complete hostel information (name, address, phone, email)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Check-in/check-out dates and times<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Room type, bed configuration, number of guests<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Guest information provided at booking<br>&nbsp;&nbsp;&nbsp;&nbsp;6.5. Special requests (if any)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.6. Price breakdown and payment information<br>&nbsp;&nbsp;&nbsp;&nbsp;6.7. Cancellation policy<br>&nbsp;&nbsp;&nbsp;&nbsp;6.8. Booking timeline (booked date, payment date, check-in/out dates)<br>7. Traveler performs booking actions<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Option: Download booking confirmation (PDF)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Option: Get directions to hostel (opens map)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Option: Contact hostel (phone or email)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Option: Cancel booking (if within cancellation window) → UC-24<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. Option: Leave review (if past check-out date and not reviewed) → UC-07<br>8. If action selected, System executes corresponding operation<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Download: System generates PDF confirmation<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Directions: System opens map with hostel location<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Contact: System displays contact options (call, email, message)<br>9. Traveler returns to bookings list or navigates elsewhere |
| **Alternative Sequences** | **Step 2**: If traveler has no bookings, System displays "No bookings yet" message with link to search hostels<br><br>**Step 3**: If no bookings in specific tab (e.g., no cancelled bookings), System displays empty state message for that tab<br><br>**Step 7.1**: If PDF generation fails, System displays error message and offers option to retry or email confirmation<br><br>**Step 7.4**: If cancellation window expired, System hides cancel option and displays "Non-refundable" status<br><br>**Step 7.5**: If traveler already submitted review, System displays "You reviewed this booking" with link to view review<br><br>**Step 7.5**: If check-out date is future, System hides review option with message "Review available after check-out" |
| **Postconditions** | **Success**: Traveler views booking information. Actions completed as requested (PDF downloaded, contact initiated, etc.). Traveler can access booking details anytime.<br><br>**Failure**: Error message displayed for failed actions. Booking information remains accessible. |
| **Nonfunctional Requirements** | **Performance**: Bookings list loads within 2 seconds. Individual booking details display within 1 second. PDF generation completes within 5 seconds.<br><br>**Usability**: Bookings sorted chronologically within each tab (upcoming: nearest first, past: most recent first). Search/filter functionality for travelers with many bookings. Mobile-optimized layout. Quick actions accessible from summary cards. Pull-to-refresh on mobile.<br><br>**Accuracy**: Booking status updated in real-time. All information matches booking confirmation. Timestamps displayed in traveler's timezone.<br><br>**Availability**: Bookings accessible 24/7. Historical booking data retained indefinitely for traveler reference.<br><br>**Data Integrity**: Booking information immutable after confirmation (except status changes). Audit trail maintained for all booking modifications. |
| **Business Requirements** | BR-026: Travelers can access booking history indefinitely<br>BR-027: Cancel option visible only within cancellation window (24 hours)<br>BR-028: Review option available only after check-out date<br>BR-029: Booking confirmation PDF includes QR code for quick check-in<br>BR-030: Contact hostel feature records communication timestamp |
| **Frequency of Use** | High (travelers check bookings before trips and review past stays) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system send reminder notifications before check-in date?<br>2. Should traveler be able to modify booking dates directly (subject to availability)?<br>3. Should system display weather forecast for check-in dates?<br>4. Should traveler be able to add bookings to calendar (Google Calendar, iCal)?<br>5. Should system show nearby attractions or transportation options?<br>6. Should traveler be able to request early check-in or late check-out through system?<br>7. Should system allow travelers to share booking details with travel companions? |

## Related Use Cases

- **UC-05: Book Accommodation** — Creates bookings managed in this use case
- **UC-07: Submit Review** — Accessible from past bookings
- **UC-24: Cancel Booking** — Accessible from upcoming bookings within cancellation window
- **UC-16: Communicate With Guest** — Hostel owner may initiate communication about booking

## Notes

- Organized tabs (Upcoming/Past/Cancelled) improve usability for frequent travelers
- Quick access to booking details reduces support inquiries
- PDF confirmation useful for offline reference and check-in
- Contact hostel feature facilitates guest-property communication
- Review prompt from bookings page increases review submission rate
- Booking status tracking provides transparency throughout guest journey

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler accesses complete booking information and performs management actions

✓ **C2: Avoids functional decomposition** — Complete booking management sequence from list view to detailed actions

✓ **C3: Maintains black box view** — No mention of database queries, PDF libraries, storage systems; only describes booking retrieval and display behavior

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (manages bookings); Secondary: None (all operations internal to system)
