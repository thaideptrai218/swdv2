# UC-14: View Analytics

| Field | Description |
|-------|-------------|
| **Use Case Name** | View Analytics |
| **Use Case ID** | UC-14 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Hostel Owner views property performance analytics including revenue, occupancy, booking trends, guest demographics, and review ratings. System aggregates data and displays visualizations for business insights. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A1: Hostel Owner. Secondary: None |
| **Preconditions** | Owner authenticated. Owner has active property with booking history. System operational. |
| **Trigger** | Owner clicks "Analytics" or "Reports" button in property dashboard |
| **Main Sequence** | 1. Owner navigates to analytics page<br>2. System retrieves property data for selected time period (default: last 30 days)<br>3. System displays analytics dashboard with sections<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. **Overview KPIs** (key metrics at top)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. **Revenue Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. **Occupancy Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. **Booking Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. **Guest Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.6. **Review Analytics** section<br>4. **Overview KPIs Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Total Revenue (with % change vs previous period)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Average Daily Rate (ADR)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Occupancy Rate (%)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Total Bookings (with % change)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.5. Average Rating (with trend indicator)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.6. Cancellation Rate (%)<br>5. **Revenue Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Revenue trend chart (daily/weekly/monthly)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Revenue by room type (pie chart)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Revenue forecast (next 30 days based on confirmed bookings)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Commission breakdown (platform fees deducted)<br>6. **Occupancy Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Occupancy rate trend chart<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Occupancy by room type (bar chart)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Peak vs off-peak occupancy comparison<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Occupancy calendar heat map (color-coded by % occupied)<br>7. **Booking Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Booking source breakdown (direct, search, referral)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Booking lead time distribution (how far in advance guests book)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Average length of stay<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Cancellation rate and reasons (if provided)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. Booking conversion rate (views → bookings)<br>8. **Guest Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Guest nationality breakdown (top 10 countries)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Guest age distribution (18-25, 26-35, 36-45, 46+)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Repeat guest rate (%)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Group size distribution (solo, couple, group)<br>9. **Review Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Average rating trend chart<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Rating breakdown by category (cleanliness, location, staff, value)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Review sentiment analysis (positive, neutral, negative word cloud)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Response rate to reviews (%)<br>10. Owner adjusts time period filter<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Options: Last 7 days, 30 days, 90 days, Year to date, Custom range<br>11. System recalculates analytics for selected period<br>12. System updates all charts and metrics<br>13. Owner exports report (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. Owner clicks "Export Report" button<br>&nbsp;&nbsp;&nbsp;&nbsp;13.2. Owner selects format (PDF, Excel)<br>&nbsp;&nbsp;&nbsp;&nbsp;13.3. System generates report file<br>&nbsp;&nbsp;&nbsp;&nbsp;13.4. System downloads report to owner's device |
| **Alternative Sequences** | **Step 2**: If property has no booking history, System displays "No data available yet" with tips to get first bookings and sample dashboard preview<br><br>**Step 10**: If custom date range selected, Owner specifies start and end dates via calendar picker, System validates date range (max 365 days), calculates analytics<br><br>**Step 13**: If export fails (report too large, timeout), System displays error and suggests narrower date range or simplified report<br><br>**Comparison Mode**:<br>- Owner enables "Compare with previous period" toggle<br>- System displays side-by-side comparison<br>- All metrics show % change (increase/decrease)<br>- Charts display two lines (current vs previous period)<br>- Useful for year-over-year or month-over-month analysis<br><br>**Drill-Down View**:<br>- Owner clicks specific data point on chart<br>- System displays detailed breakdown for that segment<br>- Example: Click on "Revenue by Room Type" pie slice → shows daily revenue for that room type<br>- Owner returns to overview via breadcrumb navigation<br><br>**Competitor Benchmarking** (future enhancement):<br>- System displays anonymous aggregated data from similar properties<br>- Shows how owner's metrics compare to market average<br>- Helps owner identify competitive advantages or improvement areas<br><br>**Goal Setting**:<br>- Owner sets revenue/occupancy goals for upcoming periods<br>- System tracks progress toward goals<br>- Dashboard shows goal progress bars<br>- Alerts when goals achieved or at risk<br><br>**Scheduled Reports**:<br>- Owner configures automatic report delivery<br>- Selects frequency (daily, weekly, monthly)<br>- System sends report via email at scheduled time<br>- Useful for passive monitoring without logging in |
| **Postconditions** | **Success**: Owner views comprehensive analytics insights. Data visualized clearly. Reports exported if requested. Informed business decisions enabled.<br><br>**Failure**: Error message displayed if data retrieval fails. Partial data shown with notice. Owner can retry or contact support. |
| **Nonfunctional Requirements** | **Performance**: Analytics dashboard loads within 3 seconds for 12 months of data. Chart rendering completes within 2 seconds. Export generates within 10 seconds for standard date ranges.<br><br>**Accuracy**: All metrics calculated accurately from booking/revenue data. Occupancy rate reflects actual bed/room usage. Revenue figures match financial transactions. Ratings calculated correctly with proper weighting.<br><br>**Usability**: Charts interactive (hover for details, click to drill down). Color-coded metrics (green=positive, red=negative). Responsive layout for tablet viewing. Tooltips explain complex metrics. Visual hierarchy emphasizes important KPIs.<br><br>**Data Privacy**: Guest personal information anonymized in analytics. Only aggregated demographic data shown. Compliance with data protection regulations.<br><br>**Visualization**: Chart types appropriate for data (line for trends, bar for comparisons, pie for composition). Consistent color scheme throughout. Mobile-friendly charts (responsive, touch-enabled). |
| **Business Requirements** | BR-072: Analytics updated in near real-time (data refresh every 15 minutes)<br>BR-073: Historical data retained indefinitely for long-term trend analysis<br>BR-074: Revenue figures show net amount after platform commission deduction<br>BR-075: Occupancy calculations based on available inventory (excludes blocked dates)<br>BR-076: Guest demographics anonymized to protect privacy<br>BR-077: Export functionality available to all property owners at no additional cost |
| **Frequency of Use** | High (owners check regularly for performance monitoring, daily or weekly) |
| **Priority** | Medium |
| **Outstanding Questions** | 1. Should system provide AI-powered insights and recommendations?<br>2. Should owner be able to create custom reports with selected metrics?<br>3. Should system send automated alerts for anomalies (sudden occupancy drop)?<br>4. Should analytics include financial forecasting models?<br>5. Should system track marketing campaign effectiveness (if supported)?<br>6. Should owner be able to share analytics with investors or partners?<br>7. Should system provide API access for custom analytics integrations? |

## Related Use Cases

- **UC-12: Process Booking** — Bookings generate data displayed in analytics
- **UC-07: Submit Review** — Reviews contribute to rating analytics
- **UC-05: Book Accommodation** — Booking activity tracked in analytics

## Notes

- Analytics critical for data-driven business decisions
- KPI overview provides quick health check
- Revenue analytics essential for financial planning
- Occupancy trends inform pricing strategies
- Guest demographics guide marketing targeting
- Review analytics identify service improvement areas
- Comparison mode essential for understanding growth
- Export functionality enables external analysis and reporting
- Real-time data updates keep owners informed
- Mobile access important for on-the-go monitoring

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Owner gains business insights enabling informed decisions on pricing, marketing, and operations

✓ **C2: Avoids functional decomposition** — Complete analytics viewing sequence across multiple data dimensions with export capability

✓ **C3: Maintains black box view** — No mention of data warehouses, aggregation queries, chart libraries; only describes analytics display behavior

✓ **C4: Primary and secondary actors identified** — Primary: Owner (views analytics); Secondary: None (analytics are internal system operations)
