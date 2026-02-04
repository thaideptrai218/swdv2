# Object Structuring: Search Hostels

**Use Case**: docs/requirements/use-cases/UC-03-search-hostels.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Primary Actor**: Traveler
**External Class (Phase 1)**: Traveler («external user»)

**Interface Type**:

- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Analysis**: Traveler initiates search by entering criteria (location, dates, guests) through web search interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| SearchUI | «user interaction» | Maps from Traveler external user. Handles search form display, criteria input, autocomplete suggestions, filter/sort controls. |

---

## 2. Entity Objects (Data)

**Data References**:

- Step 4: "System retrieves hostels within search area" → Hostel entity
- Step 5: "System checks availability for specified dates" → RoomType, Booking entities
- Step 6: "System calculates prices" → RoomType entity (pricePerNight)
- Step 7.3: "Shows name, photo, price, distance, rating, amenities" → Hostel, Photo, Amenity entities
- Step 8: "Traveler filters by amenities" → Amenity entity
- Step 2-3: "Location query" → Location entity
- Precondition: "At least one hostel registered and active" → Hostel entity

**Referenced Data**:

- **Hostel**: Primary search target
    - Attributes: hostelId, name, description, city, rating, status
- **RoomType**: Availability and pricing
    - Attributes: pricePerNight, availableRooms
- **Location**: Geographic search
    - Attributes: latitude, longitude, city, district
- **Photo**: Visual display
    - Attributes: url, displayOrder
- **Amenity**: Filtering
    - Attributes: name, category
- **Booking**: Availability check (implicit query for date conflicts)

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Hostel | Yes | ✅ | Primary search entity (step 4), filtered and displayed (step 7) |
| RoomType | Yes | ✅ | Availability check (step 5), pricing (step 6) |
| Location | Yes | ✅ | Geographic search (step 4), distance calculation (step 7.3) |
| Photo | Yes | ✅ | Thumbnail display (step 7.3) |
| Amenity | Yes | ✅ | Filtering (step 8.2) |
| Booking | Yes | ✅ | Implicit - date conflict check for availability (step 5) |

---

## 3. Boundary Objects (Output)

**Output Type**:

- [x] Display/Screen (search results list, map view, filters)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing SearchUI for results)

**Analysis**:

- Step 7: System displays search results (list + map view)
- Step 2-3: System interacts with Maps API for geocoding and map rendering
- All output displayed through web interface

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| SearchUI | «user interaction» | Reused for output - displays search results list, map view with markers, filters panel, sort controls, pagination, error messages |
| MapsAPIProxy | «proxy» | Maps from Maps API external system. Handles location geocoding (step 2-3), map rendering (step 7.2), distance calculations (step 7.3), points of interest markers. |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Search process is stateless:

- Each search executes independently
- No sequential state transitions required
- Filtering and sorting are data transformations, not state changes
- No timeout or state-dependent behavior
- All operations are request-response style

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| SearchCoordinator | «coordinator» | No | Coordinates search workflow: geocoding → hostel retrieval → availability check → price calculation → results display. Stateless orchestration without state-dependent behavior. Manages filter/sort operations. |

---

## 5. Application Logic

**Complex Rules**:

- [x] Multi-condition validation (date validation, location resolution)
- [x] Algorithms (search radius expansion, distance calculation, filtering, sorting)
- [x] Cross-entity validation (availability check across bookings)

**Identified Logic**:

- **Search algorithm**: Geographic radius search, availability filtering, price calculation
- **Distance calculation**: Calculate distance from search location to each hostel
- **Availability logic**: Check room availability against existing bookings for date range
- **Filtering logic**: Multi-criteria filtering (price range, amenities, room type, rating)
- **Sorting logic**: Multiple sort options (price, distance, rating, popularity)
- **Radius expansion**: Auto-expand from 50km to 100km if no results

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| SearchEngine | «algorithm» | Implements search algorithm: geographic radius search (50km, expands to 100km), availability filtering, result ranking. Optimized for performance (3-second target). |
| AvailabilityCalculator | «business logic» | Checks room availability by querying Booking entity for date conflicts. Real-time availability updates (5-minute freshness). Accounts for check-in/check-out date overlaps. |
| PriceCalculator | «business logic» | Calculates total price for stay duration: (pricePerNight × number of nights) + taxes + fees. Supports currency conversion if needed. |
| ResultsFilterSort | «service» | Manages filtering (price range, amenities, room type, rating threshold) and sorting (price, distance, rating, popularity). Handles pagination (20 per page). |

---

## Object Summary

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

## Phase 3 Preparation

**Objects for Communication Diagram**:

- **Primary Actor**: Traveler
- **Secondary Actor**: MapsAPI
- **Boundary**: SearchUI, MapsAPIProxy
- **Control**: SearchCoordinator
- **Entity**: Hostel, RoomType, Location, Photo, Amenity, Booking
- **Logic**: SearchEngine, AvailabilityCalculator, PriceCalculator, ResultsFilterSort

**Key Interactions**:

1. Traveler → SearchUI: Enter search criteria (location, dates, guests)
2. SearchUI → SearchCoordinator: Process search request
3. SearchCoordinator → MapsAPIProxy: Geocode location
4. MapsAPIProxy → MapsAPI: Request coordinates
5. MapsAPI → MapsAPIProxy: Return location data
6. SearchCoordinator → SearchEngine: Execute search with coordinates
7. SearchEngine → Location: Query hostels within radius
8. SearchEngine → Hostel: Retrieve hostel data
9. SearchCoordinator → AvailabilityCalculator: Check availability
10. AvailabilityCalculator → RoomType: Get room data
11. AvailabilityCalculator → Booking: Check date conflicts
12. SearchCoordinator → PriceCalculator: Calculate prices
13. PriceCalculator → RoomType: Get price per night
14. SearchCoordinator → Photo: Retrieve thumbnails
15. SearchCoordinator → SearchUI: Display results (list + map)
16. MapsAPIProxy → MapsAPI: Render map with markers
17. Traveler → SearchUI: Apply filters
18. SearchUI → SearchCoordinator: Update filters
19. SearchCoordinator → ResultsFilterSort: Filter results
20. ResultsFilterSort → Amenity: Match amenity criteria
21. SearchCoordinator → SearchUI: Display filtered results

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - SearchCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
    - SearchUI ← Traveler
    - MapsAPIProxy ← MapsAPI
- [x] Entities in Phase 1:
    - Hostel ✅
    - RoomType ✅
    - Location ✅
    - Photo ✅
    - Amenity ✅
    - Booking ✅
- [x] Control justified (stateless coordinator for workflow management)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (complex search algorithm, availability, pricing, filtering)

---

## Alternative Flows Mapping

**Step 3 Alternative** (Location not found):

- MapsAPIProxy receives no results from MapsAPI
- SearchCoordinator receives error
- SearchUI displays "Location not found" with suggestions

**Step 4 Alternative** (No hostels in 50km):

- SearchEngine finds zero results in initial radius
- SearchEngine automatically expands radius to 100km
- SearchUI displays message "No results in [location]. Showing results within 100km"

**Step 5 Alternative** (No availability):

- AvailabilityCalculator finds all rooms booked
- SearchEngine filters out unavailable hostels
- If zero results: SearchUI displays "No availability for selected dates" with date adjustment option

**Step 9 Alternative** (Filters return zero):

- ResultsFilterSort produces empty result set
- SearchUI displays "No results match your filters" with suggestion to remove filters

**Date Validation Alternatives** (Step 1):

- **Check-out ≤ check-in**: SearchUI displays error, prevents search
- **Check-in in past**: SearchUI displays error, resets to today

**Step 3 Alternative** (Maps API unavailable):

- MapsAPIProxy timeout or error
- SearchCoordinator falls back to text-based search using Location.city
- SearchUI displays results without map view, shows text-only location info

---

## Performance Considerations

- **Search results**: ≤ 3 seconds for 100 results
- **Map rendering**: ≤ 2 seconds
- **Filter/sort**: ≤ 1 second
- **Pagination**: 20 results per page
- **Concurrent queries**: 1,000 simultaneous searches supported
- **Availability freshness**: 5-minute update window
- **Graceful degradation**: Text search if Maps API unavailable

---

## Notes

- **Maps API critical**: Geocoding and map visualization enhance UX
- **Real-time availability**: AvailabilityCalculator ensures bookable properties only
- **Auto-expansion**: 50km → 100km radius prevents "no results" frustration
- **Stateless design**: Each search independent, supports horizontal scaling
- **Mobile optimization**: SearchUI responsive, pagination essential
- **Reusability**: PriceCalculator and AvailabilityCalculator reusable in UC-04, UC-05
- **Performance target**: 3-second search directly impacts conversion rate
