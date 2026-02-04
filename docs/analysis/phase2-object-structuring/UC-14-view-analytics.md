# Object Structuring: View Analytics

**Use Case**: docs/requirements/use-cases/UC-14-view-analytics.md
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

**Analysis**: Owner initiates via dashboard button, views analytics sections with charts and metrics, adjusts time period filters, exports reports through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| AnalyticsDashboardUI | «user interaction» | Maps from HostelOwner external user. Handles analytics dashboard display with multiple sections (Overview KPIs, Revenue, Occupancy, Booking, Guest, Review), interactive charts (hover for details, click to drill down), time period filter (7/30/90 days, YTD, custom range), comparison mode toggle, export button. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: "Retrieves property data for time period" → Booking, Payment, Review entities (aggregated)
- Step 4: "Overview KPIs" → Aggregated data from Booking, Payment, Review entities
- Step 5: "Revenue Analytics" → Payment entity (revenue trend, by room type, commission)
- Step 6: "Occupancy Analytics" → Booking, RoomType, Bed entities (occupancy calculations)
- Step 7: "Booking Analytics" → Booking entity (source, lead time, length of stay, cancellation)
- Step 8: "Guest Analytics" → User entity (nationality, age), Booking entity (group size, repeat guests)
- Step 9: "Review Analytics" → Review entity (ratings, sentiment, response rate)
- Step 13: "Export report" → All entity data aggregated into report
- All entities read-only for analytics

**Referenced Data**:
- **Booking**: Booking activity
  - Attributes: bookingId, checkInDate, checkOutDate, numberOfGuests, status, bookingDate
- **Payment**: Revenue data
  - Attributes: paymentId, amount, paymentDate, paymentMethod
- **Review**: Rating and feedback
  - Attributes: reviewId, rating, comment, reviewDate, status
- **RoomType**: Occupancy and room performance
  - Attributes: roomTypeId, name, availableRooms, totalRooms
- **Bed**: Bed-level occupancy (dorms)
  - Attributes: bedId, status
- **User**: Guest demographics (anonymized)
  - Attributes: nationality, dateOfBirth
- **Hostel**: Property reference
  - Attributes: hostelId, rating

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Booking | Yes | ✅ | Booking analytics (read-only aggregation) |
| Payment | Yes | ✅ | Revenue analytics (read-only aggregation) |
| Review | Yes | ✅ | Review analytics (read-only aggregation) |
| RoomType | Yes | ✅ | Occupancy analytics (read-only) |
| Bed | Yes | ✅ | Bed-level occupancy (read-only) |
| User | Yes | ✅ | Guest demographics (anonymized, read-only) |
| Hostel | Yes | ✅ | Property context |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (dashboard with charts, metrics, visualizations)
- [x] Printer (PDF export)
- [ ] External device
- [x] Same as input (reusing AnalyticsDashboardUI)

**Analysis**:
- Steps 3-9, 12: Display analytics dashboard with interactive charts
- Step 13: Export report as PDF or Excel file
- All output via web interface

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| AnalyticsDashboardUI | «user interaction» | Reused for output - displays KPI cards, interactive charts (line for trends, bar for comparisons, pie for composition), heat maps, color-coded metrics (green=positive, red=negative), tooltips, drill-down views, comparison mode side-by-side display |
| ReportExporter | «output» | Generates analytics reports (step 13): PDF format (formatted with charts), Excel format (raw data + charts), includes time period, all sections, owner branding. Download to owner's device. |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Analytics viewing is stateless data retrieval and visualization:
- No sequential workflow or state transitions
- Time period filter is data filtering parameter, not state
- Comparison mode is display configuration, not state
- Drill-down is navigation, not state change
- Export is one-time operation
- All operations are read-only queries and calculations

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| AnalyticsCoordinator | «coordinator» | No | Coordinates analytics retrieval: queries booking/payment/review data for time period → aggregates metrics → generates visualizations → handles filter changes → coordinates export. Stateless data orchestration. |

---

## 5. Application Logic

**Complex Rules**:
- [ ] Multi-condition validation (analytics are read-only)
- [x] Algorithms (aggregation, trend calculation, forecasting, sentiment analysis)
- [ ] Cross-entity validation (read-only queries)

**Identified Logic**:
- **KPI calculation**: Total revenue, ADR (Average Daily Rate), occupancy rate, booking count, average rating, cancellation rate, % changes
- **Revenue aggregation**: Trend charts, by room type breakdown, commission deduction (BR-074), forecast
- **Occupancy calculation**: Occupancy rate = (booked beds/available beds) × 100, excludes blocked dates (BR-075)
- **Trend analysis**: Time-series calculations, comparison with previous period
- **Guest demographics**: Anonymization (BR-076), aggregated breakdown
- **Sentiment analysis**: Review text analysis for positive/neutral/negative classification, word cloud

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| RevenueAnalyzer | «algorithm» | Calculates revenue metrics: total revenue, Average Daily Rate (ADR), revenue by room type (pie chart), commission breakdown (BR-074 net amount after platform fees), revenue trend chart (daily/weekly/monthly), revenue forecast (next 30 days based on confirmed bookings). |
| OccupancyAnalyzer | «algorithm» | Calculates occupancy metrics: occupancy rate % = (occupied beds/available beds) × 100, excludes blocked dates (BR-075), occupancy by room type, peak vs off-peak comparison, occupancy calendar heat map (color-coded by %). Bed-level for dorms, room-level for private. |
| BookingAnalyzer | «service» | Analyzes booking patterns: booking source breakdown (direct, search, referral), lead time distribution (how far in advance), average length of stay, cancellation rate and reasons, booking conversion rate (views → bookings). |
| GuestAnalyzer | «service» | Analyzes guest demographics: nationality breakdown (top 10 countries), age distribution (18-25, 26-35, 36-45, 46+), repeat guest rate %, group size distribution (solo, couple, group). Data anonymized per BR-076. |
| ReviewAnalyzer | «algorithm» | Analyzes review data: average rating trend, rating breakdown by category (cleanliness, location, staff, value), sentiment analysis (positive/neutral/negative classification), word cloud generation, response rate %. |
| ChartGenerator | «service» | Generates visualizations: line charts (trends), bar charts (comparisons), pie charts (composition), heat maps (occupancy calendar), interactive charts (hover tooltips, click drill-down), color-coded metrics. Responsive for tablet/mobile. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| AnalyticsDashboardUI | «user interaction» | Boundary (I/O) | HostelOwner | Analytics dashboard interface |
| ReportExporter | «output» | Boundary (Output) | N/A | PDF/Excel report generation |
| AnalyticsCoordinator | «coordinator» | Control | N/A | Analytics data orchestration (stateless) |
| Booking | «entity» | Entity | Booking | Booking data (read-only) |
| Payment | «entity» | Entity | Payment | Revenue data (read-only) |
| Review | «entity» | Entity | Review | Review data (read-only) |
| RoomType | «entity» | Entity | RoomType | Room performance (read-only) |
| Bed | «entity» | Entity | Bed | Bed occupancy (read-only) |
| User | «entity» | Entity | User | Guest demographics (anonymized, read-only) |
| Hostel | «entity» | Entity | Hostel | Property context |
| RevenueAnalyzer | «algorithm» | Application Logic | N/A | Revenue metrics calculation |
| OccupancyAnalyzer | «algorithm» | Application Logic | N/A | Occupancy metrics calculation |
| BookingAnalyzer | «service» | Application Logic | N/A | Booking pattern analysis |
| GuestAnalyzer | «service» | Application Logic | N/A | Guest demographics analysis |
| ReviewAnalyzer | «algorithm» | Application Logic | N/A | Review and sentiment analysis |
| ChartGenerator | «service» | Application Logic | N/A | Visualization generation |

**Total**: 16 objects (2 boundary, 7 entity, 1 control, 6 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actors**: None
- **Boundary**: AnalyticsDashboardUI, ReportExporter
- **Control**: AnalyticsCoordinator
- **Entity**: Booking, Payment, Review, RoomType, Bed, User, Hostel (all read-only)
- **Logic**: RevenueAnalyzer, OccupancyAnalyzer, BookingAnalyzer, GuestAnalyzer, ReviewAnalyzer, ChartGenerator

**Key Interactions**:
1. HostelOwner → AnalyticsDashboardUI: Navigate to analytics
2. AnalyticsDashboardUI → AnalyticsCoordinator: Request analytics (default 30 days)
3. AnalyticsCoordinator → Booking: Query booking data for period
4. AnalyticsCoordinator → Payment: Query revenue data
5. AnalyticsCoordinator → Review: Query review data
6. AnalyticsCoordinator → RoomType: Query room inventory
7. AnalyticsCoordinator → Bed: Query bed status
8. AnalyticsCoordinator → RevenueAnalyzer: Calculate revenue metrics
9. RevenueAnalyzer: Total revenue, ADR, by room type, commission, forecast
10. AnalyticsCoordinator → OccupancyAnalyzer: Calculate occupancy
11. OccupancyAnalyzer: Occupancy rate, by room type, heat map data
12. AnalyticsCoordinator → BookingAnalyzer: Analyze booking patterns
13. AnalyticsCoordinator → GuestAnalyzer: Analyze demographics
14. GuestAnalyzer → User: Query guest data (anonymized)
15. AnalyticsCoordinator → ReviewAnalyzer: Analyze reviews
16. AnalyticsCoordinator → ChartGenerator: Generate charts
17. ChartGenerator: Line charts, bar charts, pie charts, heat maps
18. AnalyticsCoordinator → AnalyticsDashboardUI: Display dashboard
19. HostelOwner → AnalyticsDashboardUI: Adjust time period filter
20. AnalyticsCoordinator recalculates for new period
21. HostelOwner → AnalyticsDashboardUI: Click "Export Report"
22. AnalyticsDashboardUI → AnalyticsCoordinator: Generate export
23. AnalyticsCoordinator → ReportExporter: Create PDF/Excel
24. ReportExporter: Generate report with charts
25. ReportExporter → HostelOwner: Download file

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - AnalyticsCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
  - AnalyticsDashboardUI ← HostelOwner
  - ReportExporter ← N/A (output device)
- [x] Entities in Phase 1:
  - Booking ✅
  - Payment ✅
  - Review ✅
  - RoomType ✅
  - Bed ✅
  - User ✅
  - Hostel ✅
- [x] Control justified (stateless coordinator for read-only analytics)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (aggregation algorithms, analyzers, chart generation)

---

## Alternative Flows Mapping

**Step 2 Alternative** (No booking history):
- AnalyticsCoordinator queries entities, finds zero records
- AnalyticsDashboardUI displays "No data available yet"
- Shows tips to get first bookings
- Displays sample dashboard preview for demonstration

**Step 10 Alternative** (Custom date range):
- HostelOwner selects custom range
- AnalyticsDashboardUI displays calendar picker
- Owner specifies start and end dates
- AnalyticsCoordinator validates max 365 days
- Recalculates analytics for custom period

**Step 13 Alternative** (Export fails):
- ReportExporter encounters error (report too large, timeout)
- AnalyticsCoordinator receives error
- AnalyticsDashboardUI displays error
- Suggests narrower date range or simplified report

**Comparison Mode**:
- Owner enables "Compare with previous period" toggle
- AnalyticsCoordinator queries two time periods
- ChartGenerator displays side-by-side comparison
- All metrics show % change (green=increase, red=decrease)
- Charts display two lines (current vs previous)

**Drill-Down View**:
- Owner clicks data point on chart
- AnalyticsCoordinator retrieves detailed breakdown
- Example: Click pie slice → shows daily revenue for that room type
- Breadcrumb navigation to return to overview

---

## Business Rules

- **BR-072**: Near real-time updates (15-minute data refresh) (AnalyticsCoordinator refresh interval)
- **BR-073**: Historical data retained indefinitely (query any date range)
- **BR-074**: Revenue shows net after commission (RevenueAnalyzer deducts platform fees)
- **BR-075**: Occupancy excludes blocked dates (OccupancyAnalyzer calculation)
- **BR-076**: Guest demographics anonymized (GuestAnalyzer aggregates only)
- **BR-077**: Export available at no cost (ReportExporter unrestricted)

---

## Performance Considerations

- **Dashboard load**: ≤ 3 seconds for 12 months of data
- **Chart rendering**: ≤ 2 seconds
- **Export generation**: ≤ 10 seconds for standard date ranges
- **Data refresh**: Every 15 minutes (BR-072)
- **Query optimization**: Aggregation queries indexed for performance
- **Caching**: Frequently accessed metrics cached (15-minute TTL)

---

## Notes

- **Stateless design**: Read-only data retrieval and visualization
- **KPI overview**: Quick health check at top
- **Multiple dimensions**: Revenue, occupancy, booking, guest, review analytics
- **Interactive charts**: Hover for details, click to drill down
- **Comparison mode**: Essential for trend analysis
- **Export functionality**: External analysis and reporting (BR-077)
- **Data privacy**: Guest demographics anonymized (BR-076)
- **Real-time**: 15-minute refresh keeps data current (BR-072)
- **Mobile access**: Responsive charts for on-the-go monitoring
- **Data-driven decisions**: Enables informed pricing, marketing, operations
