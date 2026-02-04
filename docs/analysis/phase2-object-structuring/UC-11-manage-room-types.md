# Object Structuring: Manage Room Types

**Use Case**: docs/requirements/use-cases/UC-11-manage-room-types.md
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

**Analysis**: Owner initiates room management via dashboard, creates/edits room types through forms, configures pricing, manages availability calendar through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| RoomManagementUI | «user interaction» | Maps from HostelOwner external user. Handles room types list display, creation/edit forms, bed configuration, pricing setup, availability calendar (color-coded, drag-select for bulk operations), quick actions (Edit, Duplicate, Disable, Delete). |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2-3: "Retrieves existing room types" → RoomType entity
- Step 6: "Basic Room Information" → RoomType entity (name, category, size, quantity)
- Step 7: "Bed Configuration" → Bed entity (for dorms), RoomType entity (bedConfiguration)
- Step 8: "Pricing Configuration" → RoomType entity (pricePerNight, seasonal pricing, weekend surcharge, min/max stay)
- Step 9: "Room-Specific Amenities" → Amenity entity (many-to-many), RoomType.description
- Step 10: "Availability Settings" → RoomType entity (availability status), calendar blocking
- Step 11: "Guest Restrictions" → RoomType entity (ageRestrictions, genderRestrictions, maxOccupancy)
- Step 14-15: "Creates room type and generates inventory" → RoomType entity, Bed entity (individual beds for dorms), Room units
- Alternative: "Seasonal pricing rules" → Pricing rules (may be RoomType attribute or separate entity)

**Referenced Data**:
- **RoomType**: Primary entity being managed
  - Attributes: roomTypeId, name, description, capacity, bedConfiguration, pricePerNight, totalRooms, availableRooms, roomCategory, roomSize, minStay, maxStay, ageRestrictions, genderRestrictions, maxOccupancy, status
- **Bed**: Individual beds in dorm rooms
  - Attributes: bedId, bedNumber, bedType, status, roomTypeId
- **Hostel**: Property reference (owner's property)
  - Attributes: hostelId, ownerId
- **Amenity**: Room-specific amenities (many-to-many)
  - Attributes: amenityId, name, category

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| RoomType | Yes | ✅ | Primary entity created/updated |
| Bed | Yes | ✅ | Generated for dorm room types (step 15.2) |
| Hostel | Yes | ✅ | Property reference for room types |
| Amenity | Yes | ✅ | Room-specific amenities selection |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (room list, forms, calendar, success messages)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing RoomManagementUI)

**Analysis**:
- Steps 3, 5-16: Display room list, forms, calendar, confirmation
- No external actors for output (all UI-based)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| RoomManagementUI | «user interaction» | Reused for output - displays room types list with quick actions, creation/edit forms, availability calendar (color-coded: green=available, red=blocked, yellow=limited, blue=booked), validation errors, success messages, pricing preview |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Room type management is stateless CRUD:
- Creation, editing, deletion are independent operations
- No sequential workflow states required
- Calendar availability is data updates, not state transitions
- Room status (Active/Disabled) is data attribute, not workflow state
- All operations are simple data persistence

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| RoomTypeCoordinator | «coordinator» | No | Coordinates room management: retrieves room types → displays list → handles create/edit/delete operations → validates data → generates inventory (rooms/beds) → saves changes. Stateless CRUD orchestration. |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (pricing thresholds, capacity limits, required fields)
- [x] Algorithms (capacity calculation, inventory generation, pricing rules)
- [x] Cross-entity validation (conflicting bookings when editing)

**Identified Logic**:
- **Capacity calculation**: Total guest capacity = rooms × beds per room (for dorms) or rooms × bed capacity (for private)
- **Inventory generation**: Create individual Bed records for dorms, room units for private rooms
- **Pricing validation**: Minimum price thresholds, seasonal pricing date range conflicts
- **Availability management**: Calendar blocking, bulk date operations
- **Booking conflict detection**: Prevent changes that conflict with active bookings (BR-062)
- **Overbooking prevention**: availableRooms ≤ totalRooms enforcement

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| RoomTypeValidator | «business logic» | Validates room configuration: required fields (name, category, pricing), pricing > minimum threshold, capacity > 0, date range validations for availability blocking. Prevents invalid configurations. |
| CapacityCalculator | «algorithm» | Calculates total guest capacity: For private rooms (capacity = totalRooms × bedCapacity), for dorms (capacity = totalRooms × bedsPerRoom). Updates RoomType.capacity attribute. Critical for availability checks. |
| InventoryGenerator | «service» | Generates room/bed inventory after room type creation (step 15): For private rooms, creates X room units. For dorms, creates X rooms each with Y individual Bed records. Assigns bedNumbers, initializes status = Available. |
| PricingRuleEngine | «service» | Manages pricing rules: base pricing, seasonal pricing (date range + adjusted price), weekend surcharge percentage, min/max stay requirements. Applies correct pricing for booking dates (BR-064 seasonal rules override base). |
| BookingConflictChecker | «business logic» | Checks for active/upcoming bookings when owner edits room type. Prevents capacity reduction, pricing retroactive changes if conflicts exist (BR-062, BR-063). Notifies admin if changes affect bookings. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| RoomManagementUI | «user interaction» | Boundary (I/O) | HostelOwner | Room types management interface |
| RoomTypeCoordinator | «coordinator» | Control | N/A | Room CRUD orchestration (stateless) |
| RoomType | «entity» | Entity | RoomType | Room type configuration |
| Bed | «entity» | Entity | Bed | Individual bed inventory (dorms) |
| Hostel | «entity» | Entity | Hostel | Property reference |
| Amenity | «entity» | Entity | Amenity | Room-specific amenities |
| RoomTypeValidator | «business logic» | Application Logic | N/A | Room configuration validation |
| CapacityCalculator | «algorithm» | Application Logic | N/A | Guest capacity calculation |
| InventoryGenerator | «service» | Application Logic | N/A | Room/bed inventory generation |
| PricingRuleEngine | «service» | Application Logic | N/A | Pricing rules management |
| BookingConflictChecker | «business logic» | Application Logic | N/A | Booking conflict detection |

**Total**: 11 objects (1 boundary, 4 entity, 1 control, 5 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actors**: None
- **Boundary**: RoomManagementUI
- **Control**: RoomTypeCoordinator
- **Entity**: RoomType, Bed, Hostel, Amenity
- **Logic**: RoomTypeValidator, CapacityCalculator, InventoryGenerator, PricingRuleEngine, BookingConflictChecker

**Key Interactions (Create Room Type)**:
1. HostelOwner → RoomManagementUI: Click "Add Room Type"
2. RoomManagementUI → RoomTypeCoordinator: Start creation
3. RoomTypeCoordinator → RoomManagementUI: Display form
4. HostelOwner → RoomManagementUI: Enter room info (Steps 6-11)
5. HostelOwner → RoomManagementUI: Save room type
6. RoomManagementUI → RoomTypeCoordinator: Submit data
7. RoomTypeCoordinator → RoomTypeValidator: Validate configuration
8. RoomTypeValidator: Check required fields, pricing threshold
9. RoomTypeCoordinator → CapacityCalculator: Calculate capacity
10. CapacityCalculator: totalRooms × bedsPerRoom (dorm) or bedCapacity (private)
11. RoomTypeCoordinator → RoomType: Create record
12. RoomTypeCoordinator → Amenity: Link selected amenities
13. RoomTypeCoordinator → InventoryGenerator: Generate inventory
14. If dorm: InventoryGenerator → Bed: Create individual beds
15. InventoryGenerator: Assign bedNumbers, set status = Available
16. RoomTypeCoordinator → PricingRuleEngine: Save pricing rules
17. RoomTypeCoordinator → RoomManagementUI: Display success

**Edit Existing Room Type**:
1. HostelOwner → RoomManagementUI: Click "Edit"
2. RoomTypeCoordinator → RoomType: Retrieve current data
3. RoomManagementUI: Display edit form
4. HostelOwner: Modify information
5. RoomTypeCoordinator → BookingConflictChecker: Check active bookings
6. BookingConflictChecker → RoomType: Query booking conflicts
7. If conflicts exist and capacity reduced: Prevent save, display error
8. If no conflicts: RoomTypeCoordinator saves changes (BR-063 future bookings only)

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

No statechart needed - RoomTypeCoordinator is stateless.

---

## Validation

- [x] At least 1 boundary object (1 identified)
- [x] Boundary → External mapping (1:1):
  - RoomManagementUI ← HostelOwner
- [x] Entities in Phase 1:
  - RoomType ✅
  - Bed ✅
  - Hostel ✅
  - Amenity ✅
- [x] Control justified (stateless coordinator for CRUD operations)
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified (validation, capacity calculation, inventory generation, pricing, conflict checking)

---

## Alternative Flows Mapping

**Step 13 Alternative** (Validation fails):
- RoomTypeValidator detects errors (missing fields, pricing below threshold)
- RoomTypeCoordinator receives validation errors
- RoomManagementUI displays specific error messages, prevents saving

**Edit Existing Room Type**:
- BookingConflictChecker queries active/upcoming bookings
- If capacity reduction conflicts with bookings: Error, prevent save
- If pricing change: Apply to future bookings only (BR-063)
- Admin notification sent if changes affect bookings (BR-062)

**Duplicate Room Type**:
- RoomTypeCoordinator copies RoomType data
- Appends "(Copy)" to name
- Owner modifies, saves as new room type
- InventoryGenerator creates new inventory

**Disable Room Type**:
- RoomTypeCoordinator → RoomType: Update status = "Disabled"
- Hidden from traveler search (UC-03)
- Existing bookings remain valid
- Visible in owner dashboard (BR-065)

**Delete Room Type**:
- BookingConflictChecker → RoomType: Check active/upcoming bookings
- If bookings exist: Error "Cannot delete room type with active bookings"
- If no bookings: Soft delete (mark as deleted, preserve data)

**Bulk Availability Update**:
- HostelOwner selects date range on calendar (drag-select)
- Marks as "Available" or "Blocked"
- RoomTypeCoordinator updates all selected dates
- Useful for seasonal closures, maintenance

**Seasonal Pricing Rules**:
- Owner specifies date range, adjusted price
- PricingRuleEngine validates no date conflicts
- Saves pricing rule
- BR-064: Seasonal pricing overrides base pricing
- Calendar displays effective price per date

---

## Business Rules

- **BR-060**: Minimum 1 active room type for bookings (RoomTypeValidator enforces)
- **BR-061**: Pricing per bed (dorm) or per room (private) (CapacityCalculator logic)
- **BR-062**: Capacity changes notify admin if bookings exist (BookingConflictChecker)
- **BR-063**: Pricing changes apply to future bookings only (PricingRuleEngine)
- **BR-064**: Seasonal pricing overrides base pricing (PricingRuleEngine)
- **BR-065**: Disabled rooms hidden from search (RoomTypeCoordinator filtering)

---

## Notes

- **Stateless CRUD**: Create, read, update, delete operations without workflow
- **Bed-level inventory**: Unique hostel requirement (vs hotel room-level)
- **Flexible pricing**: Base + seasonal + weekend surcharge
- **Visual calendar**: Color-coded, drag-select for bulk operations
- **Duplicate feature**: Saves time for similar rooms
- **Gender-specific dorms**: Common hostel industry requirement
- **Soft delete**: Preserves booking history
- **Inventory generation**: Automatic Bed record creation for dorms
- **Overbooking prevention**: availableRooms ≤ totalRooms enforced
- **Booking conflict prevention**: Critical for data integrity
