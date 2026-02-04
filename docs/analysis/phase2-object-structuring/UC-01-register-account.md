# Object Structuring: Register Account

**Use Case**: docs/requirements/use-cases/UC-01-register-account.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Primary Actor**: Traveler
**External Class (Phase 1)**: Traveler («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Analysis**: The Traveler initiates registration by clicking "Sign Up" button and interacting with registration form. This is standard user interaction through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| TravelerRegistrationUI | «user interaction» | Maps from Traveler external user. Handles registration form display, input collection, and feedback display. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 7: "System creates user account" → User entity
- Step 13: "System activates account" → User entity (accountStatus update)
- Precondition: "Traveler does not have existing account" → User entity
- Postcondition: "Traveler account created and activated" → User entity
- Postcondition: "Account status set to 'Active'" → User entity

**Referenced Data**:
- **User**: Primary entity being created
  - Attributes used: userId, email, password, fullName, role, accountStatus, createdDate
- **Notification**: Implicit - "System sends verification email" creates notification record

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| User | Yes | ✅ | Created (step 7), updated (step 13), queried (step 6) |
| Notification | Yes | ✅ | Created (step 8) for verification email tracking |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (success messages, error messages, confirmation)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing TravelerRegistrationUI)

**Analysis**:
- Steps 2, 5, 6, 9, 14: System displays various messages via web interface
- Step 8: System sends email via EmailService external system
- All screen output can be handled by same UI boundary object

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| TravelerRegistrationUI | «user interaction» | Reused for output - displays registration form, error messages, success messages, and confirmation |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends verification email and welcome email. |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Registration process has clear sequential states with state-dependent behavior:
- **Unregistered** state: Accepts registration submission
- **PendingVerification** state: Waiting for email verification (24-hour timeout)
- **Active** state: Account activated after verification
- Different responses to verification token based on state (expired, invalid, valid)
- Alternative flows depend on current state (Step 12 alternatives)

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| RegistrationControl | «state-dependent control» | ⚠️ Yes | Coordinates registration workflow through states: form submission → validation → account creation → email verification → activation. Behavior differs based on account state (PendingVerification vs Active). Handles timeout (24-hour verification window). |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (email format, password strength, uniqueness)
- [ ] Algorithms
- [x] Cross-entity validation (email availability check)

**Identified Logic**:
- **Registration validation**: Email format, password strength (8+ chars, mixed case, numbers, special chars), confirm password match, terms acceptance
- **Email uniqueness check**: Query User entity to verify email not already registered
- **Verification token validation**: Token format, expiration (24 hours), validity

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| RegistrationValidator | «business logic» | Encapsulates complex validation rules: email format validation, password strength requirements (8+ characters, mixed case, numbers, special characters), confirm password match, terms acceptance. Reusable across registration flows. |
| EmailVerificationService | «service» | Manages verification token generation, validation (format, expiration, validity), and email resend logic. Handles 24-hour expiration rule. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| TravelerRegistrationUI | «user interaction» | Boundary (I/O) | Traveler | Registration form display and user interaction |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Sends verification and welcome emails |
| RegistrationControl | «state-dependent control» | Control | N/A | Orchestrates registration workflow and state transitions |
| User | «entity» | Entity | User | Account data storage |
| Notification | «entity» | Entity | Notification | Email delivery tracking |
| RegistrationValidator | «business logic» | Application Logic | N/A | Registration data validation |
| EmailVerificationService | «service» | Application Logic | N/A | Verification token management |

**Total**: 7 objects (2 boundary, 2 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: Traveler
- **Secondary Actor**: EmailService
- **Boundary**: TravelerRegistrationUI, EmailServiceProxy
- **Control**: RegistrationControl
- **Entity**: User, Notification
- **Logic**: RegistrationValidator, EmailVerificationService

**Key Interactions**:
1. Traveler → TravelerRegistrationUI: Submit registration
2. TravelerRegistrationUI → RegistrationControl: Process registration
3. RegistrationControl → RegistrationValidator: Validate input
4. RegistrationControl → User: Check email availability
5. RegistrationControl → User: Create account
6. RegistrationControl → EmailVerificationService: Generate token
7. RegistrationControl → Notification: Create email record
8. RegistrationControl → EmailServiceProxy: Send verification email
9. EmailServiceProxy → EmailService: Deliver email
10. Traveler → TravelerRegistrationUI: Click verification link
11. TravelerRegistrationUI → RegistrationControl: Process verification
12. RegistrationControl → EmailVerificationService: Validate token
13. RegistrationControl → User: Activate account

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: RegistrationControl

**States Identified**:
1. **AwaitingRegistration**: Initial state, waiting for registration submission
2. **ValidatingInput**: Validating registration data
3. **CreatingAccount**: Creating user account in system
4. **PendingVerification**: Waiting for email verification (24-hour window)
5. **Activated**: Account successfully activated
6. **VerificationExpired**: Verification token expired (after 24 hours)
7. **RegistrationFailed**: Registration failed due to validation errors

**Events**:

- submitRegistration
- validationSuccess / validationFailed
- accountCreated
- emailSent / emailFailed
- verificationLinkClicked
- tokenValid / tokenInvalid / tokenExpired
- resendVerification

---

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
  - TravelerRegistrationUI ← Traveler
  - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
  - User ✅
  - Notification ✅
- [x] Control justified (state-dependent behavior identified)
- [x] State-dependent flagged (RegistrationControl)
- [x] Logic justified (complex validation and token management)

---

## Alternative Flows Mapping

**Step 5 Alternative** (Invalid format):
- RegistrationValidator detects errors
- RegistrationControl transitions to RegistrationFailed state
- TravelerRegistrationUI displays specific error messages
- Returns to AwaitingRegistration state

**Step 6 Alternative** (Email exists):
- User entity query returns existing record
- RegistrationControl transitions to RegistrationFailed state
- TravelerRegistrationUI displays "Email already exists" with options
- Returns to AwaitingRegistration state

**Step 8 Alternative** (Email delivery fails):
- EmailServiceProxy receives failure from EmailService
- RegistrationControl logs error but stays in PendingVerification state
- TravelerRegistrationUI displays retry message
- User remains in system but unverified

**Step 12 Alternatives** (Token validation failures):
- Token expired: EmailVerificationService detects expiration → VerificationExpired state
- Token invalid: EmailVerificationService detects invalid token → display error
- Both cases offer resend option → back to PendingVerification state

---

## Notes

- **Security**: Password never stored in plain text (hashing handled by User entity constraints)
- **Email uniqueness**: Critical business rule enforced at User entity level
- **24-hour timeout**: Managed by EmailVerificationService
- **Reusability**: RegistrationValidator and EmailVerificationService can be reused in other use cases (UC-08: Manage Profile, password reset flows)
- **State management**: RegistrationControl maintains clear state transitions for audit trail
