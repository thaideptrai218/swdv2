# UC-03: Search Hostels

| Field | Description |
|-------|-------------|
| **Use Case Name** | Search Hostels |
| **Use Case ID** | UC-03 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Traveler searches for hostels by entering location and dates. System retrieves available hostels matching criteria and displays results with map view. Traveler can filter and sort results to find suitable accommodation. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A3: Traveler. Secondary: A8: Maps API |
| **Preconditions** | Traveler has internet access. System is operational. At least one hostel property registered and active in system. Maps API service is available. |
| **Trigger** | Traveler enters location in search box or clicks "Search Hostels" on homepage |
| **Main Sequence** | 1. Traveler enters search criteria<br>&nbsp;&nbsp;&nbsp;&nbsp;1.1. Traveler enters destination (city name, address, or point of interest)<br>&nbsp;&nbsp;&nbsp;&nbsp;1.2. Traveler selects check-in date<br>&nbsp;&nbsp;&nbsp;&nbsp;1.3. Traveler selects check-out date<br>&nbsp;&nbsp;&nbsp;&nbsp;1.4. Traveler specifies number of guests (optional)<br>2. System sends location query to Maps API<br>3. Maps API returns coordinates and location details<br>4. System retrieves hostels within search area<br>5. System checks availability for specified dates<br>6. System calculates prices for stay duration<br>7. System displays search results<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. System shows list of available hostels<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. System displays map view with hostel markers from Maps API<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. For each hostel, System shows: name, thumbnail photo, price per night, total price, distance from search location, rating, amenities summary<br>8. Traveler applies filters (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Traveler filters by price range<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Traveler filters by amenities (WiFi, parking, breakfast, etc.)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Traveler filters by property type (private room, dorm, entire place)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Traveler filters by guest rating threshold<br>9. System updates search results based on filters<br>10. Traveler sorts results (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Sort by: price (low to high, high to low), distance, rating, popularity<br>11. System reorders results based on selected sort<br>12. Traveler clicks hostel card to view details (continues to UC-04) |
| **Alternative Sequences** | **Step 3**: If Maps API cannot resolve location, System displays "Location not found" message and suggests similar location names or prompts traveler to refine search<br><br>**Step 4**: If no hostels exist in search area (50km radius), System expands search radius to 100km and displays message "No results in [location]. Showing results within 100km"<br><br>**Step 5**: If no hostels available for specified dates, System displays "No availability for selected dates" and offers option to adjust dates or view fully booked properties<br><br>**Step 9**: If filtered results return zero matches, System displays "No results match your filters" and suggests removing some filters<br><br>**Step 1**: If check-out date is before or equal to check-in date, System displays error "Check-out date must be after check-in date" and prevents search<br><br>**Step 1**: If check-in date is in the past, System displays error "Check-in date cannot be in the past" and resets to today's date<br><br>**Step 3**: If Maps API service unavailable, System performs text-based search using stored location data and displays results without map view |
| **Postconditions** | **Success**: Search results displayed showing available hostels for specified criteria. Traveler can view details of individual hostels. Search parameters preserved for refinement.<br><br>**Failure**: Error message displayed with guidance. Traveler prompted to adjust search criteria. |
| **Nonfunctional Requirements** | **Performance**: Search results display within 3 seconds for queries returning up to 100 results. Map renders within 2 seconds. Filter/sort operations complete within 1 second.<br><br>**Accuracy**: Location geocoding accuracy within 100 meters. Availability data reflects real-time booking status (updated within 5 minutes). Price calculations accurate including all applicable fees.<br><br>**Usability**: Search interface intuitive with auto-complete suggestions for destinations. Date picker mobile-friendly. Map view zoomable and interactive. Filter panel accessible and clear. Results pagination for large result sets (20 per page).<br><br>**Scalability**: System handles 1,000 concurrent search queries. Results support pagination for cities with 500+ hostels.<br><br>**Availability**: Search service available 24/7 with 99.9% uptime. Graceful degradation if Maps API unavailable (text-based search without map). |
| **Business Requirements** | BR-011: Only active and verified hostel properties appear in search results<br>BR-012: Prices displayed include all base fees but exclude payment gateway fees<br>BR-013: Hostels without availability for specified dates excluded from results<br>BR-014: Search radius expands automatically if initial radius yields zero results<br>BR-015: Featured/sponsored hostels may appear at top of results (clearly marked) |
| **Frequency of Use** | Very High (primary entry point for travelers, hundreds of searches daily) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support "flexible dates" search showing prices across multiple date ranges?<br>2. Should map view display price labels on hostel markers?<br>3. What is the maximum search radius before system shows "no results"?<br>4. Should system save recent searches for quick access?<br>5. Should search results include "sponsored" or "featured" hostels with priority placement?<br>6. Should system support searching by neighborhood or district within a city?<br>7. Should traveler be required to specify dates or allow browsing without dates? |

## Related Use Cases

- **UC-04: View Hostel Details** — Traveler clicks search result to view full property details
- **UC-05: Book Accommodation** — After finding suitable hostel via search, traveler proceeds to booking

## Notes

- Maps API integration critical for user experience and location accuracy
- Real-time availability check ensures travelers see only bookable properties
- Filter and sort functionality essential for conversion optimization
- Mobile-responsive design prioritized given high mobile usage for travel planning
- Search performance directly impacts bounce rate and booking conversion
- Auto-complete and location suggestions reduce search friction

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Traveler receives list of available hostels matching criteria with map visualization

✓ **C2: Avoids functional decomposition** — Complete search sequence from criteria entry to results display with filtering/sorting

✓ **C3: Maintains black box view** — No mention of search indexes, database queries, Elasticsearch; only describes search behavior from external perspective

✓ **C4: Primary and secondary actors identified** — Primary: Traveler (initiates search); Secondary: Maps API (provides location services)
