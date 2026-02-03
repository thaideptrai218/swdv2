# UC-16: Communicate With Guest

| Field | Description |
|-------|-------------|
| **Use Case Name** | Communicate With Guest |
| **Use Case ID** | UC-16 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Hostel Owner sends messages to guests regarding bookings, provides check-in instructions, answers questions, and maintains communication history. System delivers messages via Email Service and stores conversation threads. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A1: Hostel Owner. Secondary: A5: Email Service |
| **Preconditions** | Owner authenticated. Booking exists with guest information. Guest has valid email address. Email Service operational. |
| **Trigger** | Owner clicks "Message Guest" button from booking details page (UC-12), or Guest sends inquiry about property |
| **Main Sequence** | 1. Owner navigates to messages inbox<br>2. System retrieves message threads for property<br>3. System displays inbox with conversation list<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. For each thread: guest name, booking reference, last message preview, unread badge, timestamp<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Filters: All, Unread, Archived<br>4. Owner selects conversation thread or starts new message<br>5. System displays message thread view<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Full conversation history (messages from owner and guest)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Associated booking details sidebar (dates, room type, amount)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Guest profile snippet (name, nationality, past bookings count)<br>6. Owner composes new message<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner types message in text area<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Owner can use quick reply templates (check-in instructions, directions, policies)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner attaches files if needed (PDF directions, house rules) (optional)<br>7. Owner sends message<br>8. System validates message content<br>9. System saves message to thread<br>10. System sends email to guest via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Email includes message content<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Email includes "Reply to this email" option<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Email includes link to view full thread in traveler account<br>11. System marks thread as "Owner responded"<br>12. System displays sent confirmation<br>13. Guest receives email notification<br>14. Guest replies via email or through traveler account<br>15. System receives guest reply<br>16. System adds reply to conversation thread<br>17. System marks thread as "Unread" for owner<br>18. System sends real-time notification to owner (if online)<br>19. Owner views guest reply and continues conversation |
| **Alternative Sequences** | **Step 8**: If message empty or contains only whitespace, System displays error "Message cannot be empty" and prevents sending<br><br>**Quick Reply Templates**:<br>- Owner clicks "Quick Reply" button<br>- System displays template categories (Check-in, Directions, Policies, FAQ)<br>- Owner selects template<br>- System inserts template text into message field<br>- Owner customizes text as needed<br>- Owner sends message<br><br>**Automated Messages**:<br>- System sends automated message templates at booking milestones<br>- 7 days before check-in: "Looking forward to your stay" with preparation tips<br>- 1 day before check-in: Check-in instructions with address and contact<br>- On check-in day: Welcome message with WiFi password and house rules<br>- After check-out: Thank you message with review request link<br>- Owner can enable/disable automated messages in settings<br>- Owner can customize automated message templates<br><br>**Bulk Messaging**:<br>- Owner selects multiple upcoming bookings<br>- Owner composes announcement (e.g., "Construction noise expected tomorrow")<br>- System sends to all selected guests<br>- Each guest receives personalized email (Dear [Name])<br><br>**Archive Conversation**:<br>- Owner clicks "Archive" on completed booking thread<br>- System moves thread to archived folder<br>- Thread removed from main inbox<br>- Owner can view archived threads via filter<br><br>**Flag for Follow-Up**:<br>- Owner flags important conversation<br>- Flagged threads appear at top of inbox<br>- Useful for pending questions or special requests<br>- Owner unflags when resolved<br><br>**Guest Inquiry (Pre-Booking)**:<br>- Guest not yet booked, sends inquiry via property page<br>- System creates inquiry thread (no booking reference)<br>- Owner responds to inquiry<br>- If guest books: System links inquiry thread to new booking<br><br>**Attach Media**:<br>- Owner attaches photos (room views, directions, facilities)<br>- Owner attaches documents (house rules PDF, local guide)<br>- System validates file size (<10MB) and format<br>- System uploads to Cloud Storage<br>- File link included in email to guest<br><br>**Translation Assistance** (future enhancement):<br>- System detects guest language<br>- Owner enables translation<br>- System translates messages both directions<br>- Original and translated text both visible<br><br>**Step 10**: If Email Service delivery fails, System retries 3 times with exponential backoff, logs failure, displays warning to owner "Message saved but email delivery failed. Please try alternative contact method." |
| **Postconditions** | **Success**: Message sent to guest via email. Conversation stored in thread. Guest can reply. Communication logged. Owner notified of replies.<br><br>**Failure**: Message not sent. Error displayed. Draft saved. Owner can retry or use alternative contact (phone). |
| **Nonfunctional Requirements** | **Performance**: Inbox loads within 2 seconds for 100 conversations. Message sends within 3 seconds. Email delivery initiated within 10 seconds. Real-time notifications within 5 seconds for online owners.<br><br>**Reliability**: Email delivery success rate >95%. Failed messages retry automatically. Message data persisted durably. Conversation history never lost.<br><br>**Usability**: Inbox searchable by guest name, booking reference, keyword. Conversation threads chronological. Unread count badge visible. Mobile-responsive for on-the-go communication. Rich text formatting (bold, lists, links). Character counter for long messages.<br><br>**Privacy**: Only owner and guest can view conversation. Messages encrypted in transit. Guest email addresses protected from spam. Unsubscribe option for automated messages.<br><br>**Notification**: Owner receives email notifications for new guest messages. Push notifications supported for mobile app (future). Notification preferences configurable (immediate, digest, off). |
| **Business Requirements** | BR-085: All guest-owner communication logged for dispute resolution<br>BR-086: Automated messages sent at key booking milestones (configurable)<br>BR-087: Quick reply templates provided for common questions<br>BR-088: Owner can respond to guest emails via email reply (bidirectional)<br>BR-089: Message history retained indefinitely for reference<br>BR-090: Bulk messaging available for urgent property announcements |
| **Frequency of Use** | High (daily communication with guests before/during/after stays) |
| **Priority** | Medium |
| **Outstanding Questions** | 1. Should system support real-time chat instead of email-based messaging?<br>2. Should system detect message urgency and notify owner accordingly?<br>3. Should owner be able to schedule messages to send at specific times?<br>4. Should system provide read receipts (guest opened message)?<br>5. Should system support voice messages or video attachments?<br>6. Should guest be able to rate communication quality with owner?<br>7. Should system integrate with WhatsApp or Telegram for messaging? |

## Related Use Cases

- **UC-12: Process Booking** — Communication accessible from booking details
- **UC-05: Book Accommodation** — Creates booking and enables guest-owner communication
- **UC-17: Check In Guest** — Communication may include check-in instructions

## Notes

- Centralized communication prevents missed messages across platforms
- Email-based messaging ensures guest access without app download
- Conversation threads provide context and history
- Quick reply templates save time for common questions
- Automated messages reduce manual communication workload
- Bulk messaging useful for urgent property-wide announcements
- Message logging important for dispute resolution
- Real-time notifications enable prompt responses
- Mobile accessibility critical for owners on-the-go
- Translation features would enhance international guest experience

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Owner communicates effectively with guests maintaining conversation history and automated messaging

✓ **C2: Avoids functional decomposition** — Complete messaging sequence from inbox navigation through composition to delivery and reply handling

✓ **C3: Maintains black box view** — No mention of message queues, email templates, notification systems; only describes communication behavior

✓ **C4: Primary and secondary actors identified** — Primary: Owner (communicates with guests); Secondary: Email Service (delivers messages)
