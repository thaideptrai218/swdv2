# Phase 2 Statechart Requirements

**Generated**: 2026-02-04

---

## Summary

**State-Dependent Controls**: 9/24 use cases

| UC | Control Name | States Count |
|----|--------------|--------------:|
| UC-01-REGISTER-ACCOUNT |  | TBD |
| UC-02-LOG-IN |  | TBD |
| UC-05-BOOK-ACCOMMODATION |  | TBD |
| UC-07-SUBMIT-REVIEW |  | TBD |
| UC-08-MANAGE-PROFILE |  | TBD |
| UC-09-REGISTER-HOSTEL |  | TBD |
| UC-13-MANAGE-STAFF |  | TBD |
| UC-15-MANAGE-SUBSCRIPTION |  | TBD |
| UC-24-CANCEL-BOOKING | CancellationControl | 7 |

---

## UC-01-REGISTER-ACCOUNT: Register Account

**Control**: 

**Full Phase 4 Section**:

```
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
```

---

## UC-02-LOG-IN: Log In

**Control**: 

**Full Phase 4 Section**:

```
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
```

---

## UC-05-BOOK-ACCOMMODATION: Book Accommodation

**Control**: 

**Full Phase 4 Section**:

```
**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: BookingControl

**States Identified**:
1. **VerifyingAuth**: Checking traveler authentication
2. **DisplayingForm**: Showing booking form with pre-filled data
3. **CollectingGuestInfo**: Awaiting guest information submission
4. **ValidatingInfo**: Validating guest information format
5. **CheckingAvailability**: Real-time availability validation
6. **CreatingPendingBooking**: Creating temporary booking record
7. **PendingPayment**: Waiting for payment completion (10-minute timeout)
8. **ProcessingPayment**: Payment being processed by gateway
9. **ConfirmingBooking**: Booking confirmed, sending notifications
10. **Completed**: Booking successful, confirmation displayed
11. **AvailabilityLost**: Room no longer available (race condition)
12. **PaymentFailed**: Payment declined or failed
13. **PaymentTimeout**: Payment took longer than 10 minutes
14. **ValidationFailed**: Guest information invalid

**Events**:
- initiateBooking
- authenticationVerified / authenticationRequired (→ UC-02)
- guestInfoSubmitted
- validationSuccess / validationFailed
- availabilityConfirmed / availabilityLost
- pendingBookingCreated
- paymentRedirect
- paymentSuccess / paymentFailed / paymentTimeout
- bookingConfirmed
- emailsSent
- displayConfirmation

---
```

---

## UC-07-SUBMIT-REVIEW: Submit Review

**Control**: 

**Full Phase 4 Section**:

```
**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: ReviewSubmissionControl

**States Identified**:
1. **VerifyingAuth**: Checking traveler authentication
2. **CheckingEligibility**: Validating completed stay and no existing review
3. **DisplayingForm**: Showing review form with booking context
4. **CollectingReview**: Awaiting traveler input (ratings, text)
5. **UploadingPhotos**: Optional photo upload in progress
6. **ValidatingContent**: Checking text for prohibited content
7. **ValidatingPhotos**: Checking photo formats and sizes
8. **SavingReview**: Creating Review entity with Pending status
9. **PendingModeration**: Queued for moderation (up to 24 hours)
10. **Published**: Review approved and live on hostel page
11. **Rejected**: Review rejected by moderation
12. **EditWindow**: 7-day window after publication allowing edits
13. **EligibilityFailed**: Not eligible (future checkout or already reviewed)
14. **ValidationFailed**: Content or photo validation errors

**Events**:
- initiateReview
- authenticationVerified / authenticationRequired (→ UC-02)
- eligibilityConfirmed / eligibilityFailed
- ratingsEntered
- textEntered
- photosUploaded / skipPhotos
- submitReview
- contentValid / contentInvalid
- photosValid / photosInvalid
- reviewSaved
- moderationApproved / moderationRejected
- ratingsRecalculated
- editRequested (within 7 days)

---
```

---

## UC-08-MANAGE-PROFILE: Manage Profile

**Control**: 

**Full Phase 4 Section**:

```
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
```

---

## UC-09-REGISTER-HOSTEL: Register Hostel

**Control**: 

**Full Phase 4 Section**:

```
**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: PropertyRegistrationControl

**States Identified**:
1. **WizardStart**: Initial state, wizard not started
2. **Step1_BusinessInfo**: Collecting business information
3. **Step1_ValidatingBusiness**: Validating business info and documents
4. **Step2_PropertyDetails**: Collecting property details
5. **Step2_ValidatingProperty**: Validating property details and address
6. **Step3_Amenities**: Selecting amenities and facilities
7. **Step4_Policies**: Entering policies and rules
8. **Step5_BankAccount**: Collecting bank information and verification doc
9. **Step5_ValidatingBank**: Validating bank account document
10. **ReviewingSummary**: Owner reviewing complete registration
11. **Submitting**: Processing registration submission
12. **PendingVerification**: Awaiting admin review (2-3 business days)
13. **Approved**: Admin approved, property active
14. **AdditionalInfoNeeded**: Admin requests more information
15. **Rejected**: Admin rejected registration
16. **Draft**: Saved for later completion

**Events**:
- startWizard
- enterBusinessInfo
- uploadLicenseDoc
- clickNext / clickBack
- validationSuccess / validationFailed
- enterPropertyDetails
- selectAmenities
- enterPolicies
- enterBankInfo
- uploadBankDoc
- reviewSummary
- submitRegistration
- allStepsValid / stepsIncomplete
- registrationSubmitted
- adminApproved / adminRequestsInfo / adminRejected
- ownerUpdatesInfo
- saveDraft
- resumeDraft

---
```

---

## UC-13-MANAGE-STAFF: Manage Staff

**Control**: 

**Full Phase 4 Section**:

```
**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: StaffManagementControl

**States Identified**:
1. **NoInvitation**: Initial state, no pending invitation
2. **CreatingInvitation**: Processing invitation form
3. **ValidatingInvitation**: Checking email, duplicates
4. **PendingInvitation**: Invitation sent, awaiting registration (7-day window)
5. **InvitationExpired**: Token expired after 7 days
6. **StaffRegistering**: Staff clicked link, creating/logging into account
7. **ValidatingToken**: Checking token status (valid/used/expired)
8. **Active**: Staff registered and has property access
9. **UpdatingPermissions**: Modifying staff permissions
10. **Suspended**: Access temporarily revoked
11. **Removed**: Staff association deleted
12. **InvitationCancelled**: Owner cancelled pending invitation

**Events**:
- inviteStaff
- validationSuccess / validationFailed / duplicateEmail
- tokenGenerated
- invitationSent
- tokenExpired (after 7 days)
- staffClickedLink
- tokenValid / tokenExpired / tokenUsed
- staffRegistered
- permissionsApplied
- editPermissions
- permissionsUpdated
- suspendStaff
- accessRevoked
- removeStaff
- associationDeleted
- cancelInvitation
- resendInvitation

---
```

---

## UC-15-MANAGE-SUBSCRIPTION: Manage Subscription

**Control**: 

**Full Phase 4 Section**:

```
**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: SubscriptionControl

**States Identified**:
1. **NoSubscription**: Owner account created, no subscription yet
2. **TrialActive**: 14-day Professional trial (BR-080)
3. **TrialExpiring**: 7 days or less remaining
4. **TrialExpired**: Trial ended, no payment method
5. **Active**: Subscription paid, features enabled
6. **Upgrading**: Processing mid-cycle upgrade
7. **DowngradeScheduled**: Downgrade scheduled for next billing cycle
8. **PastDue**: Payment failed, in 3-day grace period (BR-082)
9. **Restricted**: Grace period expired, features limited
10. **Cancelled**: Cancellation scheduled for period end
11. **CancelledDowngraded**: Period ended, downgraded to Free
12. **PaymentMethodUpdating**: Updating payment card

**Events**:
- createSubscription
- startTrial
- trialWarning7Days / trialWarning1Day
- trialExpired
- paymentMethodAdded
- upgradePlan
- paymentSuccess / paymentFailed
- planUpgraded
- downgradePlan
- downgradeScheduled
- billingCycleEnd
- planDowngraded
- paymentDue
- paymentRetryFailed
- gracePeriodExpired
- featuresRestricted
- cancelSubscription
- cancellationScheduled
- updatePaymentMethod

---
```

---

## UC-24-CANCEL-BOOKING: Cancel Booking

**Control**: CancellationControl

**States**:

1. EligibleForFullRefund (< 24h since booking)
2. NoRefundEligible (> 24h since booking, > 24h to check-in)
3. LateCancellation (< 24h to check-in)
4. CheckedIn (cannot cancel)
5. ProcessingRefund
6. RefundCompleted
7. CancellationCompleted

**Full Phase 4 Section**:

```
**State-Dependent Controls**: 1

⚠️ **Statechart needed for: CancellationControl**

**Temporal Dependencies**:
- Time since booking (24h window)
- Time until check-in (24h window)
- Check-in status (boolean state)
- Refund processing status

**State Transitions**:
- Booking created → EligibleForFullRefund
- 24h elapsed → NoRefundEligible
- < 24h to check-in → LateCancellation
- Guest checked in → CheckedIn (terminal - no cancel)
- Cancel requested → ProcessingRefund
- Refund confirmed → RefundCompleted → CancellationCompleted

---
```

---
