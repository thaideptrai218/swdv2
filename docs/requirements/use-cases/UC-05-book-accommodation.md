# UC-05: Book Accommodation

| Field | Description |
|-------|-------------|
| **Use Case Name** | Book Accommodation |
| **Use Case ID** | UC-05 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Traveler selects room type, provides guest information, and completes payment to book accommodation. System validates information, processes payment via Payment Gateway, creates booking, and sends confirmation via Email Service. |
| **Dependency** | Include: UC-02 (Log In). Extend: UC-24 (Cancel Booking) |
| **Actors** | Primary: A3: Traveler. Secondary: A4: Payment Gateway, A5: Email Service |
| **Preconditions** | Traveler authenticated (via UC-02). Hostel property active and available. Selected dates have availability. Payment Gateway and Email Service operational. |
| **Trigger** | Traveler clicks "Book Now" button on hostel details page (UC-04) |
| **Main Sequence** | 1. System verifies traveler authentication<br>2. Include: UC-02 (Log In) if not authenticated<br>3. System displays booking form with pre-filled information<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. System shows selected hostel, room type, dates<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. System displays price breakdown (room rate × nights, taxes, fees)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. System shows total amount<br>4. Traveler reviews and confirms booking details<br>5. Traveler enters guest information<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Traveler enters/confirms full name<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Traveler enters/confirms phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Traveler enters estimated arrival time<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Traveler enters special requests (optional)<br>6. Traveler selects payment method<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Options: Credit/Debit Card, VNPay, SePay, Bank Transfer<br>7. System validates guest information format<br>8. System checks room availability (revalidates real-time)<br>9. System creates pending booking<br>10. System redirects traveler to Payment Gateway<br>11. Traveler completes payment at Payment Gateway<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. Payment Gateway displays payment interface<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Traveler enters payment credentials<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Payment Gateway processes transaction<br>12. Payment Gateway sends payment confirmation to System<br>13. System confirms booking<br>14. System sends confirmation email to traveler via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;14.1. Email includes: booking reference number, hostel details, check-in/check-out dates, total amount paid, cancellation policy<br>15. System sends booking notification to hostel owner via Email Service<br>16. System displays booking confirmation page<br>&nbsp;&nbsp;&nbsp;&nbsp;16.1. Shows booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;16.2. Shows booking details summary<br>&nbsp;&nbsp;&nbsp;&nbsp;16.3. Provides links to view booking and contact hostel |
| **Alternative Sequences** | **Step 1**: If traveler not authenticated, System redirects to login page and returns to booking after successful login (Include UC-02)<br><br>**Step 7**: If guest information invalid (phone format incorrect), System displays error messages and returns to Step 5<br><br>**Step 8**: If room no longer available (booked by another traveler), System displays "Room no longer available for selected dates" and offers alternative dates or room types<br><br>**Step 12**: If payment fails (insufficient funds, declined card, transaction timeout), Payment Gateway returns error to System, System cancels pending booking, displays error message, and offers option to retry payment or choose different method<br><br>**Step 12**: If payment processing takes longer than 10 minutes, System displays "Payment pending" status and sends email when payment confirmed<br><br>**Step 14**: If email delivery fails, System logs error, displays warning on confirmation page, and allows traveler to resend confirmation email<br><br>**After Step 16**: Within 24 hours of booking, Traveler may cancel (Extend: UC-24 Cancel Booking) |
| **Postconditions** | **Success**: Booking confirmed and saved. Payment processed. Confirmation emails sent to traveler and hostel owner. Room availability updated. Booking accessible in traveler's account.<br><br>**Failure**: Booking not created. Payment not processed or refunded if captured. Availability not changed. Error message displayed with resolution guidance. |
| **Nonfunctional Requirements** | **Performance**: Booking form displays within 2 seconds. Payment redirect within 3 seconds. Confirmation page displays within 5 seconds after payment success.<br><br>**Security**: Payment data handled exclusively by Payment Gateway (PCI DSS compliant). Secure HTTPS connection throughout booking flow. Booking reference numbers unique and non-guessable. Sensitive data encrypted in transit.<br><br>**Reliability**: 99.9% booking success rate for valid transactions. Idempotent payment processing prevents duplicate charges. Transaction logging for audit trail.<br><br>**Usability**: Progress indicator shows booking steps. Clear price breakdown with no hidden fees. Mobile-optimized form with auto-complete. Error messages specific and actionable. Back button navigation preserves form data.<br><br>**Availability**: Booking service available 24/7. Graceful degradation if external services unavailable. |
| **Business Requirements** | BR-021: Bookings require full prepayment at time of reservation<br>BR-022: Booking confirmation sent within 5 minutes of payment<br>BR-023: Cancellation allowed within 24 hours of booking for full refund<br>BR-024: Payment processing fees disclosed before payment<br>BR-025: Booking reference number unique and includes date/property identifiers |
| **Frequency of Use** | High (primary revenue-generating transaction, multiple daily) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support split payments (multiple travelers sharing cost)?<br>2. Should system offer travel insurance during booking flow?<br>3. What happens if hostel owner rejects booking after payment?<br>4. Should system support installment payments for long-term stays?<br>5. Should system send SMS confirmation in addition to email?<br>6. What is the booking modification policy (change dates, room type)?<br>7. Should system support group bookings (multiple rooms in one transaction)? |

## Related Use Cases

- **UC-02: Log In** — Included; traveler must authenticate to book
- **UC-04: View Hostel Details** — Traveler views details before booking
- **UC-06: Manage Bookings** — After booking, traveler can view/manage booking
- **UC-24: Cancel Booking** — Extends this use case; traveler may cancel within 24 hours
- **UC-12: Process Booking** — Hostel owner receives and processes booking

## Notes

- Authentication required prevents anonymous bookings and reduces fraud
- Payment Gateway integration ensures PCI compliance without handling card data
- Real-time availability check at Step 8 prevents double-booking race conditions
- Confirmation emails critical for guest peace of mind and property coordination
- Clear cancellation policy disclosure reduces disputes
- Booking flow optimization directly impacts revenue conversion

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler receives confirmed booking with payment processed and confirmation received

✓ **C2: Avoids functional decomposition** — Complete booking sequence from form display through payment to confirmation

✓ **C3: Maintains black box view** — No mention of database transactions, payment APIs, session management; only describes booking behavior from external perspective

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (initiates booking); Secondary: Payment Gateway (processes payment), Email Service (sends confirmations)
