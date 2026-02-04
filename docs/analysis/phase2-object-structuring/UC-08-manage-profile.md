# Object Structuring: Manage Profile

**Use Case**: docs/requirements/use-cases/UC-08-manage-profile.md
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

**Analysis**: Traveler initiates profile management via navigation menu, updates various profile sections through web interface, uploads photo, changes email/password.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ProfileManagementUI | «user interaction» | Maps from Traveler external user. Handles profile sections display (personal info, photo, password, preferences, security), inline editing, photo upload with crop tool, form validation feedback, confirmation messages. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: "Retrieves current profile information" → User, UserProfile entities
- Step 3-4: "Personal information section" → User, UserProfile entities
- Step 5-8: "Updates profile information" → User, UserProfile entities
- Profile Photo Flow: Photo upload → Photo entity or User.profilePhotoUrl
- Email Change Flow: Verification → User entity (email), Notification entity
- Password Change Flow: Password update → User entity (password), Notification entity (security notification)
- Communication Preferences Flow: Toggle preferences → UserProfile entity (preferences)
- Account Security Flow: Login history, sessions → Implied session tracking, User.lastLoginDate

**Referenced Data**:
- **User**: Primary account entity
  - Attributes: userId, email, password, fullName, lastLoginDate
- **UserProfile**: Extended profile information
  - Attributes: profileId, phoneNumber, dateOfBirth, language, preferences
- **Notification**: Email notifications for security changes
  - Attributes: notificationId, type, recipientId, content
- **Photo**: Profile photo (or profilePhotoUrl in User entity)
  - Attributes: url, uploadDate

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| User | Yes | ✅ | Account data (email, password, fullName, lastLoginDate) updated |
| UserProfile | Yes | ✅ | Extended profile (phone, dateOfBirth, language, preferences) updated |
| Notification | Yes | ✅ | Security notification emails (email change, password change) |
| Photo | Yes | ✅ | Profile photo upload (or User.profilePhotoUrl attribute) |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (profile sections, success messages, validation errors)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing ProfileManagementUI)

**Analysis**:
- Steps 3, 9-10: Display profile sections and confirmation messages
- Profile Photo Flow: Photo upload to cloud storage
- Email Change Flow: Send verification email
- Password Change Flow: Send security notification email
- All screen output via same UI boundary

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ProfileManagementUI | «user interaction» | Reused for output - displays profile sections, inline validation errors, success messages, unsaved changes warning, email verification notice |
| CloudStorageProxy | «proxy» | Maps from CloudStorage external system. Uploads profile photo, provides crop tool preview, returns photo URL. Reused from UC-04, UC-07, UC-10. |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends email verification link (email change), sends security notification (email/password changes). Reused from UC-01, UC-02, UC-05. |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Profile management has state-dependent sub-flows:
- **Basic profile updates**: Stateless (simple updates)
- **Email change flow**: State-dependent (verification required)
  - States: Request change → Verification email sent → Pending verification (24-hour timeout) → Verified and updated / Expired
- **Password change flow**: State-dependent (validation + security notification)
  - States: Request change → Validate current → Validate new → Updated → Security notification sent
- Different behavior based on verification state (email pending, password changed)

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| ProfileUpdateControl | «state-dependent control» | ⚠️ Yes | Orchestrates profile update workflows. Basic updates are stateless, but email change (verification flow with 24-hour timeout) and password change (current password validation + security notification) require state management. Coordinates photo upload, preference saves, security changes. |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (email format, password strength, age 18+, phone format)
- [ ] Algorithms
- [x] Cross-entity validation (email uniqueness for email changes)

**Identified Logic**:
- **Profile validation**: Real-time phone format, age validation (18+), email format (RFC 5322)
- **Password strength validation**: 8+ chars, mixed case, numbers, special characters
- **Email verification**: Token generation, 24-hour expiration, email uniqueness check
- **Photo validation**: Format (JPG/PNG), size (max 5MB), crop tool
- **Security notifications**: Trigger emails for sensitive changes

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ProfileInfoValidator | «business logic» | Validates profile information: phone format (international), date of birth (18+ requirement per BR-039), name format. Real-time validation with inline feedback. |
| EmailChangeService | «service» | Manages email change workflow: validates new email format, checks uniqueness against User table, generates verification token, sends verification email, validates token (24-hour expiration), updates User.email. Prevents account hijacking. |
| PasswordChangeService | «service» | Manages password change workflow: validates current password, validates new password strength (8+ chars, mixed case, numbers, special chars per UC-01 requirements), validates password match, updates User.password (hashed), sends security notification email (BR-038). |
| ProfilePhotoService | «service» | Manages profile photo upload: validates format/size, provides crop tool interface, uploads to CloudStorage via proxy, updates User profile photo URL. Photo displayed on reviews/bookings (BR-041). |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| ProfileManagementUI | «user interaction» | Boundary (I/O) | Traveler | Profile sections and editing interface |
| CloudStorageProxy | «proxy» | Boundary (Output) | CloudStorage | Profile photo uploads |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Verification and security emails |
| ProfileUpdateControl | «state-dependent control» | Control | N/A | Profile update orchestration with email/password state flows |
| User | «entity» | Entity | User | Account data (email, password, fullName) |
| UserProfile | «entity» | Entity | UserProfile | Extended profile data |
| Notification | «entity» | Entity | Notification | Email notifications |
| Photo | «entity» | Entity | Photo | Profile photo |
| ProfileInfoValidator | «business logic» | Application Logic | N/A | Profile data validation |
| EmailChangeService | «service» | Application Logic | N/A | Email change workflow |
| PasswordChangeService | «service» | Application Logic | N/A | Password change workflow |
| ProfilePhotoService | «service» | Application Logic | N/A | Photo upload management |

**Total**: 12 objects (3 boundary, 4 entity, 1 control, 4 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: Traveler
- **Secondary Actors**: CloudStorage (photo), EmailService (verification/security)
- **Boundary**: ProfileManagementUI, CloudStorageProxy, EmailServiceProxy
- **Control**: ProfileUpdateControl
- **Entity**: User, UserProfile, Notification, Photo
- **Logic**: ProfileInfoValidator, EmailChangeService, PasswordChangeService, ProfilePhotoService

**Key Interactions (Basic Profile Update)**:
1. Traveler → ProfileManagementUI: Navigate to profile
2. ProfileManagementUI → ProfileUpdateControl: Request profile data
3. ProfileUpdateControl → User: Get account data
4. ProfileUpdateControl → UserProfile: Get extended profile
5. ProfileUpdateControl → ProfileManagementUI: Display sections
6. Traveler → ProfileManagementUI: Edit personal info
7. ProfileManagementUI → ProfileInfoValidator: Validate (real-time)
8. Traveler → ProfileManagementUI: Save changes
9. ProfileManagementUI → ProfileUpdateControl: Submit updates
10. ProfileUpdateControl → ProfileInfoValidator: Final validation
11. ProfileUpdateControl → UserProfile: Update data
12. ProfileUpdateControl → ProfileManagementUI: Display success

**Email Change Flow**:
1. Traveler → ProfileManagementUI: Click "Change Email"
2. ProfileManagementUI → ProfileUpdateControl: Initiate email change
3. ProfileUpdateControl → EmailChangeService: Start workflow
4. EmailChangeService → User: Validate current password
5. EmailChangeService: Generate verification token
6. EmailChangeService → Notification: Create verification email
7. EmailChangeService → EmailServiceProxy: Send email
8. EmailServiceProxy → EmailService: Deliver email
9. EmailChangeService → ProfileManagementUI: Display "Verification sent"
10. Traveler clicks link in email (24-hour window)
11. EmailChangeService: Validate token
12. EmailChangeService → User: Update email
13. EmailChangeService → ProfileManagementUI: Display success

**Password Change Flow**:
1. Traveler → ProfileManagementUI: Click "Change Password"
2. ProfileManagementUI → ProfileUpdateControl: Initiate password change
3. ProfileUpdateControl → PasswordChangeService: Start workflow
4. PasswordChangeService → User: Validate current password
5. PasswordChangeService: Validate new password strength
6. PasswordChangeService → User: Update password (hashed)
7. PasswordChangeService → Notification: Create security notification
8. PasswordChangeService → EmailServiceProxy: Send notification
9. PasswordChangeService → ProfileManagementUI: Display success

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: ProfileUpdateControl

**States Identified**:
1. **ViewingProfile**: Displaying current profile sections
2. **EditingBasicInfo**: Updating name, phone, DOB (stateless updates)
3. **ValidatingBasicInfo**: Real-time validation
4. **SavingBasicInfo**: Persisting changes to UserProfile
5. **UploadingPhoto**: Photo upload in progress
6. **CroppingPhoto**: Adjusting photo crop area
7. **EmailChangeRequested**: User requested email change
8. **ValidatingCurrentPassword**: Verifying identity for email change
9. **PendingEmailVerification**: Awaiting email link click (24-hour window)
10. **EmailVerificationExpired**: Verification link expired after 24 hours
11. **EmailUpdated**: Email successfully changed
12. **PasswordChangeRequested**: User requested password change
13. **ValidatingPasswordChange**: Checking current password and new password strength
14. **PasswordUpdated**: Password successfully changed, notification sent
15. **UpdatingPreferences**: Toggling communication preferences (stateless)

**Events**:
- viewProfile
- editBasicInfo
- validationSuccess / validationFailed
- saveChanges
- uploadPhoto
- cropPhoto
- savePhoto
- requestEmailChange
- currentPasswordValid / currentPasswordInvalid
- verificationEmailSent
- verificationLinkClicked
- tokenValid / tokenExpired
- emailUpdated
- requestPasswordChange
- newPasswordValid / newPasswordInvalid
- passwordUpdated
- updatePreferences

---

## Validation

- [x] At least 1 boundary object (3 identified)
- [x] Boundary → External mapping (1:1):
  - ProfileManagementUI ← Traveler
  - CloudStorageProxy ← CloudStorage
  - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
  - User ✅
  - UserProfile ✅
  - Notification ✅
  - Photo ✅
- [x] Control justified (state-dependent for email/password change workflows)
- [x] State-dependent flagged (ProfileUpdateControl)
- [x] Logic justified (validation, email change, password change, photo upload)

---

## Alternative Flows Mapping

**Step 5 Alternative** (Invalid format):
- ProfileInfoValidator detects errors (phone format, age <18)
- ProfileUpdateControl prevents saving
- ProfileManagementUI displays inline error messages

**Step 7 Alternative** (Validation fails):
- ProfileInfoValidator detects multiple errors
- ProfileUpdateControl highlights problematic fields
- ProfileManagementUI displays specific errors

**Profile Photo Flow**:
1. Traveler uploads image
2. ProfilePhotoService validates (JPG/PNG, ≤5MB)
3. If invalid: Error displayed, retry allowed
4. ProfileManagementUI displays crop tool
5. ProfilePhotoService → CloudStorageProxy uploads
6. Photo URL returned, User profile updated

**Email Change Flow Alternatives**:
- Current password invalid: Error, offer "Forgot Password"
- Email already exists: Error message
- Verification link expired (>24 hours): Display error, offer resend
- EmailService delivery fails: Error logged, retry option

**Password Change Flow Alternatives**:
- Current password invalid: Error, offer "Forgot Password"
- New password weak: Strength indicator shows requirements
- Passwords don't match: Error message
- Security notification email fails: Logged, but password still changed

**Communication Preferences Flow**:
- Immediate save on toggle (no explicit save button)
- Preferences respected system-wide (BR-040)

**Account Security Flow**:
- Display last 10 logins (date, location, device)
- View active sessions, logout others

---

## Business Rules

- **BR-037**: Profile changes effective immediately (enforced by ProfileUpdateControl)
- **BR-038**: Email/password changes trigger security notifications (PasswordChangeService, EmailChangeService)
- **BR-039**: Age 18+ enforced (ProfileInfoValidator)
- **BR-040**: Communication preferences respected (UserProfile.preferences)
- **BR-041**: Profile photo on reviews/bookings (Photo entity linkage)

---

## Security Considerations

- **Password verification**: Current password required for email/password changes
- **Email verification**: New email must be verified before change (prevents hijacking)
- **Password never displayed**: Show as dots, never sent to client
- **Session refresh**: Security changes refresh session tokens
- **Password strength**: Enforced 8+ chars, mixed case, numbers, special chars
- **Security notifications**: Email alerts for sensitive changes
- **Data encryption**: Personal data encrypted at rest and in transit

---

## Notes

- **State-dependent flows**: Email change (24-hour verification) and password change require state management
- **Basic updates**: Name, phone, DOB are stateless updates
- **Photo crop tool**: Enhances UX for profile photo quality
- **Real-time validation**: Inline feedback improves form completion
- **Communication preferences**: Immediate save reduces friction
- **Security features**: Login history, active sessions build trust
- **GDPR considerations**: Data privacy, deletion rights mentioned in outstanding questions
- **Reusability**: CloudStorageProxy, EmailServiceProxy reused across multiple UCs
- **Profile completeness**: Affects booking trust and review credibility
