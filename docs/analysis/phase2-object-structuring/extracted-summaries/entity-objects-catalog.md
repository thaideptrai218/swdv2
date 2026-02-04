# Entity Objects Catalog - HostelViet

**Source**: Phase 1 Static Model - `phase1-static-model.md`
**Purpose**: Reference catalog of all entity objects for COMET analysis phases
**Total Entities**: 14

---

## Quick Reference Table

| Entity ID | Entity Name  | Primary Key    | Core Domain     | Related Entities                                                        |
| --------- | ------------ | -------------- | --------------- | ----------------------------------------------------------------------- |
| E01       | User         | userId         | User Management | UserProfile, Hostel, Booking, Review, Subscription, Staff, Notification |
| E02       | UserProfile  | profileId      | User Management | User                                                                    |
| E03       | Hostel       | hostelId       | Property        | User, RoomType, Amenity, Photo, Booking, Review, Location, Staff        |
| E04       | RoomType     | roomTypeId     | Property        | Hostel, Bed, Booking                                                    |
| E05       | Bed          | bedId          | Property        | RoomType, Booking                                                       |
| E06       | Booking      | bookingId      | Booking         | User, Hostel, RoomType, Bed, Payment, Review                            |
| E07       | Payment      | paymentId      | Financial       | Booking, Subscription                                                   |
| E08       | Review       | reviewId       | Review          | User, Hostel, Booking                                                   |
| E09       | Amenity      | amenityId      | Property        | Hostel                                                                  |
| E10       | Photo        | photoId        | Property        | Hostel                                                                  |
| E11       | Subscription | subscriptionId | Financial       | User, Payment                                                           |
| E12       | Location     | locationId     | Property        | Hostel                                                                  |
| E13       | Staff        | staffId        | User Management | User, Hostel                                                            |
| E14       | Notification | notificationId | Communication   | User                                                                    |

---

## Entity Definitions

### E01: User

**Purpose**: Core entity representing all system users (Travelers, Owners, Staff, Administrators)

**Attributes**:

- `userId`: String - Unique identifier
- `email`: String - User email address for authentication
- `password`: String - Encrypted password
- `fullName`: String - User's full name
- `role`: String - User role (Traveler/Owner/Staff/Admin)
- `accountStatus`: String - Account state (Active/Unverified/Suspended/Deleted)
- `createdDate`: DateTime - Account creation timestamp
- `lastLoginDate`: DateTime - Last successful login

**Relationships**:

- Has UserProfile (1 → 0..1)
- Owns Hostel (1 → 0..\*)
- Makes Booking (1 → 0..\*) [Traveler role]
- Writes Review (1 → 0..\*) [Traveler role]
- Has Subscription (1 → 0..1) [Owner role]
- Is Staff (1 → 0..\*) [Staff role]
- Receives Notification (1 → 0..\*)

**Business Rules**:

- One user can have multiple roles
- Email must be unique
- Account status lifecycle: Unverified → Active → Suspended/Deleted
- Password must meet security requirements (8+ chars, mixed case, numbers, special chars)

---

### E02: UserProfile

**Purpose**: Extended user information and preferences

**Attributes**:

- `profileId`: String - Unique identifier
- `phoneNumber`: String - Contact phone number
- `nationality`: String - User's nationality
- `dateOfBirth`: Date - Date of birth
- `language`: String - Preferred language (Vietnamese/English)
- `preferences`: String - User preferences JSON

**Relationships**:

- Belongs to User (1 → 1)

**Business Rules**:

- Optional profile (can exist without profile)
- Minimum age requirement: 18 years
- Language defaults to Vietnamese

---

### E03: Hostel

**Purpose**: Hostel property managed by owners

**Attributes**:

- `hostelId`: String - Unique identifier
- `name`: String - Hostel name
- `description`: String - Detailed description
- `address`: String - Street address
- `city`: String - City location
- `coordinates`: String - GPS coordinates
- `rating`: Decimal - Average rating 0-5
- `status`: String - Property status (Active/Inactive/Pending)
- `ownerId`: String - Reference to owner User
- `createdDate`: DateTime - Property creation date

**Relationships**:

- Owned by User (1 → 1)
- Has RoomType (1 → 1..\*)
- Provides Amenity (1 → 0..\*)
- Displays Photo (1 → 0..\*)
- Receives Booking (1 → 0..\*)
- Receives Review (1 → 0..\*)
- Located at Location (1 → 1)
- Employs Staff (1 → 0..\*)

**Business Rules**:

- Must have at least 1 room type
- Must have location
- Rating calculated from reviews
- Maximum 20 photos per hostel
- Owner tier limits: Basic (10 properties), Pro (50), Enterprise (unlimited)

---

### E04: RoomType

**Purpose**: Type of accommodation within a hostel (dorm, private room)

**Attributes**:

- `roomTypeId`: String - Unique identifier
- `name`: String - Room type name
- `description`: String - Room description
- `capacity`: Integer - Maximum guests
- `bedConfiguration`: String - Bed setup description
- `pricePerNight`: Decimal - Nightly rate (VND)
- `totalRooms`: Integer - Total rooms of this type
- `availableRooms`: Integer - Currently available

**Relationships**:

- Belongs to Hostel (1 → 1)
- Contains Bed (1 → 1..\*) [For dorm types]
- Included in Booking (1 → 0..\*)

**Business Rules**:

- Dorm rooms must have individual beds
- Private rooms don't require bed tracking
- availableRooms ≤ totalRooms
- Price per night in VND

---

### E05: Bed

**Purpose**: Individual bed in dorm rooms

**Attributes**:

- `bedId`: String - Unique identifier
- `bedNumber`: String - Bed number/label
- `bedType`: String - Type (Bunk/Single/Double)
- `status`: String - Availability status (Available/Occupied/Maintenance)

**Relationships**:

- Belongs to RoomType (1 → 1)
- Assigned in Booking (1 → 0..\*)

**Business Rules**:

- Only for dorm room types
- Bed number unique within room type
- Status updated on check-in/check-out

---

### E06: Booking

**Purpose**: Guest reservation for accommodation

**Attributes**:

- `bookingId`: String - Unique identifier
- `checkInDate`: Date - Check-in date
- `checkOutDate`: Date - Check-out date
- `numberOfGuests`: Integer - Number of guests
- `totalPrice`: Decimal - Total booking price (VND)
- `status`: String - Booking status (Pending/Confirmed/CheckedIn/CheckedOut/Cancelled)
- `bookingDate`: DateTime - Timestamp when booking created
- `confirmationCode`: String - Unique confirmation code

**Relationships**:

- Made by User (1 → 1) [Traveler]
- For Hostel (1 → 1)
- Includes RoomType (1 → 1..\*)
- Assigns Bed (1 → 0..\*) [For dorm bookings]
- Has Payment (1 → 1)
- May have Review (1 → 0..1)

**Relationships**:

- Status lifecycle: Pending → Confirmed → CheckedIn → CheckedOut
- Can be Cancelled at Pending/Confirmed stages
- Check-out date must be after check-in date
- 24-hour cancellation window for refunds
- numberOfGuests ≤ room capacity

---

### E07: Payment

**Purpose**: Financial transaction for booking or subscription

**Attributes**:

- `paymentId`: String - Unique identifier
- `amount`: Decimal - Payment amount
- `currency`: String - Currency code (VND)
- `paymentMethod`: String - Payment method (VietQR/VNPay)
- `transactionId`: String - External transaction ID
- `paymentDate`: DateTime - Payment timestamp
- `status`: String - Payment status (Pending/Completed/Failed/Refunded)

**Relationships**:

- For Booking (1 → 1) OR For Subscription (1 → 1)
- Processed via PaymentGateway (external system)

**Business Rules**:

- One payment per booking
- Multiple payments per subscription (recurring)
- Payment method: VietQR or VNPay only
- Currency: VND only
- Transaction ID from payment gateway

---

### E08: Review

**Purpose**: Guest review and rating of hostel after stay

**Attributes**:

- `reviewId`: String - Unique identifier
- `rating`: Integer - Rating 1-5 stars
- `comment`: String - Review text
- `reviewDate`: DateTime - Review submission date
- `status`: String - Review status (Pending/Published/Rejected)

**Relationships**:

- Written by User (1 → 1) [Traveler]
- For Hostel (1 → 1)
- About Booking (1 → 1)

**Business Rules**:

- Only after check-out
- One review per booking
- Rating 1-5 stars (integer)
- Moderation required (Pending → Published/Rejected)
- Affects hostel average rating

---

### E09: Amenity

**Purpose**: Hostel facilities and services

**Attributes**:

- `amenityId`: String - Unique identifier
- `name`: String - Amenity name (WiFi, Breakfast, Parking)
- `description`: String - Amenity description
- `category`: String - Category (Basic/Premium/Special)

**Relationships**:

- Available at Hostel (many-to-many)

**Business Rules**:

- Reusable across hostels
- Categorized for filtering
- Standard amenity list (WiFi, Breakfast, Parking, AC, etc.)

---

### E10: Photo

**Purpose**: Hostel property images

**Attributes**:

- `photoId`: String - Unique identifier
- `url`: String - Cloud storage URL
- `caption`: String - Photo description
- `uploadDate`: DateTime - Upload timestamp
- `displayOrder`: Integer - Display sequence

**Relationships**:

- For Hostel (1 → 1)

**Business Rules**:

- Maximum 20 photos per hostel
- WebP format
- Maximum 5MB per photo
- Stored in CloudStorage (S3/Cloudinary)
- Display order for sorting

---

### E11: Subscription

**Purpose**: Owner subscription plan for platform access

**Attributes**:

- `subscriptionId`: String - Unique identifier
- `planType`: String - Plan tier (Basic/Pro/Enterprise)
- `startDate`: Date - Subscription start date
- `endDate`: Date - Subscription end date
- `status`: String - Subscription status (Active/Expired/Cancelled)
- `billingCycle`: String - Billing frequency (Monthly/Yearly)
- `price`: Decimal - Subscription price (USD)

**Relationships**:

- For User (1 → 1) [Owner]
- Has Payment (1 → 0..\*) [Recurring payments]

**Business Rules**:

- Plan tiers: Basic ($29/mo), Pro ($79/mo), Enterprise ($199/mo)
- Billing cycle: Monthly or Yearly
- 7-day grace period for failed payments
- Status lifecycle: Active → Expired/Cancelled
- Property limits by tier

---

### E12: Location

**Purpose**: Geographic location data for hostels

**Attributes**:

- `locationId`: String - Unique identifier
- `latitude`: Decimal - GPS latitude
- `longitude`: Decimal - GPS longitude
- `address`: String - Full address
- `city`: String - City name
- `district`: String - District/area
- `postalCode`: String - Postal code

**Relationships**:

- For Hostel (1 → 1)

**Business Rules**:

- One location per hostel
- Coordinates from Maps API
- Used for distance calculations
- Search by city/district

---

### E13: Staff

**Purpose**: Staff assignment and permissions

**Attributes**:

- `staffId`: String - Unique identifier
- `userId`: String - Reference to User
- `hostelId`: String - Reference to Hostel
- `role`: String - Staff role (Receptionist/Manager)
- `permissions`: String - Permission level
- `assignedDate`: DateTime - Assignment date

**Relationships**:

- Is User (1 → 1)
- Works at Hostel (1 → 1)

**Business Rules**:

- One user can be staff at multiple hostels
- Role-based permissions (Receptionist/Manager)
- Owner automatically has full permissions
- Staff can view/modify bookings for assigned hostel only

---

### E14: Notification

**Purpose**: System notifications and communications

**Attributes**:

- `notificationId`: String - Unique identifier
- `recipientId`: String - Target user ID
- `type`: String - Notification type (Email/SMS/InApp)
- `subject`: String - Notification subject
- `content`: String - Notification body
- `sentDate`: DateTime - Send timestamp
- `deliveryStatus`: String - Delivery status (Sent/Delivered/Failed)

**Relationships**:

- Sent to User (1 → 1)
- Delivered via EmailService or SMSGateway (external systems)

**Business Rules**:

- Email for: Verification, booking confirmation, password reset
- SMS for: Urgent booking reminders
- InApp for: General updates
- Delivery within 30 seconds
- Email deliverability target: 95%+
- Retry logic for failed deliveries

---

## Entity Grouping by Domain

### User Management Domain

- E01: User
- E02: UserProfile
- E13: Staff

### Property Domain

- E03: Hostel
- E04: RoomType
- E05: Bed
- E09: Amenity
- E10: Photo
- E12: Location

### Booking Domain

- E06: Booking
- E08: Review

### Financial Domain

- E07: Payment
- E11: Subscription

### Communication Domain

- E14: Notification

---

## Common Attribute Patterns

### Identifiers

All entities use String type for primary keys (UUIDs)

### Status Fields

Common status enumerations:

- **User.accountStatus**: Active, Unverified, Suspended, Deleted
- **Hostel.status**: Active, Inactive, Pending
- **Bed.status**: Available, Occupied, Maintenance
- **Booking.status**: Pending, Confirmed, CheckedIn, CheckedOut, Cancelled
- **Payment.status**: Pending, Completed, Failed, Refunded
- **Review.status**: Pending, Published, Rejected
- **Subscription.status**: Active, Expired, Cancelled
- **Notification.deliveryStatus**: Sent, Delivered, Failed

### Timestamps

Common timestamp fields:

- `createdDate`: DateTime (creation timestamp)
- `lastLoginDate`: DateTime (last activity)
- `uploadDate`: DateTime (file upload)
- `reviewDate`: DateTime (review submission)
- `sentDate`: DateTime (notification sent)
- `paymentDate`: DateTime (payment processed)
- `bookingDate`: DateTime (booking created)
- `assignedDate`: DateTime (staff assigned)

---

**Generated**: 2026-02-04
**Phase**: COMET Phase 1 - Static Model
**Next Use**: Phase 2 - Object Structuring
