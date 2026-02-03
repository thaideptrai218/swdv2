# UC-01: Register Account

| Field | Description |
|-------|-------------|
| **Use Case Name** | Register Account |
| **Use Case ID** | UC-01 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Traveler creates new account by providing personal information and email. System validates information, creates account, and sends verification email to confirm registration. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A3: Traveler. Secondary: A5: Email Service |
| **Preconditions** | Traveler has internet access. System is operational. Traveler does not have existing account. |
| **Trigger** | Traveler clicks "Sign Up" or "Register" button on public portal |
| **Main Sequence** | 1. Traveler clicks "Sign Up" button<br>2. System displays registration form<br>3. Traveler enters personal information<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Traveler enters full name<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Traveler enters email address<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Traveler enters password<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Traveler confirms password<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Traveler accepts terms and conditions<br>4. Traveler submits registration form<br>5. System validates information format<br>6. System checks email availability<br>7. System creates user account<br>8. System sends verification email to traveler's email address via Email Service<br>9. System displays success message with instructions to verify email<br>10. Traveler opens verification email<br>11. Traveler clicks verification link<br>12. System validates verification token<br>13. System activates account<br>14. System displays account activation confirmation |
| **Alternative Sequences** | **Step 5**: If information format invalid (email format incorrect, password too weak), System displays specific error messages and returns to Step 2<br><br>**Step 6**: If email already registered, System displays "Email already exists" message and offers "Login" or "Reset Password" options<br><br>**Step 8**: If email delivery fails, System logs error and displays message to traveler to try again later or contact support<br><br>**Step 12**: If verification token expired (after 24 hours), System displays error message and offers option to resend verification email<br><br>**Step 12**: If verification token invalid, System displays error message and offers option to resend verification email |
| **Postconditions** | **Success**: Traveler account created and activated. Traveler can log in. Welcome email sent. Account status set to "Active".<br><br>**Failure**: No account created. Traveler informed of specific error. |
| **Nonfunctional Requirements** | **Performance**: Registration form displays within 2 seconds. Account creation completes within 5 seconds. Email delivery initiated within 10 seconds.<br><br>**Security**: Passwords must meet minimum strength requirements (8+ characters, mixed case, numbers, special characters). Verification tokens expire after 24 hours. Email addresses encrypted during transmission. Password stored securely (never in plain text).<br><br>**Usability**: Registration form accessible on mobile and desktop. Clear error messages for invalid input. Inline validation for immediate feedback. Password strength indicator visible.<br><br>**Reliability**: 99.9% account creation success rate. Email delivery success rate >95%.<br><br>**Availability**: Registration available 24/7 except during scheduled maintenance windows. |
| **Business Requirements** | BR-001: Users must verify email addresses before accessing booking features<br>BR-002: Minimum age requirement: 18 years<br>BR-003: One account per email address<br>BR-004: Account credentials must meet security standards<br>BR-005: Terms and conditions acceptance required before registration |
| **Frequency of Use** | High (multiple registrations daily, especially during peak travel planning seasons) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support social login (Google, Facebook) in addition to email registration?<br>2. What is the account lockout policy after multiple failed verification attempts?<br>3. Should system collect additional information during registration (phone number, nationality, date of birth)?<br>4. What is the data retention policy for unverified accounts?<br>5. Should system allow registration without email verification for immediate booking needs? |

## Related Use Cases

- **UC-02: Log In** — After successful registration and verification, traveler can log in
- **UC-05: Book Accommodation** — Requires authentication (include UC-02), which requires registration first

## Notes

- Email verification is mandatory to prevent spam accounts and ensure valid contact information
- Registration flow follows industry best practices for conversion optimization
- Password requirements balance security and usability
- System provides clear feedback at each step to reduce abandonment
- Verification link remains valid for 24 hours, after which traveler must request new verification email

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler receives activated account enabling booking capabilities

✓ **C2: Avoids functional decomposition** — Complete registration sequence from form display to account activation

✓ **C3: Maintains black box view** — No mention of databases, controllers, internal components; only describes system behavior from external perspective

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (initiates), Secondary: Email Service (delivers verification)
