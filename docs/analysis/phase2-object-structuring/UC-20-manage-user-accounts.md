# Object Structuring: Manage User Accounts

**Use Case**: docs/requirements/use-cases/UC-20-manage-user-accounts.md
**Generated**: 2026-02-04

---

## 1. Boundary Objects (Input)

**Actor**: Platform Administrator
**External Class (Phase 1)**: PlatformAdministrator («external user»)

**Interface Type**:
- [x] Standard I/O
- [ ] Specialized devices
- [ ] External system
- [ ] Timer

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| AdminInteraction | «user interaction» | Maps from PlatformAdministrator - handles user list, search, account actions |

**Secondary Actor**: Email Service
**External Class (Phase 1)**: EmailService («external system»)

| Object | Stereotype | Justification |
|--------|------------|---------------|
| EmailServiceProxy | «proxy» | Maps from EmailService - sends notifications to users |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2: System displays user list with filters
- Step 4: System displays user profile (account details, properties, booking history, status, login activity)
- Step 5: Admin performs actions (suspend, reset password, verify property, delete)
- Step 6: System logs activity
- Postconditions: User account updated, action logged

**Objects**:
| Entity | In Phase 1? | Status |
|--------|-------------|--------|
| User | Yes | ✅ |
| Property | Yes | ✅ |
| Booking | Yes | ✅ |
| AdminAuditLog | No - Add | ❌ Compliance audit trail |
| LoginActivity | No - Add | ❌ Security tracking |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen
- [ ] Printer
- [ ] External device
- [x] Same as input

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| AdminInteraction | «user interaction» | Reused for display user list, profiles, confirmations |

---

## 4. Control Objects

**State-Dependent?**: [ ] YES / [x] NO

**Justification**: Admin actions are discrete operations without state dependencies. Each action (suspend, reset, delete) is independent. No temporal workflow or modes.

| Object | Stereotype | Phase 4? |
|--------|------------|----------|
| UserManagementCoordinator | «coordinator» | No |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation
- [x] Algorithms
- [x] Cross-entity validation

| Object | Stereotype | Justification |
|--------|------------|---------------|
| AccountAnonymizationService | «service» | GDPR compliance - anonymizes PII while retaining transactional records for legal/accounting |
| PasswordResetService | «business logic» | Generates temporary passwords, sets expiration (24h), enforces change on next login |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link |
|--------|------------|----------|--------------|
| AdminInteraction | «user interaction» | Boundary | PlatformAdministrator |
| EmailServiceProxy | «proxy» | Boundary | EmailService |
| User | «entity» | Entity | User |
| Property | «entity» | Entity | Property |
| Booking | «entity» | Entity | Booking |
| AdminAuditLog | «entity» | Entity | (New) |
| LoginActivity | «entity» | Entity | (New) |
| UserManagementCoordinator | «coordinator» | Control | N/A |
| AccountAnonymizationService | «service» | Logic | N/A |
| PasswordResetService | «business logic» | Logic | N/A |

**Total**: 10 objects (2 boundary, 5 entity, 1 control, 2 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- Actors: PlatformAdministrator, EmailService
- Boundary: AdminInteraction, EmailServiceProxy
- Control: UserManagementCoordinator
- Entity: User, Property, Booking, AdminAuditLog, LoginActivity
- Logic: AccountAnonymizationService, PasswordResetService

---

## Phase 4 Requirements

**State-Dependent Controls**: 0

(No statechart needed - coordinator only)

---

## Validation

- [x] At least 1 boundary object
- [x] Boundary → External mapping (1:1)
- [x] Entities in Phase 1 (3 existing, 2 new to add)
- [x] Control justified
- [x] State-dependent flagged (N/A - stateless)
- [x] Logic justified

**Notes**: Need to update Phase 1 to add: AdminAuditLog, LoginActivity entities. GDPR compliance critical for deletion.
