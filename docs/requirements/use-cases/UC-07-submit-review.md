# UC-07: Submit Review

| Field | Description |
|-------|-------------|
| **Use Case Name** | Submit Review |
| **Use Case ID** | UC-07 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Traveler submits review and rating for hostel after completed stay. System validates review is for past booking by authenticated traveler, publishes review, and updates hostel's aggregate rating. |
| **Dependency** | Include: UC-02 (Log In). Extend: None |
| **Actors** | Primary: A3: Traveler. Secondary: None |
| **Preconditions** | Traveler authenticated (via UC-02). Traveler has completed booking (check-out date passed). Traveler has not already reviewed this booking. System operational. |
| **Trigger** | Traveler clicks "Leave Review" button on past booking in UC-06, or System prompts traveler via email after check-out |
| **Main Sequence** | 1. System verifies traveler authentication<br>2. Include: UC-02 (Log In) if not authenticated<br>3. System verifies traveler has completed stay at hostel<br>4. System checks traveler has not already reviewed this booking<br>5. System displays review form<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. System shows booking details (hostel name, dates stayed, room type)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. System displays rating criteria (overall, cleanliness, location, staff, facilities, value)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. System shows text area for written review<br>6. Traveler provides ratings<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Traveler rates overall experience (1-5 stars, required)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Traveler rates individual aspects (1-5 stars each, optional)<br>7. Traveler writes review text<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Minimum 10 characters, maximum 2000 characters<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Character counter displays remaining characters<br>8. Traveler uploads photos (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Maximum 5 photos<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Supported formats: JPG, PNG<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Maximum 5MB per photo<br>9. Traveler submits review<br>10. System validates review content<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Checks minimum character requirement met<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Checks for prohibited content (profanity, personal info, spam)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Validates photo formats and sizes<br>11. System saves review with pending status<br>12. System sends review to moderation queue<br>13. System displays confirmation message<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. "Thank you! Your review will be published after moderation (usually within 24 hours)"<br>14. After moderation approval, System publishes review<br>15. System recalculates hostel's aggregate ratings<br>&nbsp;&nbsp;&nbsp;&nbsp;15.1. Updates overall rating average<br>&nbsp;&nbsp;&nbsp;&nbsp;15.2. Updates individual aspect averages<br>&nbsp;&nbsp;&nbsp;&nbsp;15.3. Updates review count<br>16. System marks booking as reviewed<br>17. System displays review on hostel details page (UC-04) |
| **Alternative Sequences** | **Step 1**: If traveler not authenticated, System redirects to login and returns to review form after successful login (Include UC-02)<br><br>**Step 3**: If check-out date is in future, System displays "You can review this property after your stay ends on [date]" and prevents review submission<br><br>**Step 4**: If traveler already reviewed this booking, System displays existing review with option to edit (within 7 days of original submission)<br><br>**Step 10.2**: If review contains prohibited content, System displays specific error messages (e.g., "Review contains inappropriate language", "Please remove email addresses") and returns to Step 7<br><br>**Step 10.3**: If photo validation fails (wrong format, too large), System displays error for specific photos and allows traveler to remove/replace them<br><br>**Step 14**: If moderation rejects review (spam, fake, inappropriate), System notifies traveler via email with rejection reason and allows resubmission<br><br>**Step 8**: If photo upload fails, System displays error and allows traveler to retry or submit review without photos |
| **Postconditions** | **Success**: Review submitted and queued for moderation. After approval, review published on hostel page. Hostel ratings updated. Booking marked as reviewed. Traveler receives confirmation.<br><br>**Failure**: Review not saved. Validation errors displayed. Traveler can correct and resubmit. |
| **Nonfunctional Requirements** | **Performance**: Review form loads within 2 seconds. Submission processes within 3 seconds. Photo uploads complete within 10 seconds (per photo).<br><br>**Security**: Only verified guests who completed stay can review. One review per booking. Reviews tied to authenticated traveler account. Moderation prevents fake/spam reviews.<br><br>**Usability**: Character counter provides real-time feedback. Star ratings responsive with visual feedback. Photo upload drag-and-drop supported. Mobile-optimized form. Review guidelines visible. Clear validation error messages.<br><br>**Content Moderation**: Reviews moderated within 24 hours. Automated filter for profanity and personal information. Manual review for suspicious patterns. Rejected reviews allow resubmission.<br><br>**Transparency**: Review display includes: traveler name (first name + initial), date posted, verified stay badge, helpful count (from other travelers). |
| **Business Requirements** | BR-031: Only verified guests who completed stay can submit reviews<br>BR-032: Reviews published after moderation approval (typically <24 hours)<br>BR-033: One review per booking; edits allowed within 7 days<br>BR-034: Reviews minimum 10 characters to ensure substantive feedback<br>BR-035: Aggregate ratings updated immediately upon review publication<br>BR-036: Review photos subject to content moderation |
| **Frequency of Use** | Medium (subset of travelers submit reviews after stays) |
| **Priority** | Medium |
| **Outstanding Questions** | 1. Should system allow hostel owners to respond to reviews?<br>2. Should travelers be able to mark reviews as "helpful" or report inappropriate reviews?<br>3. Should system send reminder emails to travelers who haven't reviewed?<br>4. What is the time limit for submitting reviews after check-out (30 days, 90 days, unlimited)?<br>5. Should system support review translations for international travelers?<br>6. Should system display verification badge showing review is from actual guest?<br>7. Should travelers receive incentives (discount, credits) for submitting reviews? |

## Related Use Cases

- **UC-02: Log In** — Included; traveler must authenticate to submit review
- **UC-04: View Hostel Details** — Reviews displayed on hostel details page
- **UC-06: Manage Bookings** — Review submission accessed from past bookings
- **UC-05: Book Accommodation** — Creates bookings that can later be reviewed

## Notes

- Authentication and verified stay requirements ensure review authenticity
- Moderation process maintains review quality and prevents abuse
- Aggregate rating updates provide social proof for future travelers
- Photo uploads enhance review credibility and usefulness
- Character minimum ensures reviews provide substantive feedback
- Review system critical for building trust in platform and properties

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler submits review that will be published after moderation, contributing to community knowledge

✓ **C2: Avoids functional decomposition** — Complete review submission sequence from form display through validation to publication

✓ **C3: Maintains black box view** — No mention of review tables, content filters, aggregation algorithms; only describes review submission and publication behavior

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (submits review); Secondary: None (moderation is internal system process)
