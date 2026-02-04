# Object Structuring: Manage Property

**Use Case**: docs/requirements/use-cases/UC-10-manage-property.md
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

**Analysis**: Owner initiates property management via dashboard, edits various sections (basic info, photos, description, amenities, location, policies), uploads/manages photos, adjusts map marker through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| PropertyManagementUI | «user interaction» | Maps from HostelOwner external user. Handles property sections display, inline editing, photo gallery management (upload, reorder, delete, captions), map interaction, autosave drafts, unsaved changes warning, preview mode. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: "Retrieves current property information" → Hostel entity
- Step 4: "Basic Information" → Hostel entity (name, address, contact, propertyType)
- Step 4.6-4.8: "Address geocoding" → Location entity
- Step 5: "Photos & Media" → Photo entity
- Step 6: "Description & Highlights" → Hostel entity (description, highlights)
- Step 7: "Amenities & Facilities" → Amenity entity (many-to-many)
- Step 8: "Location & Directions" → Location entity, Maps API data
- Step 9: "Policies & Rules" → Hostel entity (policies, checkInTime, checkOutTime)
- Step 14: "Publishes changes" → Hostel entity (if status = Active)
- Alternative: "Property Status Toggle" → Hostel entity (status: Active/Paused)

**Referenced Data**:
- **Hostel**: Property being managed
  - Attributes: hostelId, name, description, address, propertyType, contactPhone, contactEmail, policies, checkInTime, checkOutTime, status (Active/Paused)
- **Photo**: Property photos (gallery)
  - Attributes: photoId, url, caption, displayOrder, uploadDate
- **Location**: Geographic location
  - Attributes: locationId, latitude, longitude, address, city, district
- **Amenity**: Selected amenities (many-to-many)
  - Attributes: amenityId, name, category

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Hostel | Yes | ✅ | Primary entity updated across all sections |
| Photo | Yes | ✅ | Gallery management (upload, reorder, delete, captions) |
| Location | Yes | ✅ | Address geocoding, map marker position, directions |
| Amenity | Yes | ✅ | Amenities checklist (many-to-many) |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (property sections, photo gallery, map, success messages)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing PropertyManagementUI)

**Analysis**:
- Steps 3-9: Display various management sections
- Step 4.6-4.8, 8: Geocoding and map interaction via Maps API
- Step 5: Photo upload to Cloud Storage
- Step 13-14: Display success message and published changes

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| PropertyManagementUI | «user interaction» | Reused for output - displays sections, photo gallery with drag-and-drop, interactive map, validation errors, success messages, preview mode, unsaved changes warning |
| CloudStorageProxy | «proxy» | Maps from CloudStorage external system. Uploads property photos (step 5), optimizes images (compression, multiple sizes), returns photo URLs, deletes photos (step 5.10). Reused from UC-04, UC-07, UC-08, UC-09. |
| MapsAPIProxy | «proxy» | Maps from MapsAPI external system. Geocodes address (step 4.6-4.8), renders interactive map (step 8.1), displays property marker, fetches nearby places (step 8.5-8.7). Reused from UC-03, UC-04. |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Property management is stateless editing:
- Each section update is independent
- No sequential workflow states required
- Property status toggle (Active/Paused) is data attribute change, not workflow state
- Photo upload, map update, text editing are independent operations
- Autosave is background persistence, not state change
- All updates published immediately (BR-054) without approval workflow

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| PropertyUpdateCoordinator | «coordinator» | No | Coordinates property updates: retrieves current data → displays sections → handles edits → geocodes address if changed → uploads photos → saves changes → publishes immediately. Stateless orchestration. Manages autosave, unsaved changes detection. |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (address format, description length, photo requirements)
- [x] Algorithms (photo optimization, nearby places, auto-save)
- [ ] Cross-entity validation

**Identified Logic**:
- **Property validation**: Description 100-2000 characters (BR-056), minimum 5 photos before live (BR-055), address format
- **Photo optimization**: Compression, multiple sizes for responsive display, format conversion
- **Photo management**: Reorder by drag-and-drop, set primary photo, captions, deletion
- **Address geocoding**: Validate via Maps API, allow manual marker adjustment if needed
- **Autosave**: Draft save every 30 seconds to prevent data loss
- **Unsaved changes**: Detect modifications, warn before navigation
- **Status toggle**: Pause listing (hide from search, preserve bookings)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| PropertyInfoValidator | «business logic» | Validates property information: description length 100-2000 chars (BR-056), minimum 5 high-quality photos required before live (BR-055), address format, contact information format. |
| PhotoManagementService | «service» | Manages photo gallery: upload to CloudStorage via proxy, automatic optimization (compression, multiple sizes for CDN), reordering (displayOrder attribute), primary photo selection, caption management, deletion (removes from CloudStorage). Maximum 50 photos (BR-059). |
| GeocodingService | «service» | Geocodes property address via MapsAPIProxy when address changes (step 4.6-4.8). Updates Location entity with coordinates (accuracy within 10 meters per BR-057). Allows manual marker adjustment if geocoding fails. Flags for admin review if manual adjustment. |
| AutosaveService | «service» | Autosaves draft changes every 30 seconds. Detects unsaved modifications. Warns before navigation. Prevents data loss during lengthy editing. Undo/redo for text changes. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| PropertyManagementUI | «user interaction» | Boundary (I/O) | HostelOwner | Property editing interface |
| CloudStorageProxy | «proxy» | Boundary (Output) | CloudStorage | Photo uploads and optimization |
| MapsAPIProxy | «proxy» | Boundary (Output) | MapsAPI | Geocoding and map rendering |
| PropertyUpdateCoordinator | «coordinator» | Control | N/A | Property update orchestration (stateless) |
| Hostel | «entity» | Entity | Hostel | Property data |
| Photo | «entity» | Entity | Photo | Property photos |
| Location | «entity» | Entity | Location | Geographic location |
| Amenity | «entity» | Entity | Amenity | Amenities list |
| PropertyInfoValidator | «business logic» | Application Logic | N/A | Property data validation |
| PhotoManagementService | «service» | Application Logic | N/A | Photo gallery management |
| GeocodingService | «service» | Application Logic | N/A | Address geocoding |
| AutosaveService | «service» | Application Logic | N/A | Draft autosave |

**Total**: 12 objects (3 boundary, 4 entity, 1 control, 4 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actors**: CloudStorage (photos), MapsAPI (geocoding/map)
- **Boundary**: PropertyManagementUI, CloudStorageProxy, MapsAPIProxy
- **Control**: PropertyUpdateCoordinator
- **Entity**: Hostel, Photo, Location, Amenity
- **Logic**: PropertyInfoValidator, PhotoManagementService, GeocodingService, AutosaveService

**Key Interactions**:
1. HostelOwner → PropertyManagementUI: Navigate to property management
2. PropertyManagementUI → PropertyUpdateCoordinator: Request property data
3. PropertyUpdateCoordinator → Hostel: Retrieve current data
4. PropertyUpdateCoordinator → Photo: Get photo gallery
5. PropertyUpdateCoordinator → Location: Get location data
6. PropertyUpdateCoordinator → Amenity: Get current amenities
7. PropertyUpdateCoordinator → PropertyManagementUI: Display sections
8. HostelOwner → PropertyManagementUI: Edit basic info
9. PropertyUpdateCoordinator → PropertyInfoValidator: Validate (real-time)
10. HostelOwner → PropertyManagementUI: Change address
11. PropertyManagementUI → PropertyUpdateCoordinator: Geocode address
12. PropertyUpdateCoordinator → GeocodingService: Validate and geocode
13. GeocodingService → MapsAPIProxy: Request coordinates
14. MapsAPIProxy → MapsAPI: Geocode address
15. MapsAPI → MapsAPIProxy: Return coordinates
16. GeocodingService → Location: Update coordinates
17. HostelOwner → PropertyManagementUI: Upload photos
18. PropertyManagementUI → PropertyUpdateCoordinator: Process photos
19. PropertyUpdateCoordinator → PhotoManagementService: Manage upload
20. PhotoManagementService → PropertyInfoValidator: Validate photo (format/size)
21. PhotoManagementService → CloudStorageProxy: Upload photo
22. CloudStorageProxy → CloudStorage: Store and optimize
23. CloudStorage → CloudStorageProxy: Return photo URL
24. PhotoManagementService → Photo: Create photo record
25. HostelOwner → PropertyManagementUI: Reorder photos (drag-and-drop)
26. PropertyUpdateCoordinator → PhotoManagementService: Update displayOrder
27. AutosaveService: [Background] Auto-save every 30 seconds
28. AutosaveService → Hostel: Save draft changes
29. HostelOwner → PropertyManagementUI: Save changes
30. PropertyManagementUI → PropertyUpdateCoordinator: Submit updates
31. PropertyUpdateCoordinator → PropertyInfoValidator: Final validation
32. PropertyUpdateCoordinator → Hostel: Update property
33. PropertyUpdateCoordinator → PropertyManagementUI: Display success, publish immediately

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - PropertyUpdateCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (3 identified)
- [x] Boundary → External mapping (1:1):
  - PropertyManagementUI ← HostelOwner
  - CloudStorageProxy ← CloudStorage
  - MapsAPIProxy ← MapsAPI
- [x] Entities in Phase 1:
  - Hostel ✅
  - Photo ✅
  - Location ✅
  - Amenity ✅
- [x] Control justified (stateless coordinator for property updates)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (validation, photo management, geocoding, autosave)

---

## Alternative Flows Mapping

**Step 4.6-4.8 Alternative** (Geocoding fails):
- GeocodingService → MapsAPIProxy request
- MapsAPIProxy receives error from MapsAPI
- PropertyUpdateCoordinator receives failure
- PropertyManagementUI displays "Unable to locate address" warning
- Allows manual marker placement on map
- Flags for admin review

**Step 5.3 Alternative** (Photo validation fails):
- PhotoManagementService → PropertyInfoValidator validates photo
- Detects wrong format, too large (>10MB), or low resolution (<1200x800)
- PropertyUpdateCoordinator receives errors
- PropertyManagementUI displays specific error per photo
- Allows retry or skip problematic photos

**Step 5.4 Alternative** (Cloud Storage upload fails):
- PhotoManagementService → CloudStorageProxy upload
- CloudStorageProxy receives error from CloudStorage
- PhotoManagementService retries up to 3 times
- If all retries fail: PropertyManagementUI displays error
- Allows owner to retry later

**Step 11 Alternative** (Validation fails):
- PropertyInfoValidator detects errors (required fields empty, invalid data)
- PropertyUpdateCoordinator receives validation errors
- PropertyManagementUI highlights problems, prevents saving

**Property Status Toggle**:
- Owner clicks "Pause Listing"
- PropertyUpdateCoordinator → Hostel: Update status = "Paused"
- Property hidden from traveler search (UC-03)
- Existing confirmed bookings remain valid
- PropertyManagementUI displays "Paused" status
- Owner clicks "Activate Listing" to restore

**Bulk Photo Upload**:
- Owner selects up to 20 photos
- PropertyManagementUI displays upload progress per photo
- PhotoManagementService processes in parallel
- Shows success/failure status per photo
- Failed uploads retried individually

**Photo Optimization**:
- PhotoManagementService detects orientation issues
- PropertyManagementUI displays "Rotate photo?" prompt
- Owner rotates before upload
- CloudStorageProxy uploads corrected version

**Unsaved Changes Warning**:
- AutosaveService detects unsaved modifications
- Owner navigates away
- PropertyManagementUI displays "Discard changes?"
- Owner confirms or cancels navigation

---

## Business Rules

- **BR-054**: Updates published immediately, no admin approval (PropertyUpdateCoordinator enforces)
- **BR-055**: Minimum 5 high-quality photos before live (PropertyInfoValidator)
- **BR-056**: Description 100-2000 characters (PropertyInfoValidator)
- **BR-057**: Location accuracy via Maps API (GeocodingService)
- **BR-058**: Property can be paused without losing data (status toggle)
- **BR-059**: Photo captions support multiple languages (Photo entity attribute)

---

## Performance Considerations

- **Page load**: ≤ 2 seconds
- **Photo upload**: ≤ 10 seconds per photo (max 20 photos)
- **Map rendering**: ≤ 2 seconds
- **Save operation**: ≤ 3 seconds
- **Autosave**: Every 30 seconds (background)
- **Photo optimization**: Automatic compression, multiple sizes for CDN
- **Maximum photos**: 50 per property
- **Immediate publishing**: No caching delay (BR-054)

---

## Notes

- **Stateless design**: Independent section updates, immediate publishing
- **High-quality photos**: Critical for conversion (BR-055)
- **Cloud Storage**: Fast loading via CDN, automatic optimization
- **Maps API geocoding**: Accurate location for traveler search (BR-057)
- **Immediate publishing**: No admin approval for existing properties (BR-054)
- **Property pause**: Useful for maintenance, seasonal closures (BR-058)
- **Autosave**: Prevents data loss during lengthy editing
- **Preview mode**: Shows traveler view (UC-04)
- **Mobile optimization**: Many owners manage on-the-go
- **Reusability**: CloudStorageProxy, MapsAPIProxy reused across multiple UCs
- **Photo management**: Drag-and-drop reordering, primary photo selection
- **Location accuracy**: Within 10 meters via geocoding (BR-057)
