# Phase 2 Object Summaries - All Use Cases

**Generated**: 2026-02-04
**Use Cases Analyzed**: 24

---

## UC-01-REGISTER-ACCOUNT: Register Account

| Object                   | Stereotype                | Category          | Phase 1 Link | Purpose                                                  |
| ------------------------ | ------------------------- | ----------------- | ------------ | -------------------------------------------------------- |
| TravelerRegistrationUI   | «user interaction»        | Boundary (I/O)    | Traveler     | Registration form display and user interaction           |
| EmailServiceProxy        | «proxy»                   | Boundary (Output) | EmailService | Sends verification and welcome emails                    |
| RegistrationControl      | «state-dependent control» | Control           | N/A          | Orchestrates registration workflow and state transitions |
| User                     | «entity»                  | Entity            | User         | Account data storage                                     |
| Notification             | «entity»                  | Entity            | Notification | Email delivery tracking                                  |
| RegistrationValidator    | «business logic»          | Application Logic | N/A          | Registration data validation                             |
| EmailVerificationService | «service»                 | Application Logic | N/A          | Verification token management                            |

**Total**: 7 objects (2 boundary, 2 entity, 1 control, 2 logic)

---

## UC-02-LOG-IN: Log In

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

## UC-03-SEARCH-HOSTELS: Search Hostels

| Object                 | Stereotype         | Category          | Phase 1 Link | Purpose                                   |
| ---------------------- | ------------------ | ----------------- | ------------ | ----------------------------------------- |
| SearchUI               | «user interaction» | Boundary (I/O)    | Traveler     | Search interface with results display     |
| MapsAPIProxy           | «proxy»            | Boundary (Output) | MapsAPI      | Geocoding and map rendering               |
| SearchCoordinator      | «coordinator»      | Control           | N/A          | Search workflow orchestration (stateless) |
| Hostel                 | «entity»           | Entity            | Hostel       | Search target data                        |
| RoomType               | «entity»           | Entity            | RoomType     | Availability and pricing                  |
| Location               | «entity»           | Entity            | Location     | Geographic data                           |
| Photo                  | «entity»           | Entity            | Photo        | Visual display                            |
| Amenity                | «entity»           | Entity            | Amenity      | Filter criteria                           |
| Booking                | «entity»           | Entity            | Booking      | Availability conflicts                    |
| SearchEngine           | «algorithm»        | Application Logic | N/A          | Search and ranking algorithm              |
| AvailabilityCalculator | «business logic»   | Application Logic | N/A          | Real-time availability check              |
| PriceCalculator        | «business logic»   | Application Logic | N/A          | Stay cost calculation                     |

**Total**: 13 objects (2 boundary, 6 entity, 1 control, 4 logic)

---

## UC-04-VIEW-HOSTEL-DETAILS: View Hostel Details

| Object             | Stereotype         | Category          | Phase 1 Link | Purpose                                     |
| ------------------ | ------------------ | ----------------- | ------------ | ------------------------------------------- |
| HostelDetailsUI    | «user interaction» | Boundary (I/O)    | Traveler     | Details page with gallery and map           |
| CloudStorageProxy  | «proxy»            | Boundary (Output) | CloudStorage | Photo retrieval                             |
| MapsAPIProxy       | «proxy»            | Boundary (Output) | MapsAPI      | Interactive map rendering                   |
| DetailsCoordinator | «coordinator»      | Control           | N/A          | Content retrieval orchestration (stateless) |
| Hostel             | «entity»           | Entity            | Hostel       | Property details                            |
| Photo              | «entity»           | Entity            | Photo        | Gallery images                              |
| Location           | «entity»           | Entity            | Location     | Address and coordinates                     |
| Amenity            | «entity»           | Entity            | Amenity      | Facilities list                             |
| RoomType           | «entity»           | Entity            | RoomType     | Room options and pricing                    |
| Review             | «entity»           | Entity            | Review       | Guest feedback                              |
| User               | «entity»           | Entity            | User         | Reviewer information                        |
| ReviewManager      | «service»          | Application Logic | N/A          | Review filtering and pagination             |
| PriceCalculator    | «business logic»   | Application Logic | N/A          | Price calculation (reused)                  |

**Total**: 13 objects (3 boundary, 7 entity, 1 control, 2 logic)

---

## UC-05-BOOK-ACCOMMODATION: Book Accommodation

| Object                    | Stereotype                | Category          | Phase 1 Link   | Purpose                                              |
| ------------------------- | ------------------------- | ----------------- | -------------- | ---------------------------------------------------- |
| BookingUI                 | «user interaction»        | Boundary (I/O)    | Traveler       | Booking form and confirmation interface              |
| PaymentGatewayProxy       | «proxy»                   | Boundary (Output) | PaymentGateway | Payment processing integration                       |
| EmailServiceProxy         | «proxy»                   | Boundary (Output) | EmailService   | Confirmation email delivery                          |
| BookingControl            | «state-dependent control» | Control           | N/A            | Booking workflow orchestration with state management |
| User                      | «entity»                  | Entity            | User           | Authentication and guest data                        |
| Hostel                    | «entity»                  | Entity            | Hostel         | Property reference                                   |
| RoomType                  | «entity»                  | Entity            | RoomType       | Room reference and availability                      |
| Bed                       | «entity»                  | Entity            | Bed            | Bed assignment (dorm rooms)                          |
| Booking                   | «entity»                  | Entity            | Booking        | Reservation record                                   |
| Payment                   | «entity»                  | Entity            | Payment        | Transaction record                                   |
| Notification              | «entity»                  | Entity            | Notification   | Email confirmations                                  |
| GuestInfoValidator        | «business logic»          | Application Logic | N/A            | Guest data validation                                |
| AvailabilityValidator     | «business logic»          | Application Logic | N/A            | Real-time availability check                         |
| PriceCalculator           | «business logic»          | Application Logic | N/A            | Price calculation with breakdown                     |
| BookingReferenceGenerator | «service»                 | Application Logic | N/A            | Unique confirmation code generation                  |
| PaymentCoordinator        | «service»                 | Application Logic | N/A            | Payment gateway coordination                         |
| BedAssignmentService      | «service»                 | Application Logic | N/A            | Bed assignment for dorm bookings                     |

**Total**: 17 objects (3 boundary, 7 entity, 1 control, 6 logic)

---

## UC-06-MANAGE-BOOKINGS: Manage Bookings

| Object                       | Stereotype         | Category          | Phase 1 Link | Purpose                                     |
| ---------------------------- | ------------------ | ----------------- | ------------ | ------------------------------------------- |
| BookingManagementUI          | «user interaction» | Boundary (I/O)    | Traveler     | Bookings list and details interface         |
| PDFGenerator                 | «output»           | Boundary (Output) | N/A          | PDF confirmation generation                 |
| BookingManagementCoordinator | «coordinator»      | Control           | N/A          | Booking retrieval orchestration (stateless) |
| Booking                      | «entity»           | Entity            | Booking      | Reservation records                         |
| Hostel                       | «entity»           | Entity            | Hostel       | Property information                        |
| RoomType                     | «entity»           | Entity            | RoomType     | Room details                                |
| Photo                        | «entity»           | Entity            | Photo        | Hostel thumbnails                           |
| User                         | «entity»           | Entity            | User         | Traveler/guest data                         |
| Review                       | «entity»           | Entity            | Review       | Review existence check                      |
| BookingOrganizer             | «business logic»   | Application Logic | N/A          | Status-based organization and sorting       |
| ActionAvailabilityChecker    | «business logic»   | Application Logic | N/A          | Action availability rules                   |
| BookingPDFService            | «service»          | Application Logic | N/A          | PDF generation                              |

**Total**: 12 objects (2 boundary, 6 entity, 1 control, 3 logic)

---

## UC-07-SUBMIT-REVIEW: Submit Review

| Object                   | Stereotype                | Category          | Phase 1 Link | Purpose                                              |
| ------------------------ | ------------------------- | ----------------- | ------------ | ---------------------------------------------------- |
| ReviewSubmissionUI       | «user interaction»        | Boundary (I/O)    | Traveler     | Review form and submission interface                 |
| CloudStorageProxy        | «proxy»                   | Boundary (Output) | CloudStorage | Review photo uploads                                 |
| ReviewSubmissionControl  | «state-dependent control» | Control           | N/A          | Review workflow orchestration with moderation states |
| Review                   | «entity»                  | Entity            | Review       | Review record                                        |
| Booking                  | «entity»                  | Entity            | Booking      | Eligibility validation                               |
| Hostel                   | «entity»                  | Entity            | Hostel       | Review target, ratings aggregate                     |
| RoomType                 | «entity»                  | Entity            | RoomType     | Context display                                      |
| User                     | «entity»                  | Entity            | User         | Reviewer identity                                    |
| Photo                    | «entity»                  | Entity            | Photo        | Review photos                                        |
| ReviewEligibilityChecker | «business logic»          | Application Logic | N/A          | Eligibility validation                               |
| ReviewContentValidator   | «business logic»          | Application Logic | N/A          | Content validation                                   |
| ReviewPhotoValidator     | «business logic»          | Application Logic | N/A          | Photo validation                                     |
| RatingAggregator         | «algorithm»               | Application Logic | N/A          | Aggregate ratings calculation                        |
| ModerationService        | «service»                 | Application Logic | N/A          | Moderation queue management                          |

**Total**: 14 objects (2 boundary, 6 entity, 1 control, 5 logic)

---

## UC-08-MANAGE-PROFILE: Manage Profile

| Object                | Stereotype                | Category          | Phase 1 Link | Purpose                                                      |
| --------------------- | ------------------------- | ----------------- | ------------ | ------------------------------------------------------------ |
| ProfileManagementUI   | «user interaction»        | Boundary (I/O)    | Traveler     | Profile sections and editing interface                       |
| CloudStorageProxy     | «proxy»                   | Boundary (Output) | CloudStorage | Profile photo uploads                                        |
| EmailServiceProxy     | «proxy»                   | Boundary (Output) | EmailService | Verification and security emails                             |
| ProfileUpdateControl  | «state-dependent control» | Control           | N/A          | Profile update orchestration with email/password state flows |
| User                  | «entity»                  | Entity            | User         | Account data (email, password, fullName)                     |
| UserProfile           | «entity»                  | Entity            | UserProfile  | Extended profile data                                        |
| Notification          | «entity»                  | Entity            | Notification | Email notifications                                          |
| Photo                 | «entity»                  | Entity            | Photo        | Profile photo                                                |
| ProfileInfoValidator  | «business logic»          | Application Logic | N/A          | Profile data validation                                      |
| EmailChangeService    | «service»                 | Application Logic | N/A          | Email change workflow                                        |
| PasswordChangeService | «service»                 | Application Logic | N/A          | Password change workflow                                     |
| ProfilePhotoService   | «service»                 | Application Logic | N/A          | Photo upload management                                      |

**Total**: 12 objects (3 boundary, 4 entity, 1 control, 4 logic)

---

## UC-09-REGISTER-HOSTEL: Register Hostel

| Object                      | Stereotype                | Category          | Phase 1 Link | Purpose                                              |
| --------------------------- | ------------------------- | ----------------- | ------------ | ---------------------------------------------------- |
| PropertyRegistrationUI      | «user interaction»        | Boundary (I/O)    | HostelOwner  | 5-step wizard interface                              |
| CloudStorageProxy           | «proxy»                   | Boundary (Output) | CloudStorage | Document uploads                                     |
| EmailServiceProxy           | «proxy»                   | Boundary (Output) | EmailService | Confirmation and verification emails                 |
| PropertyRegistrationControl | «state-dependent control» | Control           | N/A          | Wizard workflow and admin verification orchestration |
| Hostel                      | «entity»                  | Entity            | Hostel       | Property record (status = Pending Verification)      |
| Location                    | «entity»                  | Entity            | Location     | Property location                                    |
| Amenity                     | «entity»                  | Entity            | Amenity      | Selected amenities                                   |
| User                        | «entity»                  | Entity            | User         | Owner reference                                      |
| Notification                | «entity»                  | Entity            | Notification | Confirmation emails                                  |
| BusinessInfoValidator       | «business logic»          | Application Logic | N/A          | Business information validation                      |
| PropertyDetailsValidator    | «business logic»          | Application Logic | N/A          | Property details validation                          |
| DocumentValidator           | «business logic»          | Application Logic | N/A          | Document validation                                  |
| AddressVerificationService  | «service»                 | Application Logic | N/A          | Address validation and duplicate detection           |
| RegistrationWorkflowManager | «service»                 | Application Logic | N/A          | Admin verification workflow                          |

**Total**: 14 objects (3 boundary, 5 entity, 1 control, 5 logic)

---

## UC-10-MANAGE-PROPERTY: Manage Property

| Object                    | Stereotype         | Category          | Phase 1 Link | Purpose                                   |
| ------------------------- | ------------------ | ----------------- | ------------ | ----------------------------------------- |
| PropertyManagementUI      | «user interaction» | Boundary (I/O)    | HostelOwner  | Property editing interface                |
| CloudStorageProxy         | «proxy»            | Boundary (Output) | CloudStorage | Photo uploads and optimization            |
| MapsAPIProxy              | «proxy»            | Boundary (Output) | MapsAPI      | Geocoding and map rendering               |
| PropertyUpdateCoordinator | «coordinator»      | Control           | N/A          | Property update orchestration (stateless) |
| Hostel                    | «entity»           | Entity            | Hostel       | Property data                             |
| Photo                     | «entity»           | Entity            | Photo        | Property photos                           |
| Location                  | «entity»           | Entity            | Location     | Geographic location                       |
| Amenity                   | «entity»           | Entity            | Amenity      | Amenities list                            |
| PropertyInfoValidator     | «business logic»   | Application Logic | N/A          | Property data validation                  |
| PhotoManagementService    | «service»          | Application Logic | N/A          | Photo gallery management                  |
| GeocodingService          | «service»          | Application Logic | N/A          | Address geocoding                         |
| AutosaveService           | «service»          | Application Logic | N/A          | Draft autosave                            |

**Total**: 12 objects (3 boundary, 4 entity, 1 control, 4 logic)

---

## UC-11-MANAGE-ROOM-TYPES: Manage Room Types

| Object                 | Stereotype         | Category          | Phase 1 Link | Purpose                             |
| ---------------------- | ------------------ | ----------------- | ------------ | ----------------------------------- |
| RoomManagementUI       | «user interaction» | Boundary (I/O)    | HostelOwner  | Room types management interface     |
| RoomTypeCoordinator    | «coordinator»      | Control           | N/A          | Room CRUD orchestration (stateless) |
| RoomType               | «entity»           | Entity            | RoomType     | Room type configuration             |
| Bed                    | «entity»           | Entity            | Bed          | Individual bed inventory (dorms)    |
| Hostel                 | «entity»           | Entity            | Hostel       | Property reference                  |
| Amenity                | «entity»           | Entity            | Amenity      | Room-specific amenities             |
| RoomTypeValidator      | «business logic»   | Application Logic | N/A          | Room configuration validation       |
| CapacityCalculator     | «algorithm»        | Application Logic | N/A          | Guest capacity calculation          |
| InventoryGenerator     | «service»          | Application Logic | N/A          | Room/bed inventory generation       |
| PricingRuleEngine      | «service»          | Application Logic | N/A          | Pricing rules management            |
| BookingConflictChecker | «business logic»   | Application Logic | N/A          | Booking conflict detection          |

**Total**: 11 objects (1 boundary, 4 entity, 1 control, 5 logic)

---

## UC-12-PROCESS-BOOKING: Process Booking

| Object                       | Stereotype         | Category          | Phase 1 Link   | Purpose                                      |
| ---------------------------- | ------------------ | ----------------- | -------------- | -------------------------------------------- |
| BookingProcessingUI          | «user interaction» | Boundary (I/O)    | HostelOwner    | Booking processing interface                 |
| EmailServiceProxy            | «proxy»            | Boundary (Output) | EmailService   | Email notifications                          |
| SMSGatewayProxy              | «proxy»            | Boundary (Output) | SMSGateway     | Urgent SMS alerts                            |
| PaymentGatewayProxy          | «proxy»            | Boundary (Output) | PaymentGateway | Refund processing                            |
| BookingProcessingCoordinator | «coordinator»      | Control           | N/A            | Booking operations orchestration (stateless) |
| Booking                      | «entity»           | Entity            | Booking        | Booking records                              |
| User                         | «entity»           | Entity            | User           | Guest information                            |
| RoomType                     | «entity»           | Entity            | RoomType       | Room details                                 |
| Bed                          | «entity»           | Entity            | Bed            | Bed assignment                               |
| Payment                      | «entity»           | Entity            | Payment        | Payment info and refunds                     |
| Notification                 | «entity»           | Entity            | Notification   | Communications log                           |
| Hostel                       | «entity»           | Entity            | Hostel         | Property reference                           |
| BookingOrganizer             | «business logic»   | Application Logic | N/A            | Status organization and urgency sorting      |
| AvailabilityRevalidator      | «business logic»   | Application Logic | N/A            | Room availability recheck                    |
| BedAssignmentManager         | «service»          | Application Logic | N/A            | Dorm bed assignment                          |
| BookingNotificationService   | «service»          | Application Logic | N/A            | Email/SMS notifications                      |
| RefundProcessor              | «service»          | Application Logic | N/A            | Decline and refund processing                |

**Total**: 16 objects (4 boundary, 7 entity, 1 control, 4 logic)

---

## UC-13-MANAGE-STAFF: Manage Staff

| Object                   | Stereotype                | Category          | Phase 1 Link | Purpose                                      |
| ------------------------ | ------------------------- | ----------------- | ------------ | -------------------------------------------- |
| StaffManagementUI        | «user interaction»        | Boundary (I/O)    | HostelOwner  | Staff management interface                   |
| EmailServiceProxy        | «proxy»                   | Boundary (Output) | EmailService | Invitation and notification emails           |
| StaffManagementControl   | «state-dependent control» | Control           | N/A          | Staff invitation and lifecycle orchestration |
| Staff                    | «entity»                  | Entity            | Staff        | Staff assignments and permissions            |
| User                     | «entity»                  | Entity            | User         | Staff accounts                               |
| Notification             | «entity»                  | Entity            | Notification | Email communications                         |
| Hostel                   | «entity»                  | Entity            | Hostel       | Property reference                           |
| StaffInvitationValidator | «business logic»          | Application Logic | N/A          | Invitation validation                        |
| InvitationTokenManager   | «service»                 | Application Logic | N/A          | Token generation and validation              |
| PermissionManager        | «service»                 | Application Logic | N/A          | Role and permission configuration            |
| AccessControlManager     | «service»                 | Application Logic | N/A          | Access enforcement and revocation            |

**Total**: 11 objects (2 boundary, 4 entity, 1 control, 4 logic)

---

## UC-14-VIEW-ANALYTICS: View Analytics

| Object               | Stereotype         | Category          | Phase 1 Link | Purpose                                    |
| -------------------- | ------------------ | ----------------- | ------------ | ------------------------------------------ |
| AnalyticsDashboardUI | «user interaction» | Boundary (I/O)    | HostelOwner  | Analytics dashboard interface              |
| ReportExporter       | «output»           | Boundary (Output) | N/A          | PDF/Excel report generation                |
| AnalyticsCoordinator | «coordinator»      | Control           | N/A          | Analytics data orchestration (stateless)   |
| Booking              | «entity»           | Entity            | Booking      | Booking data (read-only)                   |
| Payment              | «entity»           | Entity            | Payment      | Revenue data (read-only)                   |
| Review               | «entity»           | Entity            | Review       | Review data (read-only)                    |
| RoomType             | «entity»           | Entity            | RoomType     | Room performance (read-only)               |
| Bed                  | «entity»           | Entity            | Bed          | Bed occupancy (read-only)                  |
| User                 | «entity»           | Entity            | User         | Guest demographics (anonymized, read-only) |
| Hostel               | «entity»           | Entity            | Hostel       | Property context                           |
| RevenueAnalyzer      | «algorithm»        | Application Logic | N/A          | Revenue metrics calculation                |
| OccupancyAnalyzer    | «algorithm»        | Application Logic | N/A          | Occupancy metrics calculation              |
| BookingAnalyzer      | «service»          | Application Logic | N/A          | Booking pattern analysis                   |
| GuestAnalyzer        | «service»          | Application Logic | N/A          | Guest demographics analysis                |
| ReviewAnalyzer       | «algorithm»        | Application Logic | N/A          | Review and sentiment analysis              |
| ChartGenerator       | «service»          | Application Logic | N/A          | Visualization generation                   |

**Total**: 16 objects (2 boundary, 7 entity, 1 control, 6 logic)

---

## UC-15-MANAGE-SUBSCRIPTION: Manage Subscription

| Object                   | Stereotype                | Category          | Phase 1 Link   | Purpose                              |
| ------------------------ | ------------------------- | ----------------- | -------------- | ------------------------------------ |
| SubscriptionManagementUI | «user interaction»        | Boundary (I/O)    | HostelOwner    | Subscription management interface    |
| PaymentGatewayProxy      | «proxy»                   | Boundary (Output) | PaymentGateway | Payment processing                   |
| EmailServiceProxy        | «proxy»                   | Boundary (Output) | EmailService   | Confirmation and notification emails |
| SubscriptionControl      | «state-dependent control» | Control           | N/A            | Subscription lifecycle orchestration |
| Subscription             | «entity»                  | Entity            | Subscription   | Subscription record                  |
| Payment                  | «entity»                  | Entity            | Payment        | Payment transactions and invoices    |
| User                     | «entity»                  | Entity            | User           | Owner account                        |
| Hostel                   | «entity»                  | Entity            | Hostel         | Property limits enforcement          |
| SubscriptionPlanManager  | «service»                 | Application Logic | N/A            | Plan definitions and comparison      |
| ProRatedCalculator       | «algorithm»               | Application Logic | N/A            | Pro-rated charge calculation         |
| FeatureGateManager       | «service»                 | Application Logic | N/A            | Feature access enforcement           |
| TrialManager             | «service»                 | Application Logic | N/A            | Trial period management              |
| PaymentRetryManager      | «service»                 | Application Logic | N/A            | Failed payment handling              |

**Total**: 13 objects (3 boundary, 4 entity, 1 control, 5 logic)

---

## UC-16-COMMUNICATE-WITH-GUEST: Communicate With Guest

| Object                   | Stereotype         | Category | Phase 1 Link |
| ------------------------ | ------------------ | -------- | ------------ |
| MessagingInteraction     | «user interaction» | Boundary | HostelOwner  |
| EmailServiceProxy        | «proxy»            | Boundary | EmailService |
| Message                  | «entity»           | Entity   | (New)        |
| MessageThread            | «entity»           | Entity   | (New)        |
| Booking                  | «entity»           | Entity   | Booking      |
| Guest                    | «entity»           | Entity   | User/Guest   |
| Property                 | «entity»           | Entity   | Property     |
| MessageTemplate          | «entity»           | Entity   | (New)        |
| Attachment               | «entity»           | Entity   | (New)        |
| MessagingCoordinator     | «coordinator»      | Control  | N/A          |
| MessageValidationService | «business logic»   | Logic    | N/A          |
| BulkMessagingService     | «service»          | Logic    | N/A          |

**Total**: 12 objects (2 boundary, 7 entity, 1 control, 2 logic)

---

## UC-17-CHECK-IN-GUEST: Check In Guest

| Object                | Stereotype         | Category | Phase 1 Link |
| --------------------- | ------------------ | -------- | ------------ |
| CheckInInteraction    | «user interaction» | Boundary | HostelStaff  |
| Booking               | «entity»           | Entity   | Booking      |
| Guest                 | «entity»           | Entity   | User/Guest   |
| Room                  | «entity»           | Entity   | Room         |
| Bed                   | «entity»           | Entity   | (New)        |
| Payment               | «entity»           | Entity   | Payment      |
| IDDocument            | «entity»           | Entity   | (New)        |
| EmergencyContact      | «entity»           | Entity   | (New)        |
| CheckInCoordinator    | «coordinator»      | Control  | N/A          |
| BedAssignmentService  | «service»          | Logic    | N/A          |
| IDVerificationService | «business logic»   | Logic    | N/A          |

**Total**: 11 objects (1 boundary, 7 entity, 1 control, 2 logic)

---

## UC-18-CHECK-OUT-GUEST: Check Out Guest

| Object                   | Stereotype         | Category | Phase 1 Link |
| ------------------------ | ------------------ | -------- | ------------ |
| CheckOutInteraction      | «user interaction» | Boundary | HostelStaff  |
| Booking                  | «entity»           | Entity   | Booking      |
| Guest                    | «entity»           | Entity   | User/Guest   |
| Room                     | «entity»           | Entity   | Room         |
| Bed                      | «entity»           | Entity   | Bed          |
| Payment                  | «entity»           | Entity   | Payment      |
| Charge                   | «entity»           | Entity   | (New)        |
| GuestFeedback            | «entity»           | Entity   | (New)        |
| LostProperty             | «entity»           | Entity   | (New)        |
| CheckOutCoordinator      | «coordinator»      | Control  | N/A          |
| ChargeCalculationService | «business logic»   | Logic    | N/A          |
| InventoryReleaseService  | «service»          | Logic    | N/A          |

**Total**: 12 objects (1 boundary, 8 entity, 1 control, 2 logic)

---

## UC-19-VIEW-BOOKING-DETAILS: View Booking Details

| Object                    | Stereotype         | Category | Phase 1 Link  |
| ------------------------- | ------------------ | -------- | ------------- |
| BookingDetailsInteraction | «user interaction» | Boundary | HostelStaff   |
| Booking                   | «entity»           | Entity   | Booking       |
| Guest                     | «entity»           | Entity   | User/Guest    |
| Room                      | «entity»           | Entity   | Room          |
| Bed                       | «entity»           | Entity   | Bed           |
| Payment                   | «entity»           | Entity   | Payment       |
| Message                   | «entity»           | Entity   | Message       |
| MessageThread             | «entity»           | Entity   | MessageThread |
| StaffNote                 | «entity»           | Entity   | (New)         |
| ActivityLog               | «entity»           | Entity   | (New)         |

**Total**: 10 objects (1 boundary, 9 entity, 0 control, 0 logic)

---

## UC-20-MANAGE-USER-ACCOUNTS: Manage User Accounts

| Object                      | Stereotype         | Category | Phase 1 Link          |
| --------------------------- | ------------------ | -------- | --------------------- |
| AdminInteraction            | «user interaction» | Boundary | PlatformAdministrator |
| EmailServiceProxy           | «proxy»            | Boundary | EmailService          |
| User                        | «entity»           | Entity   | User                  |
| Property                    | «entity»           | Entity   | Property              |
| Booking                     | «entity»           | Entity   | Booking               |
| AdminAuditLog               | «entity»           | Entity   | (New)                 |
| LoginActivity               | «entity»           | Entity   | (New)                 |
| UserManagementCoordinator   | «coordinator»      | Control  | N/A                   |
| AccountAnonymizationService | «service»          | Logic    | N/A                   |
| PasswordResetService        | «business logic»   | Logic    | N/A                   |

**Total**: 10 objects (2 boundary, 5 entity, 1 control, 2 logic)

---

## UC-21-MANAGE-PLATFORM-SUBSCRIPTIONS: Manage Platform Subscriptions

| Object                       | Stereotype         | Category | Phase 1 Link          |
| ---------------------------- | ------------------ | -------- | --------------------- |
| AdminInteraction             | «user interaction» | Boundary | PlatformAdministrator |
| Subscription                 | «entity»           | Entity   | Subscription          |
| Property                     | «entity»           | Entity   | Property              |
| Payment                      | «entity»           | Entity   | Payment               |
| SubscriptionPlan             | «entity»           | Entity   | SubscriptionPlan      |
| Refund                       | «entity»           | Entity   | (New)                 |
| AdminAuditLog                | «entity»           | Entity   | AdminAuditLog         |
| SubscriptionAdminCoordinator | «coordinator»      | Control  | N/A                   |
| RefundCalculationService     | «business logic»   | Logic    | N/A                   |
| FeatureAccessService         | «service»          | Logic    | N/A                   |

**Total**: 10 objects (1 boundary, 6 entity, 1 control, 2 logic)

---

## UC-22-CONFIGURE-SYSTEM-SETTINGS: Configure System Settings

| Object                          | Stereotype         | Category | Phase 1 Link          |
| ------------------------------- | ------------------ | -------- | --------------------- |
| AdminInteraction                | «user interaction» | Boundary | PlatformAdministrator |
| SystemSetting                   | «entity»           | Entity   | (New)                 |
| EmailTemplate                   | «entity»           | Entity   | EmailTemplate         |
| FeatureFlag                     | «entity»           | Entity   | (New)                 |
| PaymentGatewayConfig            | «entity»           | Entity   | (New)                 |
| MaintenanceWindow               | «entity»           | Entity   | (New)                 |
| ConfigurationCoordinator        | «coordinator»      | Control  | N/A                   |
| SettingValidationService        | «business logic»   | Logic    | N/A                   |
| ConfigurationPropagationService | «service»          | Logic    | N/A                   |

**Total**: 9 objects (1 boundary, 5 entity, 1 control, 2 logic)

---

## UC-23-MONITOR-PLATFORM-METRICS: Monitor Platform Metrics

| Object                    | Stereotype         | Category | Phase 1 Link          |
| ------------------------- | ------------------ | -------- | --------------------- |
| AdminInteraction          | «user interaction» | Boundary | PlatformAdministrator |
| User                      | «entity»           | Entity   | User                  |
| Property                  | «entity»           | Entity   | Property              |
| Booking                   | «entity»           | Entity   | Booking               |
| Subscription              | «entity»           | Entity   | Subscription          |
| Payment                   | «entity»           | Entity   | Payment               |
| PlatformMetric            | «entity»           | Entity   | (New)                 |
| SystemHealthMetric        | «entity»           | Entity   | (New)                 |
| MetricsCoordinator        | «coordinator»      | Control  | N/A                   |
| MetricsAggregationService | «service»          | Logic    | N/A                   |
| ReportGenerationService   | «service»          | Logic    | N/A                   |

**Total**: 11 objects (1 boundary, 7 entity, 1 control, 2 logic)

---

## UC-24-CANCEL-BOOKING: Cancel Booking

| Object                   | Stereotype                | Category | Phase 1 Link   |
| ------------------------ | ------------------------- | -------- | -------------- |
| CancellationInteraction  | «user interaction»        | Boundary | Traveler       |
| PaymentGatewayProxy      | «proxy»                   | Boundary | PaymentGateway |
| EmailServiceProxy        | «proxy»                   | Boundary | EmailService   |
| Booking                  | «entity»                  | Entity   | Booking        |
| Payment                  | «entity»                  | Entity   | Payment        |
| Refund                   | «entity»                  | Entity   | Refund         |
| CancellationPolicy       | «entity»                  | Entity   | (New)          |
| Room                     | «entity»                  | Entity   | Room           |
| Bed                      | «entity»                  | Entity   | Bed            |
| CancellationRecord       | «entity»                  | Entity   | (New)          |
| CancellationControl      | «state-dependent control» | Control  | N/A            |
| RefundCalculationService | «business logic»          | Logic    | N/A            |
| InventoryReleaseService  | «service»                 | Logic    | N/A            |

**Total**: 13 objects (3 boundary, 7 entity, 1 control, 2 logic)

---
