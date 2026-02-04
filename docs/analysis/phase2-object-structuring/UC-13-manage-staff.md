# Object Structuring: Manage Staff

**Use Case**: docs/requirements/use-cases/UC-13-manage-staff.md
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

**Analysis**: Owner initiates via dashboard, invites staff through form, configures permissions, manages active staff (edit, suspend, remove) through web interface. Staff member receives invitation email and registers via link.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| StaffManagementUI | «user interaction» | Maps from HostelOwner external user. Handles staff list table display, invitation form, permission configuration checkboxes, edit/suspend/remove actions, confirmation dialogs, staff registration form (for invited staff). |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2-3: "Retrieves current staff list" → Staff entity
- Step 6: "Staff information" → Staff entity (email, fullName, role, permissions)
- Step 9: "Creates pending staff invitation" → Staff entity (status = Pending)
- Step 10-11: "Sends invitation email" → Notification entity
- Step 13-17: "Staff registration/login" → User entity (staff account), Staff entity (association with property)
- Step 18: "Applies role and permissions" → Staff entity (permissions configuration)
- Alternative: "Edit permissions" → Staff entity (permission updates)
- Alternative: "Suspend/Remove staff" → Staff entity (status updates, association removal)

**Referenced Data**:
- **Staff**: Staff assignment and permissions
  - Attributes: staffId, userId, hostelId, role, permissions, status (Active/Pending/Suspended), assignedDate
- **User**: Staff account (separate from traveler/owner accounts)
  - Attributes: userId, email, fullName, role
- **Notification**: Invitation and notification emails
  - Attributes: notificationId, recipientId, type, content, sentDate
- **Hostel**: Property reference
  - Attributes: hostelId, ownerId

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Staff | Yes | ✅ | Primary entity managed (invitation, permissions, status) |
| User | Yes | ✅ | Staff account (existing or new) |
| Notification | Yes | ✅ | Invitation emails, permission change notifications, suspension/removal notices |
| Hostel | Yes | ✅ | Property reference for staff assignment |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (staff list, forms, confirmation messages)
- [ ] Printer
- [ ] External device
- [x] Same as input (reusing StaffManagementUI)

**Analysis**:
- Steps 3, 11, 21: Display staff list, confirmation messages
- Steps 10, 19: Send invitation and confirmation emails via Email Service
- All screen output through same UI boundary

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| StaffManagementUI | «user interaction» | Reused for output - displays staff table, invitation form, permission editor, confirmation dialogs, success messages, registration form for staff |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends staff invitation (step 10) with registration link and 7-day expiration, confirmation emails (step 19), permission change notifications, suspension/removal notices. Reused from multiple UCs. |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Staff invitation has state-dependent workflow:
- **Pending** state: Invitation sent, awaiting staff registration (7-day expiration timer)
- **Expired** state: Invitation token expired after 7 days (BR-067)
- **Active** state: Staff registered and has property access
- **Suspended** state: Access temporarily revoked
- **Removed** state: Staff association deleted
- Token validation depends on state (used/expired/valid)
- Time-based expiration creates state dependency

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| StaffManagementControl | «state-dependent control» | ⚠️ Yes | Orchestrates staff workflow through states: invitation creation → pending (7-day window) → staff registration → active / expired. Manages invitation token validation (used/expired/valid). Handles permission updates, suspension (access revocation), removal (association deletion). Time-based state transitions for invitation expiration (BR-067). |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (email format, duplicate staff check, permission configuration)
- [x] Algorithms (invitation token generation, expiration checking)
- [x] Cross-entity validation (email uniqueness per property)

**Identified Logic**:
- **Email validation**: Format check, duplicate staff detection per property
- **Invitation token**: Generation, 7-day expiration tracking, validation (BR-067)
- **Permission enforcement**: Role-based access control (RBAC), immediate application (BR-070)
- **Role templates**: Pre-configured permissions (Receptionist, Manager)
- **Access revocation**: Token invalidation when suspending/removing staff

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| StaffInvitationValidator | «business logic» | Validates staff invitation: email format check, duplicate staff detection (step 8: email already associated with property), validates permission configuration. Prevents duplicate invitations per property. |
| InvitationTokenManager | «service» | Manages invitation tokens: generates secure tokens, tracks 7-day expiration (BR-067), validates token status (valid/used/expired in step 14), invalidates on use or cancellation. Critical for security. |
| PermissionManager | «service» | Manages role-based permissions: applies role templates (Receptionist, Manager), configures custom permissions (view/manage bookings, check-in/out, analytics, property settings, room types), enforces permission changes immediately (BR-070 no re-login required). Owner role cannot be assigned to staff (BR-069). |
| AccessControlManager | «service» | Manages staff access: associates User with Hostel via Staff entity, enforces role-based access control throughout system, revokes access tokens when suspending staff (immediate effect), maintains audit trail of permission changes. |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| StaffManagementUI | «user interaction» | Boundary (I/O) | HostelOwner | Staff management interface |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Invitation and notification emails |
| StaffManagementControl | «state-dependent control» | Control | N/A | Staff invitation and lifecycle orchestration |
| Staff | «entity» | Entity | Staff | Staff assignments and permissions |
| User | «entity» | Entity | User | Staff accounts |
| Notification | «entity» | Entity | Notification | Email communications |
| Hostel | «entity» | Entity | Hostel | Property reference |
| StaffInvitationValidator | «business logic» | Application Logic | N/A | Invitation validation |
| InvitationTokenManager | «service» | Application Logic | N/A | Token generation and validation |
| PermissionManager | «service» | Application Logic | N/A | Role and permission configuration |
| AccessControlManager | «service» | Application Logic | N/A | Access enforcement and revocation |

**Total**: 11 objects (2 boundary, 4 entity, 1 control, 4 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actor**: EmailService
- **Boundary**: StaffManagementUI, EmailServiceProxy
- **Control**: StaffManagementControl
- **Entity**: Staff, User, Notification, Hostel
- **Logic**: StaffInvitationValidator, InvitationTokenManager, PermissionManager, AccessControlManager

**Key Interactions (Invite Staff)**:
1. HostelOwner → StaffManagementUI: Click "Invite Staff Member"
2. StaffManagementUI → StaffManagementControl: Start invitation
3. StaffManagementControl → StaffManagementUI: Display invitation form
4. HostelOwner → StaffManagementUI: Enter email, name, select role, configure permissions
5. HostelOwner → StaffManagementUI: Submit invitation
6. StaffManagementUI → StaffManagementControl: Process invitation
7. StaffManagementControl → StaffInvitationValidator: Validate email
8. StaffInvitationValidator → Staff: Check duplicate (email already staff at property)
9. StaffManagementControl → InvitationTokenManager: Generate token
10. InvitationTokenManager: Create secure token with 7-day expiration
11. StaffManagementControl → Staff: Create pending invitation
12. StaffManagementControl → PermissionManager: Save role and permissions
13. StaffManagementControl → Notification: Create invitation email
14. StaffManagementControl → EmailServiceProxy: Send invitation
15. EmailServiceProxy → EmailService: Deliver email with registration link
16. StaffManagementControl → StaffManagementUI: Display confirmation

**Staff Registration Flow** (Invitee side):
1. Staff clicks invitation link in email
2. StaffManagementUI → StaffManagementControl: Validate token
3. StaffManagementControl → InvitationTokenManager: Check token
4. InvitationTokenManager validates: not used, not expired (<7 days)
5. If valid: Display registration form
6. Staff creates account or logs in
7. StaffManagementControl → User: Create/retrieve account
8. StaffManagementControl → Staff: Update status = Active
9. StaffManagementControl → AccessControlManager: Apply permissions
10. AccessControlManager associates User with Hostel via Staff
11. StaffManagementControl → Notification: Create confirmation email
12. StaffManagementControl → EmailServiceProxy: Send to staff and owner

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: StaffManagementControl

**States Identified**:
1. **NoInvitation**: Initial state, no pending invitation
2. **CreatingInvitation**: Processing invitation form
3. **ValidatingInvitation**: Checking email, duplicates
4. **PendingInvitation**: Invitation sent, awaiting registration (7-day window)
5. **InvitationExpired**: Token expired after 7 days
6. **StaffRegistering**: Staff clicked link, creating/logging into account
7. **ValidatingToken**: Checking token status (valid/used/expired)
8. **Active**: Staff registered and has property access
9. **UpdatingPermissions**: Modifying staff permissions
10. **Suspended**: Access temporarily revoked
11. **Removed**: Staff association deleted
12. **InvitationCancelled**: Owner cancelled pending invitation

**Events**:
- inviteStaff
- validationSuccess / validationFailed / duplicateEmail
- tokenGenerated
- invitationSent
- tokenExpired (after 7 days)
- staffClickedLink
- tokenValid / tokenExpired / tokenUsed
- staffRegistered
- permissionsApplied
- editPermissions
- permissionsUpdated
- suspendStaff
- accessRevoked
- removeStaff
- associationDeleted
- cancelInvitation
- resendInvitation

---

## Validation

- [x] At least 1 boundary object (2 identified)
- [x] Boundary → External mapping (1:1):
  - StaffManagementUI ← HostelOwner
  - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
  - Staff ✅
  - User ✅
  - Notification ✅
  - Hostel ✅
- [x] Control justified (state-dependent invitation workflow with 7-day expiration)
- [x] State-dependent flagged (StaffManagementControl)
- [x] Logic justified (validation, token management, permissions, access control)

---

## Alternative Flows Mapping

**Step 8 Alternative** (Email already exists):
- StaffInvitationValidator → Staff: Query existing staff
- Detects duplicate email for this property
- StaffManagementControl receives error
- StaffManagementUI displays "Email already associated with your property"

**Step 14 Alternative** (Invitation expired):
- InvitationTokenManager checks token timestamp
- Detects > 7 days (BR-067)
- StaffManagementControl → InvitationExpired state
- StaffManagementUI displays "Invitation expired" with resend option

**Step 14 Alternative** (Invitation already used):
- InvitationTokenManager detects token.status = Used
- StaffManagementUI displays "Invitation already accepted" with login link

**Edit Staff Permissions**:
- HostelOwner clicks "Edit" on active staff
- StaffManagementControl → PermissionManager: Load current permissions
- Owner modifies role or specific permissions
- PermissionManager validates changes
- StaffManagementControl → Staff: Save updated permissions
- AccessControlManager applies immediately (BR-070 no re-login)
- Notification email sent to staff about permission changes

**Suspend Staff Access**:
- HostelOwner clicks "Suspend"
- StaffManagementControl → Suspended state
- AccessControlManager revokes access tokens immediately
- Staff cannot log in to property dashboard
- StaffManagementUI displays "Suspended" status
- Notification email sent to staff

**Remove Staff Member**:
- HostelOwner clicks "Remove"
- Confirmation dialog: "Remove [name]? They will lose all access"
- StaffManagementControl → Removed state
- AccessControlManager deletes Staff association
- User account remains but loses property access (BR-071)
- Notification email sent to removed staff

**Resend/Cancel Invitation**:
- **Resend**: InvitationTokenManager generates new token, invalidates old
- **Cancel**: StaffManagementControl → InvitationCancelled state, invalidates token, no notification

---

## Business Rules

- **BR-066**: Only owner can invite/manage staff (StaffManagementControl enforces)
- **BR-067**: Invitation expires after 7 days (InvitationTokenManager enforces)
- **BR-068**: One email can be staff at multiple properties (StaffInvitationValidator checks per-property uniqueness)
- **BR-069**: Owner role cannot be assigned to staff (PermissionManager validates)
- **BR-070**: Permission changes immediate (AccessControlManager enforces)
- **BR-071**: Removed staff retain User account (AccessControlManager logic)

---

## Notes

- **State-dependent**: Invitation workflow with 7-day expiration timer
- **RBAC**: Role-based access control throughout system
- **Token security**: Secure generation, expiration, validation
- **Immediate effects**: Permission changes, suspension apply without re-login (BR-070)
- **Email-driven**: Invitation prevents unauthorized staff additions
- **Granular permissions**: Beyond role templates, custom configuration
- **Suspended vs Removed**: Temporary vs permanent access revocation
- **Audit trail**: All permission changes logged
- **Multi-property**: Staff can work at multiple properties (BR-068)
- **Owner protection**: Owner role cannot be assigned/demoted (BR-069)
