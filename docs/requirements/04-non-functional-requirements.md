# 4. Non-Functional Requirements

<!-- TODO: Workflow Step 12 - Define NFRs using Planguage -->
<!-- All NFRs must use Planguage template with measurable criteria -->

## 4.1 Performance Requirements

### NFR-P-001: Search Response Time

**Tag**: Search.Response.Time
**Gist**: Hotel search results shall return quickly
**Scale**: Time in seconds from search submission to results display
**Meter**: Automated performance test measuring end-to-end response time
**Must**: 3 seconds maximum
**Plan**: 2 seconds average
**Wish**: 1 second average
**Defined**: Search includes query processing, data retrieval, and result rendering
**Past**: Current system averages 5 seconds (baseline)
**Trend**: Improving by 20% per quarter
**Authority**: Product Manager, 2026-02-01
**Stakeholders**: End users (high priority), Operations team (monitoring)
**Risks**: Database performance degradation under high load
**Dependencies**: Requires database indexing optimization (see NFR-P-003)

### NFR-P-002: Concurrent User Capacity

**Tag**: System.Concurrent.Users
**Gist**: System shall support multiple simultaneous users
**Scale**: Number of concurrent active users
**Meter**: Load testing with realistic user scenarios
**Must**: 1,000 concurrent users
**Plan**: 5,000 concurrent users
**Wish**: 10,000 concurrent users
**Defined**: Concurrent users performing search, booking, and profile operations
**Past**: Current system supports 500 users (baseline)
**Trend**: Expected 50% growth in user base annually
**Authority**: System Architect, 2026-02-01
**Stakeholders**: Business (growth planning), Operations (capacity planning)
**Risks**: Cloud infrastructure costs may increase significantly
**Dependencies**: Requires horizontal scaling capability

### NFR-P-003: Database Query Performance

**Tag**: Database.Query.Time
**Gist**: Data retrieval operations shall be fast
**Scale**: Query execution time in milliseconds
**Meter**: Database monitoring tools (query profiler)
**Must**: 500ms maximum for complex queries
**Plan**: 200ms average for typical queries
**Wish**: 100ms average for all queries
**Defined**: Includes hotel search, booking retrieval, user profile queries
**Past**: Current average 800ms (baseline)
**Trend**: Stable with proper indexing
**Authority**: Database Administrator, 2026-02-01
**Stakeholders**: Development team, Operations
**Risks**: Data volume growth may degrade performance
**Dependencies**: Requires proper indexing strategy

## 4.2 Security Requirements

### NFR-S-001: Password Storage Security

**Tag**: Security.Password.Storage
**Gist**: User passwords shall be securely stored
**Scale**: Compliance with OWASP password storage guidelines
**Meter**: Security audit and code review
**Must**: Industry-standard encryption with salt
**Plan**: Adaptive hashing algorithm (bcrypt/argon2)
**Wish**: Hardware security module integration
**Defined**: All user password data at rest
**Past**: Current system uses basic encryption (non-compliant)
**Trend**: Industry moving to argon2
**Authority**: Security Officer, 2026-02-01
**Stakeholders**: Users (privacy), Legal (compliance), Security team
**Risks**: Security breach could expose user data
**Dependencies**: Requires security library integration

### NFR-S-002: Session Timeout

**Tag**: Security.Session.Timeout
**Gist**: Inactive user sessions shall expire automatically
**Scale**: Time in minutes of inactivity before session expires
**Meter**: Session monitoring and automated testing
**Must**: 30 minutes maximum
**Plan**: 15 minutes for authenticated sessions
**Wish**: Configurable by user (5-30 minutes)
**Defined**: Inactivity = no user interaction with system
**Past**: Current system: 60 minutes (too long)
**Trend**: Industry standard 15-20 minutes
**Authority**: Security Officer, 2026-02-01
**Stakeholders**: Users (convenience vs security), Security team
**Risks**: Too short = poor UX, too long = security risk
**Dependencies**: Requires session management framework

## 4.3 Usability Requirements

### NFR-U-001: Mobile Responsiveness

**Tag**: Usability.Mobile.Responsive
**Gist**: System shall be usable on mobile devices
**Scale**: Percentage of features fully functional on mobile (320px-428px width)
**Meter**: Manual testing on target devices, automated responsive tests
**Must**: 100% of core features (search, book, view bookings)
**Plan**: 100% of all features
**Wish**: Native app performance feel
**Defined**: Core features = search, booking, profile, history
**Past**: Current system 70% mobile functional
**Trend**: 60% of traffic from mobile devices
**Authority**: UX Designer, 2026-02-01
**Stakeholders**: Mobile users (majority), Business (revenue)
**Risks**: Complex features may be difficult on small screens
**Dependencies**: Requires responsive design framework

### NFR-U-002: Accessibility Compliance

**Tag**: Usability.Accessibility.WCAG
**Gist**: System shall be accessible to users with disabilities
**Scale**: WCAG 2.1 compliance level
**Meter**: Automated accessibility testing tools + manual testing
**Must**: WCAG 2.1 Level A
**Plan**: WCAG 2.1 Level AA
**Wish**: WCAG 2.1 Level AAA
**Defined**: All user-facing features and content
**Past**: Current system not compliant
**Trend**: Legal requirement in many jurisdictions
**Authority**: UX Designer, Legal, 2026-02-01
**Stakeholders**: Users with disabilities, Legal (compliance)
**Risks**: Non-compliance could result in legal action
**Dependencies**: Requires accessibility framework and training

## 4.4 Reliability Requirements

### NFR-R-001: System Uptime

**Tag**: Reliability.System.Uptime
**Gist**: System shall be available for use
**Scale**: Percentage uptime per month
**Meter**: Uptime monitoring service (24/7)
**Must**: 99% uptime (7.2 hours downtime/month maximum)
**Plan**: 99.9% uptime (43 minutes downtime/month)
**Wish**: 99.99% uptime (4.3 minutes downtime/month)
**Defined**: Uptime = system accessible and functional
**Past**: Current system 98.5% uptime
**Trend**: Industry standard 99.9% for SaaS
**Authority**: Operations Manager, 2026-02-01
**Stakeholders**: All users, Business (revenue), Support team
**Risks**: Downtime results in lost revenue and poor reputation
**Dependencies**: Requires redundancy and failover mechanisms

### NFR-R-002: Data Backup Frequency

**Tag**: Reliability.Backup.Frequency
**Gist**: User data shall be backed up regularly
**Scale**: Time interval between backups in hours
**Meter**: Backup system logs and verification tests
**Must**: Every 24 hours (daily)
**Plan**: Every 6 hours
**Wish**: Continuous (real-time replication)
**Defined**: All user data, bookings, and system configuration
**Past**: Current system weekly backups
**Trend**: Industry moving to continuous backup
**Authority**: Operations Manager, 2026-02-01
**Stakeholders**: Users (data safety), Business (compliance), Legal
**Risks**: Data loss could be catastrophic for business
**Dependencies**: Requires backup infrastructure and testing

## 4.5 Scalability Requirements

### NFR-SC-001: Horizontal Scalability

**Tag**: Scalability.Horizontal
**Gist**: System shall scale by adding more servers
**Scale**: Time to deploy additional capacity in minutes
**Meter**: Infrastructure deployment tests
**Must**: 30 minutes maximum
**Plan**: 10 minutes average
**Wish**: 5 minutes automatic (auto-scaling)
**Defined**: Deploy and integrate additional application servers
**Past**: Current system requires manual deployment (2 hours)
**Trend**: Moving to container orchestration
**Authority**: System Architect, 2026-02-01
**Stakeholders**: Operations (scaling events), Business (growth support)
**Risks**: Poor scalability limits business growth
**Dependencies**: Requires containerization and orchestration platform

---
*All NFRs use Planguage for measurable, testable specifications. Metrics drive verification and validation.*
