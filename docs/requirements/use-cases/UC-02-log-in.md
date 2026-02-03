# UC-02: Log In

| Field | Description |
|-------|-------------|
| **Use Case Name** | Log In |
| **Use Case ID** | UC-02 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | User (Traveler, Hostel Staff, or Platform Administrator) enters credentials to authenticate. System validates credentials and grants access to role-appropriate dashboard and features. |
| **Dependency** | Include: None. Extend: None. Included by: UC-05 (Book Accommodation), UC-07 (Submit Review) |
| **Actors** | Primary: A2a: Hostel Staff, A3: Traveler, A0: Platform Administrator. Secondary: None |
| **Preconditions** | User has registered account. Account is active (email verified). System is operational. User is not already logged in. |
| **Trigger** | User clicks "Log In" button or attempts to access protected feature requiring authentication |
| **Main Sequence** | 1. User clicks "Log In" button<br>2. System displays login form<br>3. User enters credentials<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. User enters email address<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. User enters password<br>4. User submits login form<br>5. System validates credentials<br>6. System verifies account status is active<br>7. System retrieves user role and permissions<br>8. System creates authenticated session<br>9. System redirects user to role-appropriate dashboard<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. If Traveler: redirect to traveler homepage<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. If Hostel Staff: redirect to staff dashboard<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. If Platform Administrator: redirect to admin dashboard<br>10. System displays welcome message with user name |
| **Alternative Sequences** | **Step 5**: If credentials invalid (email not found or password incorrect), System displays "Invalid email or password" error message, increments failed login counter, and returns to Step 2<br><br>**Step 5**: If failed login attempts exceed 5 within 15 minutes, System temporarily locks account for 30 minutes and displays lockout message with time remaining<br><br>**Step 6**: If account status is "Unverified", System displays message "Please verify your email address" and offers option to resend verification email<br><br>**Step 6**: If account status is "Suspended", System displays message "Account suspended. Contact support for assistance" and prevents login<br><br>**Step 6**: If account status is "Deleted", System displays "Invalid email or password" error (same as invalid credentials for security)<br><br>**Step 3**: User clicks "Forgot Password" link, System displays password reset form, User enters email, System sends password reset link via Email Service |
| **Postconditions** | **Success**: User authenticated with active session. User redirected to role-appropriate dashboard. Session token stored. Last login timestamp recorded. Failed login counter reset.<br><br>**Failure**: User not authenticated. Error message displayed. Failed login attempt logged. Account may be temporarily locked if threshold exceeded. |
| **Nonfunctional Requirements** | **Performance**: Login form displays within 1 second. Authentication completes within 2 seconds. Dashboard loads within 3 seconds.<br><br>**Security**: Passwords transmitted over HTTPS only. Session tokens use secure, httpOnly cookies. Failed login attempts logged with IP address and timestamp. Account lockout after 5 failed attempts within 15 minutes. Session expires after 24 hours of inactivity. Generic error messages prevent username enumeration.<br><br>**Usability**: Login form accessible from all public pages. "Remember me" option available for extended sessions. Password visibility toggle available. Keyboard navigation supported. Clear error messages guide user to resolution.<br><br>**Reliability**: 99.95% authentication success rate for valid credentials. Login service available 24/7.<br><br>**Availability**: Authentication service highly available with failover mechanisms. |
| **Business Requirements** | BR-006: Session timeout after 24 hours of inactivity for security<br>BR-007: Account lockout after 5 failed attempts to prevent brute force attacks<br>BR-008: Multi-factor authentication optional for high-security accounts<br>BR-009: Only active accounts can log in<br>BR-010: Different dashboards and permissions per user role |
| **Frequency of Use** | Very High (multiple logins daily across all user types) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support multi-factor authentication (2FA) as optional or mandatory?<br>2. What is the session timeout policy for different user roles?<br>3. Should system support "remember me" functionality for extended sessions?<br>4. Should system log all login attempts or only failed ones?<br>5. Should system notify users via email of successful logins from new devices?<br>6. Should system support single sign-on (SSO) integration for enterprise customers? |

## Related Use Cases

- **UC-01: Register Account** — Users must register before they can log in
- **UC-05: Book Accommodation** — Includes this use case (requires authentication)
- **UC-07: Submit Review** — Includes this use case (requires authentication)
- **All staff/owner/admin use cases** — Require authentication via this use case

## Notes

- Generic error messages ("Invalid email or password") prevent attackers from determining if email exists in system
- Account lockout threshold balances security and user experience
- Role-based redirection ensures users land on appropriate dashboard immediately after login
- Session management follows OWASP security best practices
- Failed login attempts logged for security monitoring and forensics

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — User gains authenticated access to system features appropriate to their role

✓ **C2: Avoids functional decomposition** — Complete login sequence from form display to dashboard access

✓ **C3: Maintains black box view** — No mention of authentication tables, JWT tokens, session storage; only describes authentication behavior from external perspective

✓ **C4: Primary and secondary actors identified** — Primary: Multiple user types (Traveler, Staff, Admin); Secondary: None for basic login flow
