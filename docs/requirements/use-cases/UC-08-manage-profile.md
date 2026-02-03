# UC-08: Manage Profile

| Field | Description |
|-------|-------------|
| **Use Case Name** | Manage Profile |
| **Use Case ID** | UC-08 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Traveler views and updates personal profile information including name, email, phone, password, communication preferences, and profile photo. System validates changes and updates profile accordingly. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A3: Traveler. Secondary: None |
| **Preconditions** | Traveler authenticated. System operational. Traveler has active account. |
| **Trigger** | Traveler clicks "Profile" or "Account Settings" link in navigation menu or user dropdown |
| **Main Sequence** | 1. Traveler navigates to profile page<br>2. System retrieves traveler's current profile information<br>3. System displays profile sections<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Personal Information section (name, email, phone, date of birth)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Profile Photo section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Password section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Communication Preferences section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Account Security section<br>4. Traveler edits personal information<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Traveler updates full name<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Traveler updates phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Traveler updates date of birth<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Email display only (changes require verification)<br>5. System validates information format as traveler types<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Real-time validation for phone format<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Age validation (must be 18+)<br>6. Traveler saves changes<br>7. System validates all modified fields<br>8. System updates profile information<br>9. System displays success message "Profile updated successfully"<br>10. System refreshes page with updated information |
| **Alternative Sequences** | **Step 5**: If information format invalid (phone format incorrect, age under 18), System displays inline error messages and prevents saving<br><br>**Step 7**: If validation fails, System displays specific error messages and highlights problematic fields<br><br>**Profile Photo Flow**:<br>- Traveler clicks "Upload Photo"<br>- Traveler selects image file (JPG, PNG, max 5MB)<br>- System validates image format and size<br>- System displays image preview with crop tool<br>- Traveler adjusts crop area<br>- Traveler saves photo<br>- System stores photo and updates profile<br>- If upload fails: System displays error and allows retry<br><br>**Email Change Flow**:<br>- Traveler clicks "Change Email"<br>- System displays email change form<br>- Traveler enters new email address<br>- Traveler enters current password for verification<br>- System validates password<br>- System sends verification link to new email<br>- System displays "Verification email sent to [new email]"<br>- Traveler clicks verification link in email<br>- System updates email address<br>- If verification not completed within 24 hours: link expires<br><br>**Password Change Flow**:<br>- Traveler clicks "Change Password"<br>- System displays password change form<br>- Traveler enters current password<br>- Traveler enters new password<br>- Traveler confirms new password<br>- System validates current password<br>- System validates new password strength (8+ chars, mixed case, numbers, special chars)<br>- System validates passwords match<br>- System updates password<br>- System displays "Password changed successfully"<br>- System sends security notification email<br>- If current password invalid: display error and offer "Forgot Password" link<br><br>**Communication Preferences Flow**:<br>- Traveler toggles email preferences (booking confirmations, promotional emails, review reminders, newsletter)<br>- Traveler toggles SMS preferences (booking confirmations, urgent notifications)<br>- Traveler selects preferred language (English, Vietnamese)<br>- System saves preferences immediately<br>- System displays "Preferences updated"<br><br>**Account Security Flow**:<br>- Traveler views login history (last 10 logins with dates, locations, devices)<br>- Traveler enables/disables two-factor authentication (if available)<br>- Traveler views active sessions<br>- Traveler logs out other sessions<br>- System terminates selected sessions<br>- System displays "Sessions logged out successfully" |
| **Postconditions** | **Success**: Profile information updated. Changes reflected throughout system. Traveler receives confirmation. Email/password changes trigger security notifications.<br><br>**Failure**: Profile not updated. Validation errors displayed. Traveler can correct and retry. |
| **Nonfunctional Requirements** | **Performance**: Profile page loads within 1.5 seconds. Updates save within 2 seconds. Photo uploads complete within 10 seconds.<br><br>**Security**: Password changes require current password verification. Email changes require new email verification. Passwords never displayed (show as dots). Session tokens refreshed after security changes. Password strength enforced. Security notification emails sent for sensitive changes.<br><br>**Usability**: Inline validation provides immediate feedback. Required fields clearly marked. Password strength indicator visible. Photo crop tool intuitive. Mobile-responsive layout. Unsaved changes warning when navigating away.<br><br>**Data Privacy**: Traveler controls communication preferences. Data deletion option available (separate account closure flow). Personal data encrypted at rest and in transit.<br><br>**Validation**: Phone number format supports international formats. Email format validation RFC 5322 compliant. Age validation ensures legal compliance (18+). |
| **Business Requirements** | BR-037: Profile changes take effect immediately throughout system<br>BR-038: Email and password changes trigger security notification emails<br>BR-039: Minimum age requirement 18 years enforced<br>BR-040: Communication preferences respected for all system notifications<br>BR-041: Profile photo displayed on reviews and bookings |
| **Frequency of Use** | Medium (occasional updates, increased activity around first bookings) |
| **Priority** | Medium |
| **Outstanding Questions** | 1. Should system support social profile links (Facebook, Instagram)?<br>2. Should traveler be able to add emergency contact information?<br>3. Should system support multiple phone numbers (home, mobile)?<br>4. Should system show account activity timeline (bookings, reviews, logins)?<br>5. Should traveler be able to download personal data (GDPR compliance)?<br>6. Should system support account deletion with data retention policy?<br>7. Should profile completion percentage be displayed to encourage complete profiles? |

## Related Use Cases

- **UC-01: Register Account** — Creates initial profile during registration
- **UC-05: Book Accommodation** — Uses profile information for booking
- **UC-07: Submit Review** — Profile name and photo displayed with reviews

## Notes

- Profile completeness affects booking trust and review credibility
- Security features (password change, login history) build user confidence
- Communication preferences reduce unsubscribe rates and improve engagement
- Email verification for changes prevents account hijacking
- Phone number useful for booking coordination and SMS notifications
- Profile photo personalization increases engagement
- GDPR compliance considerations for data privacy and deletion rights

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler updates profile information with immediate system-wide effect

✓ **C2: Avoids functional decomposition** — Complete profile management sequence covering all profile sections and update flows

✓ **C3: Maintains black box view** — No mention of user tables, encryption algorithms, session storage; only describes profile update behavior

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (manages profile); Secondary: None (email notifications are system-initiated internal operations)
