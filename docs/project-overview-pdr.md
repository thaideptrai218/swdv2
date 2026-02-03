# HostelViet - Product Development Requirements (PDR)

**Version:** 1.0
**Last Updated:** 2026-02-03
**Status:** Initial Planning

## Executive Summary

HostelViet is a B2B SaaS platform designed to modernize hostel management in Vietnam. The platform provides hostel owners with comprehensive property management tools while offering travelers a seamless booking experience with localized payment options.

## Product Vision

**Mission:** Empower Vietnamese hostel owners with enterprise-grade management tools while creating a trusted booking platform for travelers exploring Vietnam.

**Vision:** Become the leading hostel management platform in Vietnam, serving 500+ hostels and facilitating 100,000+ bookings annually by 2027.

**Value Proposition:**

- **For Hostel Owners:** Eliminate manual booking management, increase occupancy rates, reduce operational overhead
- **For Travelers:** Discover authentic hostels with transparent pricing, real-time availability, and local payment methods

## Target Market

### Primary Market

**Vietnam Hostel Industry**

- Market Size: 2,000+ hostels across Vietnam (2025 estimate)
- Key Cities: Ho Chi Minh City, Hanoi, Da Nang, Hoi An, Nha Trang
- Customer Segments:
    - Small hostels (1-2 properties, 20-50 beds)
    - Medium chains (3-5 properties, 50-200 beds)
    - Large operators (6+ properties, 200+ beds)

### Secondary Users

**International and Domestic Travelers**

- Backpackers and budget travelers (18-35 years old)
- Digital nomads and long-term travelers
- Vietnamese domestic tourists
- Annual Volume Target: 100,000+ bookings by Year 2

## User Personas

### Persona 1: Hostel Owner (Primary Customer)

**Profile:**

- Age: 30-50
- Role: Small business owner or property manager
- Properties: 1-3 hostels in tourist cities
- Tech Savvy: Moderate (uses Facebook, Zalo, basic software)

**Pain Points:**

- Manual booking tracking via Excel or notebooks
- Double bookings and availability conflicts
- Limited visibility into revenue and occupancy trends
- Difficulty managing multiple staff schedules
- Commission fees from OTAs (15-20%)

**Goals:**

- Reduce administrative time by 50%
- Increase direct bookings to avoid OTA commissions
- Access real-time occupancy and revenue data
- Streamline guest check-in/check-out processes

**Success Metrics:**

- Time saved: 10+ hours/week
- Direct booking increase: 30%+
- Revenue visibility: Real-time dashboards
- Guest satisfaction: 4.5+ star average

### Persona 2: Guest/Traveler (End User)

**Profile:**

- Age: 22-32
- Nationality: 60% international, 40% Vietnamese
- Trip Duration: 3-14 nights per booking
- Budget: $5-15 USD/night

**Pain Points:**

- Outdated availability on booking platforms
- Limited payment options (credit card only)
- Lack of authentic local hostel reviews
- Language barriers when booking

**Goals:**

- Find authentic hostels with real-time availability
- Pay using local methods (VietQR, bank transfer)
- Read genuine reviews from other travelers
- Easy cancellation and booking management

**Success Metrics:**

- Booking completion rate: 70%+
- Search-to-book time: <5 minutes
- Payment success rate: 95%+
- Repeat booking rate: 25%+

### Persona 3: Hostel Staff (Secondary User)

**Profile:**

- Age: 22-35
- Role: Receptionist or hostel manager
- Tech Experience: Basic smartphone and computer skills

**Pain Points:**

- Manual check-in processes
- Difficulty tracking bed assignments in dorms
- Communication gaps with owners
- Limited access to booking information

**Goals:**

- Quick check-in/check-out workflow
- Clear bed assignment visibility
- Access to guest booking details
- Simple reporting to owners

## Core Features

### 1. Multi-Tenant Property Management

**Description:** Hostel owners manage multiple properties from unified dashboard

**Functional Requirements:**

- Create and edit property profiles (name, location, amenities, photos)
- Define room types (private rooms, dorms) and bed inventory
- Set pricing rules (base rates, seasonal pricing, discounts)
- Upload property photos (max 20 photos, 5MB each)
- Multi-language support (English, Vietnamese)

**Non-Functional Requirements:**

- Load property dashboard in <2 seconds
- Support up to 50 properties per tenant
- Image optimization for web delivery (WebP format)

**Success Criteria:**

- 95% of users can create property in <10 minutes
- Zero data leakage between tenants
- 99.9% uptime for property data access

### 2. Real-Time Booking Management

**Description:** Track bookings, availability, and reservations in real-time

**Functional Requirements:**

- Calendar view with occupancy visualization
- Booking creation (manual and automated from public portal)
- Booking status tracking (pending, confirmed, checked-in, completed, cancelled)
- Bed/room assignment for dorm bookings
- Automated booking confirmations via email/SMS
- Booking modification and cancellation

**Non-Functional Requirements:**

- Real-time availability updates via SSE (<1 second latency)
- Handle 1,000 concurrent bookings per property
- Prevent double bookings with optimistic locking

**Success Criteria:**

- Zero double bookings
- Booking confirmation sent within 30 seconds
- 99% availability accuracy

### 3. Guest Booking Portal

**Description:** Public-facing platform for travelers to search and book hostels

**Functional Requirements:**

- Full-text search with filters (location, price range, amenities, ratings)
- Hostel listing pages with photos, descriptions, reviews
- Real-time availability calendar
- Multi-step booking flow (search → select → pay → confirm)
- Guest account management (profile, booking history)
- Review and rating system (1-5 stars with comments)

**Non-Functional Requirements:**

- Search results in <500ms for 1,000+ hostels
- Mobile-responsive design (50%+ mobile traffic)
- Support 10,000 concurrent users
- SEO optimized for Vietnam travel keywords

**Success Criteria:**

- 70% search-to-booking conversion rate
- Mobile usability score: 90+
- Page load time: <3 seconds on 3G

### 4. Subscription and Payment Management

**Description:** Tiered subscription plans with Vietnam-localized payments

**Functional Requirements:**

- Subscription tiers: Basic ($29/month), Pro ($79/month), Enterprise ($199/month)
- Feature gating per tier (property limits, staff accounts, advanced analytics)
- Auto-renewal and payment reminders
- Payment integration: SePay (VietQR), VNPay
- Invoice generation and download
- Grace period for failed payments (7 days)

**Non-Functional Requirements:**

- Payment processing: <10 seconds
- 99.5% payment success rate
- PCI DSS compliance for card storage
- Automated retry for failed payments

**Success Criteria:**

- 95% subscription retention rate
- <5% payment failure rate
- Auto-renewal rate: 85%+

### 5. Analytics and Reporting

**Description:** Business intelligence for hostel owners

**Functional Requirements:**

- Occupancy rate dashboard (daily, weekly, monthly)
- Revenue reports with trend analysis
- Booking source tracking (direct vs. third-party)
- Guest demographics and behavior analytics
- Exportable reports (PDF, CSV)
- Comparative analysis across properties

**Non-Functional Requirements:**

- Report generation: <5 seconds for 12 months of data
- Real-time metric updates every 15 minutes
- Data retention: 3 years minimum

**Success Criteria:**

- 80% of owners view reports weekly
- Revenue insights accuracy: 99.9%
- Report export success rate: 100%

### 6. Staff Management and Permissions

**Description:** Role-based access control for hostel teams

**Functional Requirements:**

- User roles: Owner, Manager, Receptionist, Cleaner
- Granular permissions (view bookings, edit pricing, manage staff)
- Activity logs for audit trails
- Staff invitation and onboarding
- Role switching for multi-role users

**Non-Functional Requirements:**

- Permission checks: <50ms latency
- Support 100 staff members per tenant
- Audit log retention: 2 years

**Success Criteria:**

- Zero unauthorized access incidents
- Staff onboarding time: <5 minutes
- Permission accuracy: 100%

### 7. Notifications and Communication

**Description:** Automated alerts and messaging for owners, staff, and guests

**Functional Requirements:**

- Booking confirmations and reminders
- Payment success/failure notifications
- Availability alerts for low inventory
- Staff task assignments
- Email and in-app notifications
- Customizable notification preferences

**Non-Functional Requirements:**

- Notification delivery: <30 seconds
- Email deliverability rate: 95%+
- Support 50,000 notifications/day

**Success Criteria:**

- 90% notification open rate
- <1% spam complaints
- Guest satisfaction with communication: 4.5+/5

## Technical Requirements

### Architecture Constraints

- **Monorepo:** Turborepo with workspace management
- **Frontend Framework:** Next.js 15+ with App Router
- **Backend Framework:** NestJS with tRPC
- **Database:** PostgreSQL 16+ with row-level security
- **Real-time:** Server-Sent Events (no WebSockets)

### Performance Requirements

- **API Response Time:** p95 <200ms, p99 <500ms
- **Page Load Time:** <3 seconds on 3G networks
- **Database Queries:** <100ms for 95% of queries
- **Search Latency:** <500ms for 10,000 records
- **Concurrent Users:** 10,000+ simultaneous connections

### Security Requirements

- **Authentication:** JWT with 15-minute access tokens, 7-day refresh tokens
- **Authorization:** RBAC with row-level security (RLS)
- **Data Encryption:** TLS 1.3 in transit, AES-256 at rest
- **Password Policy:** Minimum 8 characters, complexity requirements
- **Rate Limiting:** 100 requests/minute per IP for public APIs
- **Audit Logging:** All data mutations logged with user context

### Scalability Requirements

- **Horizontal Scaling:** Stateless API servers (auto-scale on CPU >70%)
- **Database:** Read replicas for analytics queries
- **Cache Layer:** Redis with 90%+ hit rate for frequent queries
- **File Storage:** CDN for static assets and images
- **Queue System:** RabbitMQ for async jobs (emails, notifications)

### Compliance Requirements

- **Data Privacy:** GDPR-compliant data handling
- **Payment Security:** PCI DSS Level 1 compliance
- **Data Residency:** Vietnam-based hosting for local data
- **Accessibility:** WCAG 2.1 Level AA compliance

## Integration Requirements

### Third-Party Services

- **Payment Gateways:** SePay (VietQR), VNPay
- **File Storage:** AWS S3 or Cloudinary
- **Email Service:** SendGrid or AWS SES
- **SMS Provider:** Twilio or local Vietnam provider
- **Maps:** Google Maps API for location services
- **Monitoring:** ELK Stack, Prometheus, Grafana

### API Contracts

- **tRPC Endpoints:** Type-safe client-server communication
- **REST Webhooks:** Payment provider callbacks
- **OpenAPI Spec:** Public API documentation

## Success Metrics

### Business KPIs

- **Customer Acquisition:** 50 hostels by Month 6, 200 by Month 12
- **Revenue:** $10,000 MRR by Month 6, $50,000 MRR by Month 12
- **Churn Rate:** <10% monthly
- **Customer Lifetime Value:** $2,000+ average

### Product KPIs

- **Booking Volume:** 5,000 bookings/month by Month 12
- **Platform Uptime:** 99.9% availability
- **User Engagement:** 80% weekly active users (hostel owners)
- **Mobile Traffic:** 50%+ of guest bookings via mobile

### Quality KPIs

- **Bug Density:** <1 critical bug per 1,000 lines of code
- **Test Coverage:** 80%+ code coverage
- **Performance:** 95% of API calls <200ms
- **Customer Satisfaction:** NPS score 50+

## Risk Assessment

### Technical Risks

| Risk                                | Probability | Impact   | Mitigation                                                |
| ----------------------------------- | ----------- | -------- | --------------------------------------------------------- |
| Payment provider integration delays | High        | High     | Early API testing, sandbox environment, fallback provider |
| Database performance degradation    | Medium      | High     | Indexed queries, caching, read replicas                   |
| Multi-tenancy data leakage          | Low         | Critical | RLS enforcement, security audits, penetration testing     |
| Real-time notification failures     | Medium      | Medium   | Message queue with retries, fallback to polling           |

### Business Risks

| Risk                                     | Probability | Impact | Mitigation                                                      |
| ---------------------------------------- | ----------- | ------ | --------------------------------------------------------------- |
| Low hostel adoption                      | Medium      | High   | Free trial period, onboarding support, local partnerships       |
| Competition from international platforms | High        | Medium | Focus on Vietnam-specific features, local payments              |
| Regulatory changes in hospitality        | Low         | High   | Legal compliance monitoring, flexible architecture              |
| Payment fraud                            | Medium      | High   | Anti-fraud detection, manual review for suspicious transactions |

## Development Roadmap

### Phase 1: MVP (Months 1-3)

- Core property and booking management
- Basic guest portal with search
- Subscription payment integration
- Owner dashboard with analytics

### Phase 2: Growth (Months 4-6)

- Advanced reporting and analytics
- Mobile app (React Native)
- Multi-language support
- Third-party integrations (booking.com export)

### Phase 3: Scale (Months 7-12)

- AI-powered pricing recommendations
- Advanced fraud detection
- Loyalty program for frequent travelers
- API marketplace for developers

## Competitive Analysis

### Competitors

- **Booking.com/Agoda:** High OTA commissions (15-20%), global focus
- **Local Vietnam platforms:** Limited features, poor UX
- **Cloudbeds/Guesty:** International tools, expensive ($100+/month)

### Competitive Advantages

- **Vietnam-first:** Local payments, Vietnamese UI, local support
- **Fair Pricing:** $29-199/month vs. $100+ for international tools
- **Direct Booking Focus:** Help hostels reduce OTA dependency
- **Modern Tech Stack:** Fast, mobile-responsive, real-time updates

## Constraints and Dependencies

### External Dependencies

- SePay/VNPay merchant onboarding (2-4 weeks)
- Domain registration and SSL certificates
- Cloud hosting setup (AWS/Google Cloud)
- Third-party API rate limits

### Resource Constraints

- Development team: 2-3 full-stack developers
- Timeline: 12-16 weeks for MVP
- Budget: Infrastructure costs ($500-1,000/month initially)

### Technical Constraints

- Vietnam internet speeds (average 50 Mbps)
- Mobile-first design (60% mobile traffic expected)
- Multi-language support complexity
- Payment provider limitations

## Future Considerations

### Post-MVP Features

- **Channel Manager:** Sync inventory with OTAs (Booking.com, Airbnb)
- **Dynamic Pricing:** AI-powered rate optimization
- **Mobile Apps:** Native iOS/Android applications
- **Loyalty Program:** Rewards for frequent bookers
- **Property Management:** Housekeeping, maintenance tracking
- **Guest Experience:** Self-check-in kiosks, digital keys

### Scalability Planning

- **Geographic Expansion:** Southeast Asia markets (Thailand, Cambodia)
- **Vertical Expansion:** Hotels, guesthouses, boutique properties
- **B2B Partnerships:** Travel agencies, tour operators
- **White-Label Solution:** Hostel chains with custom branding

---

**Document Owner:** Product Manager
**Approval Required:** Technical Lead, Business Stakeholder
**Review Cycle:** Monthly during development, quarterly post-launch
