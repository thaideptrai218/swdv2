# UC-17: Check In Guest

| Field | Description |
|-------|-------------|
| **Use Case Name** | Check In Guest |
| **Use Case ID** | UC-17 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Hostel Staff verifies guest identity, confirms booking details, collects required information, assigns bed or room, and completes check-in process. System updates booking status and records check-in timestamp. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A2a: Hostel Staff (generalized actor covering Receptionist and Owner). Secondary: None |
| **Preconditions** | Staff authenticated with check-in permissions. Confirmed booking exists with check-in date today or past. Guest has arrived at property. System operational. |
| **Trigger** | Guest arrives at property and approaches reception desk, or Staff proactively checks in arriving guest |
| **Main Sequence** | 1. Staff navigates to check-in interface<br>2. System displays today's expected arrivals list<br>&nbsp;&nbsp;&nbsp;&nbsp;2.1. For each arrival: guest name, booking reference, room type, check-in time, status<br>&nbsp;&nbsp;&nbsp;&nbsp;2.2. List sorted by check-in time<br>3. Staff searches for guest booking<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Staff enters guest name, booking reference, or email<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. System displays matching bookings<br>4. Staff selects correct booking<br>5. System displays check-in form with booking details<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Guest information (name, email, phone, nationality)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Booking details (dates, room type, number of guests, amount paid)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Special requests from guest<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Payment status (fully paid, deposit paid, balance due)<br>6. Staff verifies guest identity<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Staff requests government-issued ID (passport, driver's license)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Staff visually confirms name matches booking<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Staff optionally scans or photographs ID (for records)<br>7. Staff confirms guest count and details<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Staff confirms number of guests arriving<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. If guest count differs from booking: Staff updates count or discusses with guest<br>8. Staff collects additional required information<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Emergency contact (name and phone)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Expected check-out time<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Any additional requirements (extra towels, late check-out request)<br>9. Staff reviews house rules with guest<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Quiet hours, smoking policy, common area etiquette<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Guest verbally acknowledges understanding<br>10. If dorm booking: Staff assigns specific bed<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. System displays room layout with available beds<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Staff selects bed for guest<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. If multiple guests: Staff assigns multiple beds<br>&nbsp;&nbsp;&nbsp;&nbsp;10.4. System marks selected beds as occupied<br>11. Staff provides property information to guest<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. WiFi name and password<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Breakfast time and location (if included)<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Key or access card for room/locker<br>&nbsp;&nbsp;&nbsp;&nbsp;11.4. Tour of facilities (optional)<br>12. Staff processes any outstanding payment<br>&nbsp;&nbsp;&nbsp;&nbsp;12.1. If balance due: Staff collects payment (cash, card, transfer)<br>&nbsp;&nbsp;&nbsp;&nbsp;12.2. Staff marks payment as received in system<br>13. Staff completes check-in<br>14. System validates all required information collected<br>15. System updates booking status to "Checked-In"<br>16. System records check-in timestamp<br>17. System marks assigned bed/room as occupied<br>18. System displays success message "Guest checked in successfully"<br>19. Staff provides receipt or check-in confirmation to guest |
| **Alternative Sequences** | **Step 3**: If booking not found (wrong name, reference), Staff tries alternative search criteria, contacts guest for booking confirmation email, or manually creates booking if prepaid proof provided<br><br>**Step 5**: If booking not confirmed (still pending), Staff contacts owner/manager for approval, or proceeds with check-in if payment verified<br><br>**Step 5**: If check-in date is future, System displays warning "Early check-in - Booking scheduled for [date]", Staff can proceed if room available or offer luggage storage<br><br>**Step 7**: If guest count exceeds booking capacity, Staff discusses options: pay for additional guest, book additional bed/room, or guest finds alternative accommodation<br><br>**Step 10**: If no beds available (overbooking error), Staff apologizes, contacts manager, offers alternatives (upgrade to private room, nearby property referral, refund), escalates to owner<br><br>**Step 12**: If guest unable to pay balance, Staff contacts manager, may allow pay later with signed agreement, or refuses check-in per policy<br><br>**Step 14**: If required information missing (no ID, no emergency contact), Staff cannot complete check-in, explains requirements, guest must provide information<br><br>**Late Check-In Flow**:<br>- Guest arrives after reception closes<br>- Self check-in option: Guest receives access code via email<br>- Guest retrieves key from lockbox<br>- Booking already assigned bed/room<br>- Staff completes formal check-in next morning<br><br>**Group Check-In**:<br>- Multiple guests on single booking<br>- Staff checks in lead guest<br>- Lead guest provides information for all group members<br>- Staff assigns all beds at once<br>- Individual guests collect keys<br><br>**No-Show Update**:<br>- Guest doesn't arrive by end of check-in day<br>- Staff marks booking as "No-Show"<br>- System applies no-show policy<br>- Bed/room released for resale<br><br>**ID Verification Failure**:<br>- ID name doesn't match booking<br>- Staff contacts booking holder<br>- If authorized: Guest provides authorization letter/message<br>- If not authorized: Staff refuses check-in per security policy |
| **Postconditions** | **Success**: Guest checked in. Booking status updated to "Checked-In". Bed/room assigned and marked occupied. Check-in timestamp recorded. Guest has room access. WiFi credentials provided.<br><br>**Failure**: Check-in not completed. Booking status unchanged. Bed/room not assigned. Error or missing information displayed. Staff resolves issue before proceeding. |
| **Nonfunctional Requirements** | **Performance**: Expected arrivals list loads within 2 seconds. Booking search returns results within 1 second. Check-in form displays within 1 second. Check-in completion saves within 2 seconds.<br><br>**Usability**: Check-in interface optimized for speed (minimal clicks). Barcode/QR code scanner supported for booking reference. Touch-friendly for tablet at reception desk. Keyboard shortcuts for power users. Clear visual indicators for completed steps.<br><br>**Security**: ID verification mandatory for security and legal compliance. Check-in timestamps immutable for audit trail. Only staff with check-in permissions can access. Guest personal information displayed only to authorized staff.<br><br>**Reliability**: Check-in process works offline (sync when connection restored). Duplicate check-in prevention (can't check in twice). Real-time bed availability to prevent conflicts.<br><br>**Accuracy**: Check-in timestamp accurate to second. Bed assignments conflict-free. Payment records reconcile with financial reports. |
| **Business Requirements** | BR-091: ID verification mandatory for all check-ins (legal compliance)<br>BR-092: Check-in available from scheduled date forward (early check-in allowed if room available)<br>BR-093: House rules acknowledgment required before check-in completion<br>BR-094: Emergency contact information mandatory for safety<br>BR-095: Check-in timestamp recorded for legal/operational records<br>BR-096: Bed assignment required for dorm bookings before check-in completion |
| **Frequency of Use** | Very High (daily for active properties, multiple check-ins per day) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support digital ID verification (scan passport, extract data)?<br>2. Should guest sign digital agreement on tablet during check-in?<br>3. Should system generate printed welcome sheet with WiFi, rules, local tips?<br>4. Should check-in be possible via mobile app (contactless self check-in)?<br>5. Should system collect COVID vaccination status or health declarations?<br>6. Should staff be able to check in guests remotely before arrival?<br>7. Should system support express check-in for returning guests (saved information)? |

## Related Use Cases

- **UC-12: Process Booking** — Owner confirms booking that staff checks in
- **UC-19: View Booking Details** — Staff views booking before check-in
- **UC-18: Check Out Guest** — Completes stay cycle started by check-in
- **UC-05: Book Accommodation** — Guest books accommodation that requires check-in

## Notes

- Check-in critical touchpoint for guest experience
- ID verification required for security and legal compliance
- Bed assignment unique to hostel operations
- Quick check-in reduces guest wait time and friction
- WiFi credentials highly requested information
- House rules acknowledgment protects property from disputes
- Emergency contact important for guest safety
- Offline capability ensures check-in during connectivity issues
- Tablet interface common in modern hostel reception desks
- Express check-in for returning guests improves efficiency

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Staff completes guest check-in with all required information collected and room access granted

✓ **C2: Avoids functional decomposition** — Complete check-in sequence from guest search through verification to room assignment

✓ **C3: Maintains black box view** — No mention of booking tables, status enums, bed assignment algorithms; only describes check-in behavior

✓ **C4: Primary and secondary actors identified** — Primary: Hostel Staff (performs check-in); Secondary: None (all operations internal)
