# UC-19: View Booking Details

| Field | Description |
|-------|-------------|
| **Use Case Name** | View Booking Details |
| **Use Case ID** | UC-19 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Hostel Staff views comprehensive booking information including guest details, booking dates, room assignment, payment status, and communication history. System retrieves and displays all relevant booking data for staff reference. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A2a: Hostel Staff (generalized actor covering Receptionist and Owner). Secondary: None |
| **Preconditions** | Staff authenticated with booking view permissions. Booking exists for property. System operational. |
| **Trigger** | Staff clicks on booking from dashboard, searches for booking, or scans booking reference barcode/QR code |
| **Main Sequence** | 1. Staff navigates to booking search or dashboard<br>2. Staff searches for booking<br>&nbsp;&nbsp;&nbsp;&nbsp;2.1. Staff enters guest name, booking reference, email, or phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;2.2. Alternatively: Staff scans booking confirmation QR code<br>3. System retrieves matching bookings<br>4. System displays search results (if multiple matches)<br>5. Staff selects specific booking<br>6. System displays comprehensive booking details page<br>7. **Guest Information Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Full name<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Email address<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Nationality<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. Number of past bookings at property<br>&nbsp;&nbsp;&nbsp;&nbsp;7.6. Emergency contact (if checked in)<br>8. **Booking Details Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Booking status (Confirmed, Checked-In, Completed, Cancelled)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Booked date and time<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Check-in date (scheduled and actual)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.5. Check-out date (scheduled and actual)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.6. Number of nights<br>&nbsp;&nbsp;&nbsp;&nbsp;8.7. Number of guests<br>9. **Room/Bed Assignment Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Room type<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Specific room number (if assigned)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Bed number(s) for dorm bookings (if assigned)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Special requests from guest<br>10. **Payment Information Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Total amount<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Amount paid<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Outstanding balance (if any)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.4. Payment method used<br>&nbsp;&nbsp;&nbsp;&nbsp;10.5. Transaction ID<br>&nbsp;&nbsp;&nbsp;&nbsp;10.6. Payment date and time<br>&nbsp;&nbsp;&nbsp;&nbsp;10.7. Refund information (if applicable)<br>11. **Communication History Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. All messages between owner and guest<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Automated system messages sent<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Timestamps for each communication<br>12. **Activity Timeline Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;12.1. Booking created<br>&nbsp;&nbsp;&nbsp;&nbsp;12.2. Payment received<br>&nbsp;&nbsp;&nbsp;&nbsp;12.3. Booking confirmed<br>&nbsp;&nbsp;&nbsp;&nbsp;12.4. Bed/room assigned<br>&nbsp;&nbsp;&nbsp;&nbsp;12.5. Guest checked in (with timestamp)<br>&nbsp;&nbsp;&nbsp;&nbsp;12.6. Guest checked out (with timestamp)<br>&nbsp;&nbsp;&nbsp;&nbsp;12.7. Any modifications or cancellations<br>13. **Staff Notes Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. Internal notes added by staff (not visible to guest)<br>&nbsp;&nbsp;&nbsp;&nbsp;13.2. Note author and timestamp<br>14. **Quick Actions Panel**<br>&nbsp;&nbsp;&nbsp;&nbsp;14.1. Contact Guest button<br>&nbsp;&nbsp;&nbsp;&nbsp;14.2. Check In button (if confirmed and date appropriate)<br>&nbsp;&nbsp;&nbsp;&nbsp;14.3. Check Out button (if checked in)<br>&nbsp;&nbsp;&nbsp;&nbsp;14.4. Modify Booking button (if permissions allow)<br>&nbsp;&nbsp;&nbsp;&nbsp;14.5. Print Booking button<br>15. Staff reviews information as needed<br>16. Staff performs action from quick actions or navigates elsewhere |
| **Alternative Sequences** | **Step 3**: If no bookings found, System displays "No bookings found matching your search" with suggestions to try different search criteria<br><br>**Step 3**: If search is ambiguous (common name), System displays list of all matches with key distinguishing details (dates, booking reference) for staff to select correct booking<br><br>**QR Code Scan Flow**:<br>- Staff uses device camera or barcode scanner<br>- Scans booking confirmation QR code (from guest's email or phone)<br>- System decodes QR code to extract booking reference<br>- System retrieves booking directly (skips search results)<br>- System displays booking details<br><br>**Print Booking**:<br>- Staff clicks "Print Booking" button<br>- System generates printable booking summary<br>- Includes: guest info, dates, room assignment, payment status, QR code<br>- Staff prints for physical records or guest copy<br><br>**Add Staff Note**:<br>- Staff clicks "Add Note" button<br>- Staff enters note text (e.g., "Guest requested quiet room", "VIP - owner's friend")<br>- Staff saves note<br>- System adds note to booking with staff name and timestamp<br>- Note visible to all staff, not to guest<br><br>**Contact Guest from Details**:<br>- Staff clicks "Contact Guest" button<br>- System opens message compose interface<br>- Guest email pre-filled<br>- Booking context automatically included<br>- Staff sends message (continues to UC-16)<br><br>**Quick Check-In from Details**:<br>- Staff viewing booking on check-in day<br>- Guest arrives at desk<br>- Staff clicks "Check In" button from details page<br>- System navigates to check-in form with data pre-filled (continues to UC-17)<br><br>**Mobile View Optimization**:<br>- Staff viewing on mobile device<br>- System displays responsive layout<br>- Collapsible sections to reduce scrolling<br>- Key information prioritized at top<br>- Touch-friendly action buttons<br><br>**Booking History for Repeat Guests**:<br>- System detects guest has previous bookings<br>- "View Past Bookings" link displayed<br>- Staff clicks to see guest's booking history<br>- Useful for recognizing VIP guests or addressing repeat issues |
| **Postconditions** | **Success**: Staff views complete booking information. Staff can take appropriate action based on information. Booking data remains unchanged (read-only unless action taken).<br><br>**Failure**: Error message if booking not accessible. Partial data displayed if available. Staff can retry or contact support. |
| **Nonfunctional Requirements** | **Performance**: Booking search returns results within 1 second. Booking details page loads within 1 second. QR code scan processes within 2 seconds.<br><br>**Usability**: Information organized logically in clear sections. Most important information visible without scrolling (above the fold). Search supports partial matches and fuzzy searching. Booking reference easily copyable. Contact methods (email, phone) clickable for direct action. Responsive design for mobile/tablet.<br><br>**Security**: Staff can only view bookings for their assigned property. Sensitive payment info (full card numbers) not displayed. Access logged for audit trail. Guest personal data protected per privacy regulations.<br><br>**Accessibility**: Information hierarchy clear with headings. Key information highlighted visually. Print-friendly layout. Screen reader compatible. Keyboard navigation supported.<br><br>**Data Accuracy**: All information synced with latest booking state. Real-time updates if booking modified elsewhere. Timestamps displayed in staff's local timezone. |
| **Business Requirements** | BR-104: Staff can view all bookings for their property<br>BR-105: Payment card numbers masked (only last 4 digits shown)<br>BR-106: Staff notes private and not visible to guests<br>BR-107: Communication history includes all messages for context<br>BR-108: Activity timeline provides complete audit trail<br>BR-109: Quick actions context-sensitive to booking status<br>BR-110: Booking details accessible offline (cached recent bookings) |
| **Frequency of Use** | Very High (staff check booking details multiple times daily) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should staff be able to export individual booking details to PDF?<br>2. Should system highlight important information (special requests, VIP status)?<br>3. Should guest photo (from profile) be displayed for easier identification?<br>4. Should system show other guests in same room (for dorm awareness)?<br>5. Should staff be able to flag bookings for attention/follow-up?<br>6. Should system display estimated arrival time (from booking or communication)?<br>7. Should booking details include link to guest's full profile/history? |

## Related Use Cases

- **UC-12: Process Booking** — Owner processes bookings viewed by staff
- **UC-17: Check In Guest** — Staff checks in after viewing booking details
- **UC-18: Check Out Guest** — Staff checks out after viewing booking details
- **UC-16: Communicate With Guest** — Contact guest action from booking details

## Notes

- Quick access to booking details essential for efficient operations
- Comprehensive view reduces need to switch between multiple screens
- QR code scanning speeds up guest identification at check-in
- Staff notes enable coordination between shift workers
- Communication history provides context for guest interactions
- Activity timeline helpful for troubleshooting and audit
- Quick actions reduce clicks for common tasks
- Mobile optimization important for staff working floor vs desk
- Repeat guest recognition enhances personalized service
- Read-only view prevents accidental modifications

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Staff accesses comprehensive booking information enabling informed service delivery

✓ **C2: Avoids functional decomposition** — Complete viewing sequence from search to detailed information display

✓ **C3: Maintains black box view** — No mention of database queries, data models, caching; only describes information retrieval and display

✓ **C4: Primary and secondary actors identified** — Primary: Hostel Staff (views booking details); Secondary: None (read-only operation)
