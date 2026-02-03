# 1. Overall Description

## 1.1 Product Perspective

HostelViet is a standalone B2B SaaS platform designed to modernize hostel management in Vietnam's 2,000+ hostel market. The system serves two distinct user groups: hostel owners who manage properties and travelers who search and book accommodations. Target is to serve 500+ hostels and facilitate 100,000+ bookings annually by 2027.

**Market Focus:**

- Primary market: Vietnam hostel industry (HCMC, Hanoi, Da Nang, Hoi An, Nha Trang)
- Customer segments: Small hostels (1-2 properties), medium chains (3-5 properties), large operators (6+ properties)
- Annual booking volume target: 100,000+ bookings by Year 2

**System Context:**

- Standalone web-based platform accessible via modern web browsers
- Dual-sided platform connecting hostel owners with travelers
- Subscription-based business model: Basic ($29/month), Pro ($79/month), Enterprise ($199/month)
- Free-to-use booking platform for travelers (commission-free model)
- Competitive positioning: Lower cost than international tools ($100+/month), Vietnam-first approach

**System Interfaces:**

- Payment gateways (SePay for VietQR, VNPay for Vietnamese bank transfers)
- Email service providers (SendGrid or AWS SES) for automated communications
- SMS gateway (Twilio or local Vietnam provider) for booking confirmations
- File storage services (AWS S3 or Cloudinary) for property photos and documents
- Google Maps API for location services
- Monitoring systems (ELK Stack, Prometheus, Grafana)

**User Interfaces:**

- Public web portal for travelers (hostel search, booking, profile management)
- Owner dashboard for hostel management (property, bookings, staff, analytics)
- Responsive design supporting desktop, tablet, and mobile devices (60% mobile traffic expected)
- Multi-language support: Vietnamese (primary), English (secondary)
- Mobile-first design optimized for 3G networks (50 Mbps average Vietnam internet)

**Hardware Interfaces:**

- No specific hardware requirements beyond standard web access devices
- Users access system via desktop computers, tablets, or smartphones
- Internet connectivity required for all operations (minimum 2Mbps)
- Compatible with Vietnamese internet infrastructure

**Software Interfaces:**

- Payment gateway APIs for processing VietQR and VNPay transactions
- Email service API for booking confirmations and notifications (95%+ deliverability)
- SMS gateway API for urgent communications
- Cloud storage API for managing property photos (max 20 photos, 5MB each, WebP format)
- Google Maps API for property location and mapping features

**Communications Interfaces:**

- HTTPS (TLS 1.3) for secure web communications
- RESTful APIs for payment webhook notifications from SePay and VNPay
- Real-time updates delivered via Server-Sent Events (SSE) with <1 second latency
- Email and SMS protocols for user notifications (delivery within 30 seconds)

## 1.2 Product Functions

The system provides the following major functions:

**For Hostel Owners:**

- Register and manage hostel properties with detailed information
- Define room types, bed configurations, and pricing structures
- Track real-time availability and manage booking calendars
- Process booking requests and manage reservations
- Assign staff members with specific roles and permissions
- View revenue reports and occupancy analytics
- Manage subscription plans and billing
- Communicate with guests regarding bookings

**For Travelers:**

- Search hostels by location, price, amenities, and availability
- View detailed hostel information with photos and reviews
- Check real-time room/bed availability
- Make instant bookings with secure payment
- Pay via VietQR or VNPay bank transfers
- Receive booking confirmations and updates
- Manage booking history and user profile
- Submit reviews and ratings for completed stays

## 1.3 User Characteristics

**Primary Users:**

- **Hostel Owners/Managers:**
  - Education: Business or hospitality management background
  - Experience: Managing 1-3 hostel properties in tourist cities, familiar with booking systems
  - Technical expertise: Moderate (uses Facebook, Zalo, basic software)
  - Age range: 30-50 years
  - Language: Vietnamese (primary), English (secondary)
  - Usage frequency: Daily for managing properties and bookings
  - Pain Points: Manual tracking via Excel, double bookings, limited analytics, high OTA commissions (15-20%)
  - Goals: Reduce admin time by 50%, increase direct bookings 30%+, real-time dashboards

- **Hostel Staff (Receptionists):**
  - Education: High school or vocational training
  - Experience: Front desk operations, customer service at hostels
  - Technical expertise: Basic computer and smartphone skills
  - Age range: 22-35 years
  - Language: Vietnamese (primary)
  - Usage frequency: Daily during work shifts
  - Pain Points: Manual check-in, difficulty tracking bed assignments in dorms
  - Goals: Quick check-in workflow, clear bed visibility, access to booking details

- **Travelers/Guests:**
  - Education: Varied (high school to university)
  - Experience: Comfortable with online shopping and booking platforms
  - Technical expertise: Comfortable using websites and mobile apps
  - Age range: 18-35 years (primary demographic - backpackers, digital nomads)
  - Nationality: 60% international, 40% Vietnamese
  - Language: Vietnamese (domestic), English (international visitors)
  - Budget: $5-15 USD per night
  - Trip Duration: 3-14 nights per booking
  - Usage frequency: Occasional, primarily during trip planning
  - Pain Points: Outdated availability, limited payment options, language barriers
  - Goals: Real-time availability, local payment methods (VietQR), genuine reviews

**Secondary Users:**

- **Hostel Cleaning Staff:**
  - Education: Minimal formal education required
  - Experience: Housekeeping and room maintenance
  - Technical expertise: Very basic, limited to viewing assigned tasks
  - Age range: 20-55 years
  - Language: Vietnamese
  - Usage frequency: Daily for task assignments

## 1.4 Constraints

**Regulatory Constraints:**

- Must comply with Vietnamese data protection laws and regulations (GDPR-compliant data handling)
- Must adhere to Vietnamese tax and invoicing requirements for subscription billing
- Must follow Vietnamese consumer protection laws for booking transactions
- Must comply with State Bank of Vietnam payment gateway regulations
- Must implement PCI DSS Level 1 compliance for payment security
- Must maintain data residency with Vietnam-based hosting for local data
- Must meet WCAG 2.1 Level AA compliance for accessibility

**Technical Constraints:**

- System must support modern web browsers (Chrome, Firefox, Safari, Edge - last 2 versions)
- Must be accessible via standard internet connections (minimum 2Mbps)
- Must operate within Vietnamese internet infrastructure (average 50 Mbps)
- Mobile responsiveness required for all interfaces (60% mobile traffic expected)
- Session timeout required for security (15 minutes inactivity)
- JWT authentication with 15-minute access tokens, 7-day refresh tokens
- Password policy: Minimum 8 characters with complexity requirements
- API rate limiting: 100 requests/minute per IP for public APIs
- SSL/TLS certificates required for all communications
- Page load time: <3 seconds on 3G networks

**Design Constraints:**

- User interface must support Vietnamese language as primary
- Date/time formats must follow Vietnamese conventions (dd/mm/yyyy)
- Currency display must be in Vietnamese Dong (VND)
- Must follow WCAG 2.1 Level AA accessibility guidelines
- Must provide clear error messages in user's language
- Mobile-first responsive design required
- SEO optimization for Vietnam travel keywords
- Maximum file size for property photos: 5MB each, 20 photos per property
- Image format: WebP for optimized web delivery

**Business Constraints:**

- Subscription tiers with feature gating: Basic ($29/mo), Pro ($79/mo), Enterprise ($199/mo)
- Free-tier limitations for trial hostel accounts
- Payment processing fees passed to hostel owners
- Commission-free model for traveler bookings
- 24-hour booking cancellation window for refunds
- Maximum 10 properties per Basic tier, 50 per Pro/Enterprise (configurable)
- Grace period for failed subscription payments: 7 days
- Monthly churn rate target: <10%
- Customer lifetime value target: $2,000+ average
- Platform uptime requirement: 99.9% availability

## 1.5 Assumptions and Dependencies

**Assumptions:**

- Users have access to devices with modern web browsers (Chrome, Firefox, Safari, Edge)
- Users have reliable internet connectivity (minimum 2Mbps, average 50Mbps in Vietnam)
- Hostel owners have valid business registration in Vietnam
- Travelers have valid email addresses for account registration
- Users can receive SMS messages for critical notifications
- Hostel owners understand basic property management concepts
- Payment gateway services maintain 99.9% uptime (SePay, VNPay)
- Users accept terms of service and privacy policy
- Vietnamese internet infrastructure maintains stable connectivity
- OTA commissions remain at 15-20% (competitive advantage for direct bookings)
- Target market of 2,000+ hostels in Vietnam remains stable
- Users are willing to adopt new booking platforms with local payment methods

**Dependencies:**

- **Payment Gateways:** SePay and VNPay services must be operational for processing payments (99.5% success rate required)
  - Merchant onboarding timeline: 2-4 weeks
  - API rate limits and webhook reliability
- **Email Service:** SendGrid or AWS SES must be available for notifications (95%+ deliverability required)
  - Delivery within 30 seconds for booking confirmations
  - Support for 50,000 notifications/day
- **SMS Gateway:** Twilio or local Vietnam provider must be functional for urgent communications
  - <1% spam complaint rate required
- **Cloud Storage:** AWS S3 or Cloudinary must be accessible for property photos
  - CDN distribution for static assets and images
  - WebP format optimization support
- **Google Maps API:** Must be available for location services and mapping
  - API rate limits and quota management
- **Internet Infrastructure:** Vietnamese ISPs must maintain connectivity and DNS resolution
- **Web Browser Support:** Modern browsers must continue supporting current web standards (ES2020+)
- **Mobile Devices:** Smartphone and tablet manufacturers must maintain OS compatibility (iOS 14+, Android 10+)
- **Currency Exchange:** VND exchange rates available for international payment processing
- **Monitoring Services:** ELK Stack, Prometheus, Grafana must be operational for system health tracking
- **Domain and SSL:** Domain registration and SSL certificate renewals must be maintained
- **Development Team:** 2-3 full-stack developers available for 12-16 weeks MVP timeline
- **Infrastructure Budget:** $500-1,000/month initially for cloud hosting costs

**External Service Uptime Requirements:**

- Payment gateways: 99.9% uptime
- Email delivery: 95%+ deliverability
- Cloud storage: 99.9% availability
- API response time: p95 <200ms, p99 <500ms
- Database queries: <100ms for 95% of queries
- Search latency: <500ms for 10,000+ records

---

_This section establishes context for detailed requirements that follow._
