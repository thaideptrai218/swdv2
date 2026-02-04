# Object Structuring: Log In

**Use Case**: docs/requirements/use-cases/UC-02-log-in.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Primary Actors**: Multiple user types (Traveler, Hostel Staff, Platform Administrator)
**External Classes (Phase 1)**:

- Traveler («external user»)
- HostelStaff («external user»)
- PlatformAdministrator («external user»)

**Interface Type**:

- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Analysis**: Multiple user types initiate login through web interface. All share same login mechanism but different post-authentication routing.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| UserLoginUI | «user interaction» | Unified login interface for all user types. Handles login form display, credential input, error messages, and role-specific dashboard redirection. |

---

## 2. Entity Objects (Data)

**Data References**:

- Step 5: "System validates credentials" → User entity (email, password)
- Step 6: "System verifies account status" → User entity (accountStatus)
- Step 7: "System retrieves user role and permissions" → User entity (role)
- Step 8: "System creates authenticated session" → Session data (implied)
- Step 9: "System redirects to role-appropriate dashboard" → User entity (role)
- Precondition: "User has registered account" → User entity
- Postcondition: "Last login timestamp recorded" → User entity (lastLoginDate)
- Alternative: "Failed login counter" → User entity or Login tracking

**Referenced Data**:

- **User**: Primary entity for authentication
    - Attributes used: userId, email, password, role, accountStatus, lastLoginDate
- **Notification**: Implicit - "System sends password reset link" creates notification

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| User | Yes | ✅ | Queried for credentials (step 5), status verification (step 6), role retrieval (step 7), timestamp update (postcondition) |
| Notification | Yes | ✅ | Created for password reset email (alternative step 3) |

---

## 3. Boundary Objects (Output)

**Output Type**:

- [x] Display/Screen (error messages, dashboard redirection)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing UserLoginUI)

**Analysis**:

- Steps 2, 5, 6: System displays login form, error messages, lockout messages
- Step 9-10: System redirects and displays welcome message
- Password reset: System sends email (alternative flow)
- All screen output handled by same UI boundary object

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| UserLoginUI | «user interaction» | Reused for output - displays login form, error messages (invalid credentials, account locked, unverified), success redirection |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends password reset link (alternative flow step 3). |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Login process has state-dependent behavior:

- **Active account** state: Allows login
- **Unverified account** state: Blocks login, offers resend verification
- **Suspended account** state: Blocks login, displays support message
- **Account lockout** state: Temporarily blocks login after 5 failed attempts (30-minute window)
- Failed login counter state affects behavior (increments, threshold check, reset on success)
- Different responses based on account status (Step 6 alternatives)

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| AuthenticationControl | «state-dependent control» | ⚠️ Yes | Coordinates login workflow through states: credential validation → account status check → session creation → role-based routing. Manages failed login counter and lockout state. Behavior differs based on account status (Unverified/Suspended/Deleted/Active) and lockout state. |

---

## 5. Application Logic

**Complex Rules**:

- [x] Multi-condition validation (credentials, account status, lockout threshold)
- [ ] Algorithms
- [x] Cross-entity validation (email lookup, password verification)

**Identified Logic**:

- **Credential validation**: Email lookup, password hash verification
- **Account lockout logic**: Failed attempt counting (5 within 15 minutes), 30-minute lockout enforcement
- **Role-based routing**: Different dashboard URLs based on user role
- **Session management**: Token generation, expiration (24 hours)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| CredentialValidator | «business logic» | Encapsulates authentication logic: email lookup, password hash comparison, generic error messages for security (prevents username enumeration). |
| AccountLockoutManager | «business logic» | Manages failed login attempts: counter increment, threshold detection (5 attempts within 15 minutes), lockout enforcement (30 minutes), counter reset on success. Includes IP address logging for security. |
| SessionManager | «service» | Creates and manages authenticated sessions: token generation, 24-hour expiration, secure cookie configuration (httpOnly, secure). |

---

## Object Summary

| Object                | Stereotype                | Category          | Phase 1 Link         | Purpose                                           |
| --------------------- | ------------------------- | ----------------- | -------------------- | ------------------------------------------------- |
| UserLoginUI           | «user interaction»        | Boundary (I/O)    | Traveler/Staff/Admin | Login form and multi-role interface               |
| EmailServiceProxy     | «proxy»                   | Boundary (Output) | EmailService         | Password reset email delivery                     |
| AuthenticationControl | «state-dependent control» | Control           | N/A                  | Login workflow orchestration and state management |
| User                  | «entity»                  | Entity            | User                 | Credential storage and role data                  |
| Notification          | «entity»                  | Entity            | Notification         | Password reset email tracking                     |
| CredentialValidator   | «business logic»          | Application Logic | N/A                  | Authentication validation                         |
| AccountLockoutManager | «business logic»          | Application Logic | N/A                  | Failed attempt tracking and lockout enforcement   |
| SessionManager        | «service»                 | Application Logic | N/A                  | Session creation and management                   |

**Total**: 8 objects (2 boundary, 2 entity, 1 control, 3 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:

- **Primary Actors**: Traveler, HostelStaff, PlatformAdministrator (unified flow)
- **Secondary Actor**: EmailService (password reset only)
- **Boundary**: UserLoginUI, EmailServiceProxy
- **Control**: AuthenticationControl
- **Entity**: User, Notification
- **Logic**: CredentialValidator, AccountLockoutManager, SessionManager

**Key Interactions (Main Flow)**:

1. User → UserLoginUI: Submit credentials
2. UserLoginUI → AuthenticationControl: Process login
3. AuthenticationControl → CredentialValidator: Validate credentials
4. CredentialValidator → User: Lookup email and verify password
5. AuthenticationControl → User: Check account status
6. AuthenticationControl → AccountLockoutManager: Check lockout status
7. AuthenticationControl → User: Retrieve role
8. AuthenticationControl → SessionManager: Create session
9. SessionManager → User: Update lastLoginDate
10. AuthenticationControl → AccountLockoutManager: Reset failed counter
11. AuthenticationControl → UserLoginUI: Redirect to role-specific dashboard

**Alternative Flow (Failed Login)**:

1. CredentialValidator → User: Invalid credentials
2. AuthenticationControl → AccountLockoutManager: Increment failed counter
3. AccountLockoutManager checks threshold (5 attempts in 15 min)
4. If threshold exceeded: AuthenticationControl enters lockout state (30 min)
5. UserLoginUI displays error/lockout message

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: AuthenticationControl

**States Identified**:

1. **AwaitingCredentials**: Initial state, waiting for login submission
2. **ValidatingCredentials**: Checking email and password
3. **CheckingAccountStatus**: Verifying account state (Active/Unverified/Suspended/Deleted)
4. **CheckingLockout**: Verifying lockout status
5. **CreatingSession**: Generating session token
6. **Authenticated**: Successful login, redirecting to dashboard
7. **CredentialsFailed**: Invalid credentials, incrementing counter
8. **AccountLocked**: Temporarily locked (30 minutes after 5 failed attempts)
9. **AccountUnverified**: Account exists but email not verified
10. **AccountSuspended**: Account suspended by admin
11. **PasswordResetRequested**: User requested password reset

**Events**:

- submitLogin
- credentialsValid / credentialsInvalid
- accountActive / accountUnverified / accountSuspended / accountDeleted
- lockoutActive / lockoutExpired
- thresholdExceeded
- sessionCreated
- requestPasswordReset
- resetEmailSent

---

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
    - UserLoginUI ← Traveler, HostelStaff, PlatformAdministrator (unified)
    - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
    - User ✅
    - Notification ✅
- [x] Control justified (state-dependent behavior with account status and lockout states)
- [x] State-dependent flagged (AuthenticationControl)
- [x] Logic justified (complex authentication, lockout logic, session management)

---

## Alternative Flows Mapping

**Step 5 Alternative** (Invalid credentials):

- CredentialValidator detects invalid email/password
- AuthenticationControl → CredentialsFailed state
- AccountLockoutManager increments counter
- If < 5 attempts: Returns to AwaitingCredentials
- If ≥ 5 attempts in 15 min: Transitions to AccountLocked state (30 min)

**Step 6 Alternatives** (Account status checks):

- **Unverified**: AuthenticationControl → AccountUnverified state → Display verification message
- **Suspended**: AuthenticationControl → AccountSuspended state → Display support message
- **Deleted**: AuthenticationControl → CredentialsFailed state → Display generic error (security)

**Step 3 Alternative** (Password reset):

- User → UserLoginUI: Click "Forgot Password"
- AuthenticationControl → PasswordResetRequested state
- AuthenticationControl → SessionManager: Generate reset token
- AuthenticationControl → Notification: Create reset email record
- AuthenticationControl → EmailServiceProxy: Send reset link
- EmailServiceProxy → EmailService: Deliver email

---

## Security Considerations

- **Generic error messages**: "Invalid email or password" prevents username enumeration
- **Account lockout**: Prevents brute force attacks (5 attempts in 15 min → 30 min lockout)
- **HTTPS only**: Credentials transmitted securely
- **Secure session tokens**: httpOnly, secure cookies prevent XSS/CSRF attacks
- **IP logging**: Failed attempts logged with IP for forensics
- **24-hour session timeout**: Balances security and usability
- **Password hashing**: Never stored in plain text (User entity constraint)

---

## Notes

- **Multi-role support**: Single login interface serves all user types (Traveler, Staff, Admin)
- **Role-based routing**: Post-authentication redirection based on User.role attribute
- **Lockout window**: 15-minute sliding window for 5 failed attempts
- **Lockout duration**: 30 minutes temporary block
- **Session expiration**: 24 hours inactivity timeout
- **Reusability**: CredentialValidator and SessionManager reusable across authentication flows
- **State tracking**: AuthenticationControl maintains audit trail for security monitoring
