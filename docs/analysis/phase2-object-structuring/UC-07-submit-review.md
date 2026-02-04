# Object Structuring: Submit Review

**Use Case**: docs/requirements/use-cases/UC-07-submit-review.md
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

**Analysis**: Traveler initiates review submission from UC-06 or email prompt, enters ratings and text, uploads photos through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ReviewSubmissionUI | «user interaction» | Maps from Traveler external user. Handles review form display with rating criteria, text editor with character counter, photo upload interface, submission confirmation. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 3: "Verifies completed stay" → Booking entity (checkOutDate validation)
- Step 4: "Checks not already reviewed" → Review entity (existence check)
- Step 5.1: "Shows booking details" → Booking, Hostel, RoomType entities
- Step 6-7: "Ratings and review text" → Review entity (being created)
- Step 8: "Uploads photos" → Photo entity (review photos, separate from hostel photos)
- Step 11: "Saves review with pending status" → Review entity
- Step 14-15: "Publishes review, recalculates ratings" → Review, Hostel entities
- Step 16: "Marks booking as reviewed" → Booking entity (reviewed flag)
- Step 17: "Displays on hostel page" → Review entity (status = Published)
- Includes: UC-02 (Log In)

**Referenced Data**:
- **Review**: Primary entity being created
  - Attributes: reviewId, rating, comment, reviewDate, status (Pending/Published/Rejected)
- **Booking**: Validated for eligibility
  - Attributes: bookingId, checkOutDate, travelerreviewed flag
- **Hostel**: Review target
  - Attributes: hostelId, rating (aggregate), reviewCount
- **RoomType**: Displayed in context
  - Attributes: name
- **User**: Reviewer (traveler)
  - Attributes: userId, fullName
- **Photo**: Optional review photos
  - Attributes: photoId, url, uploadDate

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Review | Yes | ✅ | Created (step 11), published (step 14), displayed (step 17) |
| Booking | Yes | ✅ | Eligibility validation (steps 3-4), marked reviewed (step 16) |
| Hostel | Yes | ✅ | Review target (step 5.1), ratings updated (step 15) |
| RoomType | Yes | ✅ | Context display (step 5.1) |
| User | Yes | ✅ | Reviewer identity (authentication, review attribution) |
| Photo | Yes | ✅ | Optional review photos (step 8) - may need review photo variant |

**Note**: Photo entity used for both hostel photos and review photos. May need to distinguish via attribute or context.

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (review form, confirmation, moderation message)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing ReviewSubmissionUI)

**Analysis**:
- Steps 5, 13: Display review form and confirmation
- Step 8: Photo upload to cloud storage (reusing CloudStorageProxy from UC-04/UC-10)
- No external actor notifications (moderation is internal process)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ReviewSubmissionUI | «user interaction» | Reused for output - displays form with rating stars, text area with character counter, photo upload preview, validation errors, confirmation message, moderation notice |
| CloudStorageProxy | «proxy» | Maps from CloudStorage external system (Phase 1). Uploads review photos (step 8). Validates format/size. Returns photo URLs. Reused from UC-04, UC-10. |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Review submission has state-dependent workflow:
- **VerifyingEligibility** state: Checking authentication, completed stay, not reviewed
- **CollectingReview** state: Gathering ratings and text
- **UploadingPhotos** state: Optional photo upload
- **ValidatingContent** state: Content moderation checks
- **PendingModeration** state: Awaiting moderator approval (up to 24 hours)
- **Published** state: Review approved and live
- **Rejected** state: Review rejected, can resubmit
- State transitions based on moderation outcome
- Time-based state: 7-day edit window (BR-033)

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| ReviewSubmissionControl | «state-dependent control» | ⚠️ Yes | Orchestrates review workflow through states: eligibility verification → content collection → photo upload → validation → pending moderation → published/rejected. Manages 24-hour moderation timeout, 7-day edit window. Behavior differs based on moderation state. |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (character count, prohibited content, photo format)
- [x] Algorithms (rating aggregation, content filtering)
- [x] Cross-entity validation (completed stay verification, duplicate review check)

**Identified Logic**:
- **Eligibility validation**: Authentication check, checkOutDate < today, no existing review for booking
- **Content validation**: Character limits (10-2000), profanity filter, personal info detection, spam detection
- **Photo validation**: Format (JPG/PNG), size (max 5MB), count (max 5)
- **Rating aggregation**: Calculate hostel average ratings (overall and individual aspects)
- **Moderation rules**: Automated filters + manual review for suspicious patterns

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ReviewEligibilityChecker | «business logic» | Validates traveler can submit review: authentication verified, booking exists, checkOutDate < today (BR-031), not already reviewed (BR-033 one review per booking). |
| ReviewContentValidator | «business logic» | Validates review content: minimum 10 characters (BR-034), maximum 2000, profanity filter, personal information detection (email, phone), spam patterns. Returns specific error messages. |
| ReviewPhotoValidator | «business logic» | Validates uploaded photos: format (JPG/PNG only), size (≤5MB per photo), count (≤5 photos total). Validates each photo individually with specific errors. |
| RatingAggregator | «algorithm» | Recalculates hostel aggregate ratings after review publication (BR-035): overall rating average, individual aspect averages (cleanliness, location, staff, facilities, value), review count. Updates Hostel entity immediately. |
| ModerationService | «service» | Manages review moderation queue: automated content filtering, flags for manual review, moderation within 24 hours (BR-032), approval/rejection processing, rejection reason notification. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| ReviewSubmissionUI | «user interaction» | Boundary (I/O) | Traveler | Review form and submission interface |
| CloudStorageProxy | «proxy» | Boundary (Output) | CloudStorage | Review photo uploads |
| ReviewSubmissionControl | «state-dependent control» | Control | N/A | Review workflow orchestration with moderation states |
| Review | «entity» | Entity | Review | Review record |
| Booking | «entity» | Entity | Booking | Eligibility validation |
| Hostel | «entity» | Entity | Hostel | Review target, ratings aggregate |
| RoomType | «entity» | Entity | RoomType | Context display |
| User | «entity» | Entity | User | Reviewer identity |
| Photo | «entity» | Entity | Photo | Review photos |
| ReviewEligibilityChecker | «business logic» | Application Logic | N/A | Eligibility validation |
| ReviewContentValidator | «business logic» | Application Logic | N/A | Content validation |
| ReviewPhotoValidator | «business logic» | Application Logic | N/A | Photo validation |
| RatingAggregator | «algorithm» | Application Logic | N/A | Aggregate ratings calculation |
| ModerationService | «service» | Application Logic | N/A | Moderation queue management |

**Total**: 14 objects (2 boundary, 6 entity, 1 control, 5 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: Traveler
- **Secondary Actors**: CloudStorage (photo uploads)
- **Boundary**: ReviewSubmissionUI, CloudStorageProxy
- **Control**: ReviewSubmissionControl (+ AuthenticationControl from UC-02 if needed)
- **Entity**: Review, Booking, Hostel, RoomType, User, Photo
- **Logic**: ReviewEligibilityChecker, ReviewContentValidator, ReviewPhotoValidator, RatingAggregator, ModerationService

**Key Interactions**:
1. Traveler → ReviewSubmissionUI: Click "Leave Review"
2. ReviewSubmissionUI → ReviewSubmissionControl: Initiate review
3. ReviewSubmissionControl → User: Verify authentication
4. [If not authenticated: Include UC-02 login flow]
5. ReviewSubmissionControl → ReviewEligibilityChecker: Check eligibility
6. ReviewEligibilityChecker → Booking: Get checkOutDate
7. ReviewEligibilityChecker → Review: Check existing review for booking
8. ReviewSubmissionControl → Booking: Get booking details
9. ReviewSubmissionControl → Hostel: Get hostel info
10. ReviewSubmissionControl → RoomType: Get room info
11. ReviewSubmissionControl → ReviewSubmissionUI: Display review form
12. Traveler → ReviewSubmissionUI: Enter ratings and text
13. Traveler → ReviewSubmissionUI: Upload photos (optional)
14. ReviewSubmissionUI → CloudStorageProxy: Upload photos
15. CloudStorageProxy → CloudStorage: Store photos
16. CloudStorage → CloudStorageProxy: Return photo URLs
17. Traveler → ReviewSubmissionUI: Submit review
18. ReviewSubmissionUI → ReviewSubmissionControl: Process submission
19. ReviewSubmissionControl → ReviewContentValidator: Validate text
20. ReviewSubmissionControl → ReviewPhotoValidator: Validate photos
21. ReviewSubmissionControl → Review: Create with Pending status
22. ReviewSubmissionControl → ModerationService: Queue for moderation
23. ReviewSubmissionControl → ReviewSubmissionUI: Display confirmation
24. ModerationService: [Automated + manual review within 24 hours]
25. ModerationService → ReviewSubmissionControl: Approval/rejection
26. If approved: ReviewSubmissionControl → Review: Update status to Published
27. ReviewSubmissionControl → RatingAggregator: Recalculate hostel ratings
28. RatingAggregator → Review: Get all published reviews
29. RatingAggregator → Hostel: Update aggregate ratings
30. ReviewSubmissionControl → Booking: Mark as reviewed

---

## Phase 4 Requirements

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

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
  - ReviewSubmissionUI ← Traveler
  - CloudStorageProxy ← CloudStorage
- [x] Entities in Phase 1:
  - Review ✅
  - Booking ✅
  - Hostel ✅
  - RoomType ✅
  - User ✅
  - Photo ✅
- [x] Control justified (state-dependent with moderation workflow)
- [x] State-dependent flagged (ReviewSubmissionControl)
- [x] Logic justified (eligibility checks, content validation, rating aggregation, moderation)

---

## Alternative Flows Mapping

**Step 1 Alternative** (Not authenticated):
- ReviewSubmissionControl checks User authentication
- If not authenticated: Include UC-02 (Log In)
- After login: Returns to VerifyingEligibility state

**Step 3 Alternative** (Future checkout):
- ReviewEligibilityChecker detects checkOutDate > today
- ReviewSubmissionControl → EligibilityFailed state
- ReviewSubmissionUI displays "Review available after check-out on [date]"

**Step 4 Alternative** (Already reviewed):
- ReviewEligibilityChecker finds existing Review for booking
- ReviewSubmissionControl checks review date
- If within 7 days: Transition to EditWindow state, allow editing
- If > 7 days: Display existing review, no edit allowed

**Step 10.2 Alternative** (Prohibited content):
- ReviewContentValidator detects profanity/personal info/spam
- ReviewSubmissionControl → ValidationFailed state
- ReviewSubmissionUI displays specific errors: "Remove inappropriate language", "Remove email addresses"
- Returns to CollectingReview state

**Step 10.3 Alternative** (Photo validation fails):
- ReviewPhotoValidator detects wrong format or size >5MB
- ReviewSubmissionControl → ValidationFailed state
- ReviewSubmissionUI displays per-photo errors
- Allows removal/replacement of problematic photos

**Step 14 Alternative** (Moderation rejects):
- ModerationService determines review is spam/fake/inappropriate
- ReviewSubmissionControl → Rejected state
- Email notification sent to traveler with reason
- Allows resubmission with corrections

**Step 8 Alternative** (Photo upload fails):
- CloudStorageProxy receives error from CloudStorage
- ReviewSubmissionControl logs error
- ReviewSubmissionUI displays error, allows retry or skip photos

---

## Include Relationship: UC-02 (Log In)

**Integration Point**: Step 1-2
- ReviewSubmissionControl checks authentication
- If User not authenticated:
  - Delegates to AuthenticationControl (UC-02)
  - AuthenticationControl handles complete login flow
  - On success: Returns to ReviewSubmissionControl
  - On failure: Review cancelled
- Review form preserves booking context during login

---

## Business Rules

- **BR-031**: Only verified guests with completed stay can review (enforced by ReviewEligibilityChecker)
- **BR-032**: Reviews moderated within 24 hours (ModerationService SLA)
- **BR-033**: One review per booking, edits allowed 7 days (ReviewEligibilityChecker + ReviewSubmissionControl)
- **BR-034**: Minimum 10 characters (ReviewContentValidator)
- **BR-035**: Aggregate ratings updated immediately (RatingAggregator)
- **BR-036**: Review photos moderated (ModerationService)

---

## Notes

- **State-dependent**: Moderation workflow creates distinct states (Pending → Published/Rejected)
- **Eligibility critical**: Ensures review authenticity (only real guests)
- **Content moderation**: Automated + manual maintains quality
- **Rating aggregation**: Immediate update provides real-time social proof
- **Photo uploads**: Enhance credibility via CloudStorageProxy
- **7-day edit window**: Allows corrections while preventing abuse
- **Character minimum**: Ensures substantive feedback (BR-034)
- **Include UC-02**: Authentication required before review
- **Reusability**: CloudStorageProxy reused from UC-04, UC-10
