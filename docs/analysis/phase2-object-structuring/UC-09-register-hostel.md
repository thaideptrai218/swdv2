# Object Structuring: Register Hostel

**Use Case**: docs/requirements/use-cases/UC-09-register-hostel.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Primary Actor**: Hostel Owner
**External Class (Phase 1)**: HostelOwner («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Analysis**: Owner initiates property registration via dashboard button, completes 5-step wizard (business info, property details, amenities, policies, bank account), uploads documents through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| PropertyRegistrationUI | «user interaction» | Maps from HostelOwner external user. Handles 5-step wizard display with progress indicator, form input for each step, document upload (business license, bank verification), navigation (Next/Back/Save Draft), validation feedback, submission confirmation. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 3-6: "Business Information" → Hostel entity or separate Business entity (businessName, registrationNumber, taxID, contactInfo)
- Step 6-8: "Property Details" → Hostel entity
- Step 9: "Amenities & Facilities" → Amenity entity (many-to-many)
- Step 11: "Policies & Rules" → Hostel entity (cancellationPolicy, houseRules, paymentTerms)
- Step 13: "Bank Account Information" → Hostel entity or separate BankAccount entity (bankName, accountNumber, accountHolder)
- Step 3.4, 13.4: "Document uploads" → Photo entity or Document entity
- Step 18: "Creates property listing" → Hostel entity (status = "Pending Verification")
- Step 19-20: "Confirmation emails" → Notification entity
- Precondition: "Owner has registered user account" → User entity

**Referenced Data**:
- **Hostel**: Property being registered
  - Attributes: hostelId, name, description, address, city, propertyType, totalRooms, capacity, amenities, policies, status (Pending/Active/Rejected), ownerId, createdDate
- **Location**: Property location (geocoded during registration or later)
  - Attributes: locationId, address, city, coordinates
- **Amenity**: Selected amenities (many-to-many)
  - Attributes: amenityId, name, category
- **User**: Owner account
  - Attributes: userId, role (Owner)
- **Notification**: Confirmation and verification emails
  - Attributes: notificationId, recipientId, type, content
- **Business/Bank Info**: May be attributes of Hostel or separate entities

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Hostel | Yes | ✅ | Property created (step 18), status = "Pending Verification" |
| Location | Yes | ✅ | Property location (address from step 6.2) |
| Amenity | Yes | ✅ | Selected amenities (step 9), many-to-many relationship |
| User | Yes | ✅ | Owner reference (ownerId), authentication |
| Notification | Yes | ✅ | Confirmation emails (steps 19-20) |

**Note**: Business license and bank documents stored as files, may need Document entity or use Photo entity for file storage.

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (wizard steps, progress, confirmation)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing PropertyRegistrationUI)

**Analysis**:
- Steps 2-21: Display wizard, progress, validation errors, confirmation
- Steps 3.4, 13.4: Document upload to cloud storage
- Steps 19-20: Send confirmation emails
- Step 20: Notify admin (internal system process)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| PropertyRegistrationUI | «user interaction» | Reused for output - displays wizard steps with progress indicator, validation errors per step, document upload status, final confirmation message with next steps |
| CloudStorageProxy | «proxy» | Maps from CloudStorage external system. Uploads business license and bank verification documents (steps 3.4, 13.4). Returns document URLs. Reused from UC-04, UC-07, UC-08, UC-10. |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends registration confirmation to owner (step 19), sends verification request to admin (step 20), sends approval/rejection emails after admin review. Reused from multiple UCs. |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Property registration is a multi-step wizard with state-dependent behavior:
- **Step 1**: Business Information (incomplete → complete)
- **Step 2**: Property Details (incomplete → complete)
- **Step 3**: Amenities & Facilities (incomplete → complete)
- **Step 4**: Policies & Rules (incomplete → complete)
- **Step 5**: Bank Account (incomplete → complete)
- **Submitted**: Pending admin verification (2-3 business days)
- **Pending Verification**: Awaiting admin review
- **Approved**: Property activated
- **Additional Info Needed**: Owner must update and resubmit
- **Rejected**: Can appeal or create new registration
- State persists as draft (Save and Continue Later)
- Back/forward navigation preserves step data

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| PropertyRegistrationControl | «state-dependent control» | ⚠️ Yes | Orchestrates 5-step wizard workflow: business info → property details → amenities → policies → bank info → submission → pending verification → approved/additional info needed/rejected. Manages wizard state (current step, completion status), draft save/resume, validation per step, admin review workflow (2-3 business day SLA). |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (business registration format, tax ID, address, document formats)
- [x] Algorithms (address validation, duplicate property detection)
- [x] Cross-entity validation (business registration uniqueness, owner property limits by subscription tier)

**Identified Logic**:
- **Business info validation**: Registration number format, tax ID format, license document (PDF/JPG, max 10MB)
- **Property validation**: Address format, description minimum 100 characters, property type selection
- **Document validation**: Business license and bank verification documents (format, size)
- **Address validation**: Attempt geocoding, flag for manual review if fails
- **Duplicate detection**: Check for existing property at same address
- **Admin verification**: Business license authenticity, tax ID verification, bank account verification
- **Property limits**: Check owner subscription tier limits (BR-053 allows multiple properties)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| BusinessInfoValidator | «business logic» | Validates business information: registration number format, tax ID format, business name uniqueness, contact information format. Required for compliance (BR-049). |
| PropertyDetailsValidator | «business logic» | Validates property details: address format, description length (≥100 chars per BR-056), property type selection, capacity/rooms > 0. Attempts address validation. |
| DocumentValidator | «business logic» | Validates uploaded documents: format (PDF/JPG), size (≤10MB per doc), readability checks. Required for BR-048, BR-049, BR-050. |
| AddressVerificationService | «service» | Validates property address: attempts geocoding, checks for duplicates at same address. Flags for manual admin review if cannot geocode. Address accuracy critical for traveler search. |
| RegistrationWorkflowManager | «service» | Manages admin verification workflow: sends registration to admin queue, tracks verification status (Pending/Approved/Additional Info/Rejected), enforces 2-3 business day SLA (BR-048), sends notification emails based on admin decision. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| PropertyRegistrationUI | «user interaction» | Boundary (I/O) | HostelOwner | 5-step wizard interface |
| CloudStorageProxy | «proxy» | Boundary (Output) | CloudStorage | Document uploads |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Confirmation and verification emails |
| PropertyRegistrationControl | «state-dependent control» | Control | N/A | Wizard workflow and admin verification orchestration |
| Hostel | «entity» | Entity | Hostel | Property record (status = Pending Verification) |
| Location | «entity» | Entity | Location | Property location |
| Amenity | «entity» | Entity | Amenity | Selected amenities |
| User | «entity» | Entity | User | Owner reference |
| Notification | «entity» | Entity | Notification | Confirmation emails |
| BusinessInfoValidator | «business logic» | Application Logic | N/A | Business information validation |
| PropertyDetailsValidator | «business logic» | Application Logic | N/A | Property details validation |
| DocumentValidator | «business logic» | Application Logic | N/A | Document validation |
| AddressVerificationService | «service» | Application Logic | N/A | Address validation and duplicate detection |
| RegistrationWorkflowManager | «service» | Application Logic | N/A | Admin verification workflow |

**Total**: 14 objects (3 boundary, 5 entity, 1 control, 5 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actors**: CloudStorage (documents), EmailService (notifications)
- **Boundary**: PropertyRegistrationUI, CloudStorageProxy, EmailServiceProxy
- **Control**: PropertyRegistrationControl
- **Entity**: Hostel, Location, Amenity, User, Notification
- **Logic**: BusinessInfoValidator, PropertyDetailsValidator, DocumentValidator, AddressVerificationService, RegistrationWorkflowManager

**Key Interactions (Main Flow)**:
1. HostelOwner → PropertyRegistrationUI: Click "Add Property"
2. PropertyRegistrationUI → PropertyRegistrationControl: Start wizard
3. PropertyRegistrationControl → PropertyRegistrationUI: Display Step 1 (Business Info)
4. HostelOwner → PropertyRegistrationUI: Enter business info, upload license
5. PropertyRegistrationUI → CloudStorageProxy: Upload license document
6. CloudStorageProxy → CloudStorage: Store document
7. HostelOwner → PropertyRegistrationUI: Click "Next"
8. PropertyRegistrationUI → PropertyRegistrationControl: Validate Step 1
9. PropertyRegistrationControl → BusinessInfoValidator: Validate business info
10. PropertyRegistrationControl → DocumentValidator: Validate license doc
11. PropertyRegistrationControl → PropertyRegistrationUI: Display Step 2 (Property Details)
12. HostelOwner → PropertyRegistrationUI: Enter property details
13. HostelOwner → PropertyRegistrationUI: Click "Next"
14. PropertyRegistrationControl → PropertyDetailsValidator: Validate details
15. PropertyRegistrationControl → AddressVerificationService: Verify address
16. AddressVerificationService: Attempt geocoding (may flag for manual review)
17. PropertyRegistrationControl → PropertyRegistrationUI: Display Step 3 (Amenities)
18. [Steps 4-5 similar pattern]
19. HostelOwner → PropertyRegistrationUI: Submit registration
20. PropertyRegistrationControl: Validate all steps complete
21. PropertyRegistrationControl → Hostel: Create with status = "Pending Verification"
22. PropertyRegistrationControl → Location: Create location record
23. PropertyRegistrationControl → Amenity: Link selected amenities
24. PropertyRegistrationControl → Notification: Create confirmation email
25. PropertyRegistrationControl → EmailServiceProxy: Send owner confirmation
26. PropertyRegistrationControl → RegistrationWorkflowManager: Queue for admin
27. RegistrationWorkflowManager → Notification: Create admin notification
28. RegistrationWorkflowManager → EmailServiceProxy: Send admin alert
29. PropertyRegistrationControl → PropertyRegistrationUI: Display confirmation with next steps

**Admin Verification Flow** (Post-submission):
1. Admin reviews within 2-3 business days
2. RegistrationWorkflowManager tracks status
3. **If approved**: Update Hostel.status = "Active", send approval email
4. **If additional info needed**: Send request email, owner updates, resubmit
5. **If rejected**: Update status = "Rejected", send rejection email with reason

---

## Phase 4 Requirements

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

## Validation

- [x] At least 1 boundary object (3 identified)
- [x] Boundary → External mapping (1:1):
  - PropertyRegistrationUI ← HostelOwner
  - CloudStorageProxy ← CloudStorage
  - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
  - Hostel ✅
  - Location ✅
  - Amenity ✅
  - User ✅
  - Notification ✅
- [x] Control justified (state-dependent 5-step wizard with admin verification workflow)
- [x] State-dependent flagged (PropertyRegistrationControl)
- [x] Logic justified (multi-step validation, address verification, admin workflow)

---

## Alternative Flows Mapping

**Step 5 Alternative** (Business info invalid):
- BusinessInfoValidator or DocumentValidator detects errors
- PropertyRegistrationControl → Step1_ValidatingBusiness (failed)
- PropertyRegistrationUI displays errors, prevents "Next"

**Step 8 Alternative** (Address cannot be validated):
- AddressVerificationService cannot geocode
- PropertyRegistrationControl flags for manual admin review
- Allows continuation (owner can complete wizard)
- Admin will verify address during review

**Step 17 Alternative** (Required info missing):
- PropertyRegistrationControl checks all steps complete
- PropertyRegistrationControl highlights incomplete sections
- Returns to specific incomplete step

**Admin Verification Alternatives** (After step 19):
- **Approved (2-3 days)**:
  - RegistrationWorkflowManager → Approved state
  - Hostel.status = "Active"
  - EmailServiceProxy sends approval email
  - Owner can manage property (UC-10, UC-11)
- **Additional Info Needed**:
  - RegistrationWorkflowManager → AdditionalInfoNeeded state
  - EmailServiceProxy sends request with requirements
  - Owner updates registration
  - PropertyRegistrationControl resubmits
- **Rejected**:
  - RegistrationWorkflowManager → Rejected state
  - EmailServiceProxy sends rejection with reason
  - Owner can appeal or create new registration

**Document Upload Alternatives** (Steps 3.4, 13.4):
- Wrong format: DocumentValidator error, retry
- Too large (>10MB): DocumentValidator error, resize/reupload
- Upload fails: CloudStorageProxy error, retry up to 3 times

**Wizard Navigation**:
- Click "Back": PropertyRegistrationControl saves current step, returns to previous
- Data preserved for all steps (auto-save)

**Save as Draft**:
- Owner clicks "Save and Continue Later"
- PropertyRegistrationControl → Draft state
- Hostel created with status = "Draft"
- Owner can resume from last step

---

## Business Rules

- **BR-048**: Admin verification required before live (RegistrationWorkflowManager enforces)
- **BR-049**: Business license and tax docs mandatory (BusinessInfoValidator, DocumentValidator)
- **BR-050**: Bank account verification required (DocumentValidator)
- **BR-051**: Commission structure in terms (wizard step 15: T&C acceptance)
- **BR-052**: Can manage property during verification (Hostel.status = "Pending Verification" allows setup)
- **BR-053**: Multiple properties allowed (separate registrations per property)

---

## Notes

- **Multi-step wizard**: Reduces abandonment vs single long form
- **State-dependent**: 5 steps + admin verification workflow requires state management
- **Draft save**: Auto-save prevents data loss, allows resume
- **Admin verification**: 2-3 business day SLA (72 hours)
- **Property setup during verification**: Keeps owners engaged (BR-052)
- **Document security**: Business license, tax ID, bank docs encrypted
- **Address validation**: Geocoding attempted, manual review if fails
- **Duplicate detection**: Prevents same property registered twice
- **Compliance**: Business registration, tax docs required per local regulations
- **Reusability**: CloudStorageProxy, EmailServiceProxy reused across UCs
