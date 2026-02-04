# Phase 2 Object Analysis Statistics

**Generated**: 2026-02-04

---

## Overall Statistics

**Total Objects**: 287
**Total Use Cases**: 24
**Average Objects per UC**: 12.0

---

## Objects by Category

| Category | Count | Percentage |
|----------|------:|-----------:|
| Application Logic | 62 | 21.6% |
| Boundary | 13 | 4.5% |
| Boundary (I/O) | 15 | 5.2% |
| Boundary (Output) | 22 | 7.7% |
| Control | 23 | 8.0% |
| Entity | 136 | 47.4% |
| Logic | 16 | 5.6% |

---

## Objects by Stereotype

| Stereotype | Count | Percentage |
|------------|------:|-----------:|
| entity | 136 | 47.4% |
| service | 40 | 13.9% |
| business logic | 31 | 10.8% |
| user interaction | 24 | 8.4% |
| proxy | 24 | 8.4% |
| coordinator | 14 | 4.9% |
| state-dependent control | 9 | 3.1% |
| algorithm | 7 | 2.4% |
| output | 2 | 0.7% |

---

## State-Dependent Controls

**Total**: 9/24 use cases (37.5%)

| UC | Control Name |
|----|--------------|
| UC-01-REGISTER-ACCOUNT |  |
| UC-02-LOG-IN |  |
| UC-05-BOOK-ACCOMMODATION |  |
| UC-07-SUBMIT-REVIEW |  |
| UC-08-MANAGE-PROFILE |  |
| UC-09-REGISTER-HOSTEL |  |
| UC-13-MANAGE-STAFF |  |
| UC-15-MANAGE-SUBSCRIPTION |  |
| UC-24-CANCEL-BOOKING | CancellationControl |

---

## Objects per Use Case Distribution

| UC | Name | Total Objects | Breakdown |
|----|------|--------------|-----------|
| UC-05-BOOK-ACCOMMODATION | Book Accommodation | 17 | 3 boundary, 7 entity, 1 control, 6 logic |
| UC-12-PROCESS-BOOKING | Process Booking | 16 | 4 boundary, 7 entity, 1 control, 4 logic |
| UC-14-VIEW-ANALYTICS | View Analytics | 16 | 2 boundary, 7 entity, 1 control, 6 logic |
| UC-07-SUBMIT-REVIEW | Submit Review | 14 | 2 boundary, 6 entity, 1 control, 5 logic |
| UC-09-REGISTER-HOSTEL | Register Hostel | 14 | 3 boundary, 5 entity, 1 control, 5 logic |
| UC-03-SEARCH-HOSTELS | Search Hostels | 13 | 2 boundary, 6 entity, 1 control, 4 logic |
| UC-04-VIEW-HOSTEL-DETAILS | View Hostel Details | 13 | 3 boundary, 7 entity, 1 control, 2 logic |
| UC-15-MANAGE-SUBSCRIPTION | Manage Subscription | 13 | 3 boundary, 4 entity, 1 control, 5 logic |
| UC-24-CANCEL-BOOKING | Cancel Booking | 13 | 3 boundary, 7 entity, 1 control, 2 logic |
| UC-06-MANAGE-BOOKINGS | Manage Bookings | 12 | 2 boundary, 6 entity, 1 control, 3 logic |
| UC-08-MANAGE-PROFILE | Manage Profile | 12 | 3 boundary, 4 entity, 1 control, 4 logic |
| UC-10-MANAGE-PROPERTY | Manage Property | 12 | 3 boundary, 4 entity, 1 control, 4 logic |
| UC-16-COMMUNICATE-WITH-GUEST | Communicate With Guest | 12 | 2 boundary, 7 entity, 1 control, 2 logic |
| UC-18-CHECK-OUT-GUEST | Check Out Guest | 12 | 1 boundary, 8 entity, 1 control, 2 logic |
| UC-11-MANAGE-ROOM-TYPES | Manage Room Types | 11 | 1 boundary, 4 entity, 1 control, 5 logic |
| UC-13-MANAGE-STAFF | Manage Staff | 11 | 2 boundary, 4 entity, 1 control, 4 logic |
| UC-17-CHECK-IN-GUEST | Check In Guest | 11 | 1 boundary, 7 entity, 1 control, 2 logic |
| UC-23-MONITOR-PLATFORM-METRICS | Monitor Platform Metrics | 11 | 1 boundary, 7 entity, 1 control, 2 logic |
| UC-19-VIEW-BOOKING-DETAILS | View Booking Details | 10 | 1 boundary, 9 entity, 0 control, 0 logic |
| UC-20-MANAGE-USER-ACCOUNTS | Manage User Accounts | 10 | 2 boundary, 5 entity, 1 control, 2 logic |
| UC-21-MANAGE-PLATFORM-SUBSCRIPTIONS | Manage Platform Subscriptions | 10 | 1 boundary, 6 entity, 1 control, 2 logic |
| UC-22-CONFIGURE-SYSTEM-SETTINGS | Configure System Settings | 9 | 1 boundary, 5 entity, 1 control, 2 logic |
| UC-02-LOG-IN | Log In | 8 | 2 boundary, 2 entity, 1 control, 3 logic |
| UC-01-REGISTER-ACCOUNT | Register Account | 7 | 2 boundary, 2 entity, 1 control, 2 logic |

---

## Conclusion

- **24 use cases** analyzed with **287 total objects**
- **9 state-dependent controls** requiring Phase 4 statecharts
- **13 boundary objects** (4.5%) for external interactions
- **136 entity objects** (47.4%) for data persistence
- **23 control objects** (8.0%) for workflow coordination
- **78 application logic objects** for business rules and services

**Key Insights**:
- Average 12.0 objects per use case indicates moderate complexity
- 37.5% of use cases require state machines (temporal behavior)
- High entity object count (136) reflects data-rich domain
