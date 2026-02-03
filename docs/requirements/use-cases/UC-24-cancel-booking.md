# UC-24: Cancel Booking

| Field | Description |
|-------|-------------|
| **Use Case Name** | Cancel Booking |
| **Use Case ID** | UC-24 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Summary** | Traveler cancels confirmed booking within cancellation window. System validates cancellation policy, processes refund via Payment Gateway, notifies hostel owner via Email Service, and releases inventory. |
| **Dependency** | Include: None. Extend: UC-05 (Book Accommodation) |
| **Actors** | Primary: A3: Traveler. Secondary: A4: Payment Gateway, A5: Email Service |
| **Preconditions** | Traveler authenticated. Booking exists with status "Confirmed". System operational. Payment Gateway and Email Service available. |
| **Trigger** | Traveler clicks "Cancel Booking" button from booking details page (UC-06) |
| **Main Sequence** | 1. Traveler navigates to booking details<br>2. System displays booking with cancellation option<br>3. Traveler clicks "Cancel Booking"<br>4. System retrieves cancellation policy for booking<br>5. System calculates refund amount based on policy and timing<br>6. System displays cancellation confirmation screen<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Booking details summary<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Cancellation policy terms<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Refund amount breakdown (total paid - cancellation fee = refund)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Processing timeline (7-14 business days)<br>7. Traveler selects cancellation reason (optional dropdown)<br>8. Traveler confirms cancellation<br>9. System validates cancellation eligibility<br>10. System updates booking status to "Cancelled"<br>11. System records cancellation timestamp and reason<br>12. System initiates refund process via Payment Gateway<br>13. Payment Gateway processes refund to original payment method<br>14. Payment Gateway confirms refund initiated<br>15. System releases bed/room from occupancy<br>16. System updates availability for future bookings<br>17. System sends cancellation confirmation email to traveler via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;17.1. Email includes: cancellation confirmation, refund amount, processing timeline, booking reference<br>18. System sends cancellation notification to owner via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;18.1. Email includes: guest name, booking details, cancellation date, released inventory<br>19. System displays success message "Booking cancelled successfully. Refund will be processed within 7-14 business days." |
| **Alternative Sequences** | **Outside Cancellation Window** (>24 hours after booking):<br>- Step 5: System calculates zero refund per policy<br>- Step 6: Shows "No refund available - outside cancellation window"<br>- Traveler can proceed with cancellation (no refund) or keep booking<br>- If cancels: No refund processed, room released, notifications sent<br><br>**Partial Refund Scenario** (group booking, partial cancel):<br>- Traveler cancels portion of group booking<br>- System calculates pro-rated refund<br>- Updates booking to reduced guest count<br>- Processes partial refund<br><br>**Payment Gateway Refund Failure**:<br>- Step 13: Payment Gateway returns error<br>- System logs failure<br>- System displays "Booking cancelled but refund processing failed. Support team notified."<br>- Support team manually processes refund<br>- System updates when refund completed<br><br>**Already Checked In**:<br>- Step 9: Validation fails - guest already checked in<br>- System displays "Cannot cancel after check-in. Please contact property for early check-out."<br>- Directs traveler to contact owner<br><br>**Within 24h of Check-In**:<br>- System displays warning "Cancellation within 24h may result in fees per property policy"<br>- Shows adjusted refund amount (cancellation fee applied)<br>- Traveler proceeds or keeps booking<br><br>**Email Delivery Failure**:<br>- Step 17/18: Email Service returns error<br>- System retries delivery 3 times<br>- If all fail: System logs error, cancellation still valid<br>- Support team manually notifies parties<br><br>**Cancellation by Owner/Admin**:<br>- Owner cancels booking (overbooking, property issue)<br>- Full refund issued regardless of policy<br>- Profuse apology email sent<br>- System may offer compensation credit |
| **Postconditions** | **Success**: Booking cancelled. Refund processed or scheduled. Inventory released. All parties notified. Cancellation logged.<br><br>**Failure**: Cancellation blocked (validation failed). Error message displayed with reason. Booking remains active. |
| **Nonfunctional Requirements** | **Performance**: Cancellation form displays <1s. Cancellation processes <5s. Refund initiated <30s. Email notifications sent <1min.<br><br>**Financial**: Refund calculations accurate per policy. Payment Gateway refunds tracked. Reconciliation with accounting system. Refund processing timeline 7-14 business days.<br><br>**User Experience**: Clear cancellation policy displayed before confirmation. Refund amount breakdown transparent. Processing timeline communicated clearly. Confirmation email provides peace of mind.<br><br>**Reliability**: Refund failures logged and escalated. Inventory release atomic with cancellation. Notifications sent even if refund pending. Audit trail complete. |
| **Business Requirements** | BR-023: Cancellation allowed within 24 hours of booking for full refund<br>BR-128: Refund processing timeline 7-14 business days via Payment Gateway<br>BR-129: Inventory released immediately upon cancellation<br>BR-130: Owner notified of all cancellations automatically<br>BR-131: Cancellation reason collected for analytics<br>BR-132: Admin can cancel on behalf of traveler with same workflow |
| **Frequency of Use** | Medium (cancellation rate typically 5-10% of bookings) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system offer rebooking instead of full cancellation?<br>2. Should travelers be able to cancel via email/SMS?<br>3. Should system allow cancellation without login (secure link)?<br>4. Should refund go to platform credit vs original payment method?<br>5. Should system provide cancellation protection insurance option?<br>6. What happens to loyalty points earned from cancelled bookings?<br>7. Should system blacklist users with high cancellation rates? |

## Related Use Cases

- **UC-05: Book Accommodation** — Extended by this use case; cancellation option available after booking
- **UC-06: Manage Bookings** — Cancellation initiated from booking management
- **UC-12: Process Booking** — Owner sees cancelled bookings
- **UC-15: Manage Subscription** — Refunds impact commission calculations

## Notes

- Flexible cancellation policy competitive advantage for travelers
- 24-hour cancellation window standard in hospitality industry
- Clear communication reduces support inquiries
- Immediate inventory release maximizes rebooking opportunity
- Refund transparency builds trust
- Payment Gateway handles PCI compliance for refunds
- Email notifications keep all parties informed
- Cancellation analytics help identify property/booking issues
- Extends UC-05 as per source document (optional cancellation within window)

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler cancels booking with refund processed and clear communication

✓ **C2: Avoids functional decomposition** — Complete cancellation sequence from request through refund to notifications

✓ **C3: Maintains black box view** — No mention of refund APIs, inventory tables, email queues; only describes cancellation behavior

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (cancels booking); Secondary: Payment Gateway (refunds), Email Service (notifies)
