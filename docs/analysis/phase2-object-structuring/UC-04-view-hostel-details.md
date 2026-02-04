# Object Structuring: View Hostel Details

**Use Case**: docs/requirements/use-cases/UC-04-view-hostel-details.md
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

**Analysis**: Traveler initiates by clicking hostel card from search results or accessing via direct link. Interacts with photo gallery, map, and reviews.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| HostelDetailsUI | «user interaction» | Maps from Traveler external user. Handles details page display, photo gallery navigation, map interaction, review filtering, "Book Now" action. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: "System retrieves hostel details" → Hostel entity
- Step 5.1: "Photo gallery" → Photo entity
- Step 5.3: "Location address with map" → Location entity
- Step 5.4: "Lists amenities" → Amenity entity
- Step 5.6: "Available room types with prices" → RoomType entity
- Step 5.8: "Guest reviews with ratings" → Review entity
- Step 8: "Availability" → RoomType entity (availableRooms), Booking entity (implicit)
- Precondition: "Hostel exists and is active" → Hostel entity

**Referenced Data**:
- **Hostel**: Primary entity displayed
  - Attributes: hostelId, name, description, rating, status
- **Photo**: Gallery images
  - Attributes: url, caption, displayOrder
- **Location**: Address and map
  - Attributes: address, city, latitude, longitude
- **Amenity**: Facility list
  - Attributes: name, category
- **RoomType**: Room options and pricing
  - Attributes: name, capacity, bedConfiguration, pricePerNight, availableRooms
- **Review**: Guest feedback
  - Attributes: rating, comment, reviewDate, status
- **User**: Reviewer names (implicit via Review relationship)

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Hostel | Yes | ✅ | Primary entity retrieved (step 2), all details displayed (step 5) |
| Photo | Yes | ✅ | Gallery display (step 5.1, step 6) |
| Location | Yes | ✅ | Address and map (step 5.3, step 7) |
| Amenity | Yes | ✅ | Amenities list (step 5.4) |
| RoomType | Yes | ✅ | Room options, pricing, availability (step 5.6, step 8) |
| Review | Yes | ✅ | Guest reviews display (step 5.8, step 9) |
| User | Yes | ✅ | Reviewer information (via Review relationship) |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (details page, photo gallery, reviews)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing HostelDetailsUI)

**Analysis**:
- Step 3: System requests photos from Cloud Storage
- Step 4: System requests map from Maps API
- Step 5-9: All information displayed via web interface
- Two external systems provide content

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| HostelDetailsUI | «user interaction» | Reused for output - displays hostel details, photo gallery (full-screen, navigation), interactive map, amenities, room types, policies, reviews, "Book Now" button |
| CloudStorageProxy | «proxy» | Maps from CloudStorage external system. Retrieves property photos (step 3). Handles progressive loading and full-screen gallery. |
| MapsAPIProxy | «proxy» | Maps from MapsAPI external system. Renders interactive map (step 4, 7), displays points of interest markers, supports zoom/pan. |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Details viewing is stateless:
- No sequential state transitions
- Gallery navigation and map interaction are UI state, not business state
- Review filtering is data transformation
- Page displays information without state-dependent behavior
- All operations are read-only data retrieval

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| DetailsCoordinator | «coordinator» | No | Coordinates content retrieval: hostel data → photos → map → reviews. Stateless orchestration for page assembly. Handles graceful degradation if external services unavailable. |

---

## 5. Application Logic

**Complex Rules**:
- [ ] Multi-condition validation
- [x] Algorithms (review filtering, rating calculation)
- [ ] Cross-entity validation

**Identified Logic**:
- **Review filtering**: Filter by rating threshold (5-star, 4-star, etc.)
- **Review sorting**: Sort by most recent, highest rating, most helpful
- **Review pagination**: 10 reviews per page
- **Price calculation**: If dates selected, calculate total for stay duration
- **Progressive image loading**: Lazy load below-fold photos
- **Availability display**: Real-time room availability check

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| ReviewManager | «service» | Manages review display: filtering by rating, sorting (recent/rating/helpful), pagination (10 per page). Only shows verified guest reviews (status = Published). |
| PriceCalculator | «business logic» | Calculates total price if dates selected: (pricePerNight × nights). Reused from UC-03. Displays per-night and total pricing. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| HostelDetailsUI | «user interaction» | Boundary (I/O) | Traveler | Details page with gallery and map |
| CloudStorageProxy | «proxy» | Boundary (Output) | CloudStorage | Photo retrieval |
| MapsAPIProxy | «proxy» | Boundary (Output) | MapsAPI | Interactive map rendering |
| DetailsCoordinator | «coordinator» | Control | N/A | Content retrieval orchestration (stateless) |
| Hostel | «entity» | Entity | Hostel | Property details |
| Photo | «entity» | Entity | Photo | Gallery images |
| Location | «entity» | Entity | Location | Address and coordinates |
| Amenity | «entity» | Entity | Amenity | Facilities list |
| RoomType | «entity» | Entity | RoomType | Room options and pricing |
| Review | «entity» | Entity | Review | Guest feedback |
| User | «entity» | Entity | User | Reviewer information |
| ReviewManager | «service» | Application Logic | N/A | Review filtering and pagination |
| PriceCalculator | «business logic» | Application Logic | N/A | Price calculation (reused) |

**Total**: 13 objects (3 boundary, 7 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: Traveler
- **Secondary Actors**: CloudStorage, MapsAPI
- **Boundary**: HostelDetailsUI, CloudStorageProxy, MapsAPIProxy
- **Control**: DetailsCoordinator
- **Entity**: Hostel, Photo, Location, Amenity, RoomType, Review, User
- **Logic**: ReviewManager, PriceCalculator

**Key Interactions**:
1. Traveler → HostelDetailsUI: Click hostel card
2. HostelDetailsUI → DetailsCoordinator: Request hostel details
3. DetailsCoordinator → Hostel: Retrieve property data
4. DetailsCoordinator → Photo: Get photo gallery
5. DetailsCoordinator → CloudStorageProxy: Request photo URLs
6. CloudStorageProxy → CloudStorage: Fetch images
7. DetailsCoordinator → Location: Get address and coordinates
8. DetailsCoordinator → MapsAPIProxy: Request map
9. MapsAPIProxy → MapsAPI: Render interactive map
10. DetailsCoordinator → Amenity: Get amenities list
11. DetailsCoordinator → RoomType: Get room types and pricing
12. DetailsCoordinator → Review: Get guest reviews
13. DetailsCoordinator → ReviewManager: Process reviews
14. ReviewManager → User: Get reviewer names
15. DetailsCoordinator → HostelDetailsUI: Display all content
16. Traveler → HostelDetailsUI: Navigate photo gallery
17. Traveler → HostelDetailsUI: Interact with map
18. Traveler → HostelDetailsUI: Filter reviews
19. HostelDetailsUI → ReviewManager: Apply filter
20. ReviewManager → Review: Filter by rating
21. Traveler → HostelDetailsUI: Click "Book Now"

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - DetailsCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (3 identified)
- [x] Boundary → External mapping (1:1):
  - HostelDetailsUI ← Traveler
  - CloudStorageProxy ← CloudStorage
  - MapsAPIProxy ← MapsAPI
- [x] Entities in Phase 1:
  - Hostel ✅
  - Photo ✅
  - Location ✅
  - Amenity ✅
  - RoomType ✅
  - Review ✅
  - User ✅
- [x] Control justified (stateless coordinator for content assembly)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (review management, price calculation)

---

## Alternative Flows Mapping

**Step 2 Alternative** (Hostel not found):
- DetailsCoordinator queries Hostel entity
- No record or status ≠ Active
- HostelDetailsUI displays "Property not available"
- Redirect to search page

**Step 3 Alternative** (Cloud Storage unavailable):
- CloudStorageProxy timeout or error
- DetailsCoordinator receives error
- HostelDetailsUI displays placeholder images
- Continues with text content

**Step 4 Alternative** (Maps API unavailable):
- MapsAPIProxy timeout or error
- DetailsCoordinator receives error
- HostelDetailsUI displays static location info (address, coordinates text)
- No interactive map

**Step 8 Alternative** (No availability):
- If dates selected: DetailsCoordinator checks RoomType.availableRooms
- All rooms booked: HostelDetailsUI displays "No availability for selected dates"
- Offers date adjustment option

**Step 9 Alternative** (No reviews):
- ReviewManager queries Review entity
- Zero published reviews found
- HostelDetailsUI displays "No reviews yet. Be the first to review!"

**Pre-sequence Alternative** (No dates selected):
- Traveler accesses details without search context
- HostelDetailsUI prompts for check-in/check-out dates
- Required before displaying availability and total pricing
- Shows per-night pricing only

---

## Performance Considerations

- **Page load**: ≤ 2 seconds
- **Photos**: Progressive/lazy loading (below-fold images)
- **Map rendering**: ≤ 1.5 seconds
- **Review pagination**: 10 per page
- **Image resolution**: 1200px width for desktop
- **Full-screen gallery**: High-res on demand
- **Graceful degradation**: Functions without Cloud Storage or Maps API
- **Mobile optimization**: Touch-friendly gallery swipes

---

## Notes

- **High-quality photos**: Critical for conversion rates
- **Interactive map**: Builds trust, visualizes location context
- **Guest reviews**: Essential credibility factor
- **Mobile-first**: High mobile traffic for travel research
- **Performance impact**: Page load speed affects bounce rate
- **Sticky "Book Now"**: Visible during scroll for conversion
- **Reusability**: PriceCalculator reused from UC-03
- **Read-only**: No data modification, optimized for caching
- **Progressive enhancement**: Core content works without external services
