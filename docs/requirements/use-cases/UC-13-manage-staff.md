# UC-13: Manage Staff

| Field | Description |
|-------|-------------|
| **Use Case Name** | Manage Staff |
| **Use Case ID** | UC-13 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Hostel Owner invites staff members, assigns roles and permissions, manages active staff accounts, and removes staff access. System sends invitation emails via Email Service and enforces role-based access control. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A1: Hostel Owner. Secondary: A5: Email Service |
| **Preconditions** | Owner authenticated. Owner has active property. System operational. Email Service available. |
| **Trigger** | Owner clicks "Manage Staff" or "Team" button in property dashboard |
| **Main Sequence** | 1. Owner navigates to staff management page<br>2. System retrieves current staff list for property<br>3. System displays staff members table<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. For each staff: name, email, role, status (active/pending), last login, actions<br>4. Owner clicks "Invite Staff Member" button<br>5. System displays staff invitation form<br>6. Owner enters staff information<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner enters staff email address<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Owner enters staff full name<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner selects role (Receptionist, Manager, Owner)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Owner configures permissions<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.1. View bookings (read-only)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.2. Manage bookings (confirm, decline, modify)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.3. Check-in/check-out guests<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.4. View analytics reports<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.5. Manage property settings<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.6. Manage room types and pricing<br>7. System validates email format<br>8. System checks if email already associated with property staff<br>9. System creates pending staff invitation<br>10. System sends invitation email to staff via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Email includes: property name, inviting owner name, role assigned, registration link, expiration (7 days)<br>11. System displays confirmation "Invitation sent to [email]"<br>12. Staff member receives invitation email<br>13. Staff member clicks registration link<br>14. System validates invitation token<br>15. System displays staff registration form<br>16. Staff member creates account or logs in with existing account<br>17. System associates staff account with property<br>18. System applies assigned role and permissions<br>19. System sends confirmation email to staff and owner via Email Service<br>20. System updates staff status to "Active"<br>21. Staff member can now access property dashboard with assigned permissions |
| **Alternative Sequences** | **Step 8**: If email already exists as staff for this property, System displays error "This email is already associated with your property" and prevents duplicate invitation<br><br>**Step 14**: If invitation expired (>7 days), System displays "Invitation expired" and offers option to request new invitation from owner<br><br>**Step 14**: If invitation already used, System displays "Invitation already accepted" with link to login<br><br>**Edit Staff Permissions**:<br>- Owner clicks "Edit" on active staff member<br>- System displays permission editing form<br>- Owner modifies role or specific permissions<br>- System validates permission changes<br>- System saves updated permissions<br>- Changes apply immediately<br>- System sends notification email to staff about permission changes<br><br>**Suspend Staff Access**:<br>- Owner clicks "Suspend" on active staff<br>- System displays confirmation dialog<br>- Owner confirms suspension<br>- System updates staff status to "Suspended"<br>- System revokes access tokens<br>- Staff cannot log in to property dashboard<br>- Owner can reactivate anytime<br>- System sends notification email to staff<br><br>**Remove Staff Member**:<br>- Owner clicks "Remove" on staff<br>- System displays confirmation "Remove [name] from your property? They will lose all access."<br>- Owner confirms removal<br>- System checks for pending tasks assigned to staff<br>- System removes staff association with property<br>- System sends notification email to removed staff<br>- Staff account remains but loses property access<br><br>**Resend Invitation**:<br>- Owner clicks "Resend" on pending invitation<br>- System generates new invitation token<br>- System sends new invitation email via Email Service<br>- Previous invitation token invalidated<br><br>**Cancel Invitation**:<br>- Owner clicks "Cancel" on pending invitation<br>- System invalidates invitation token<br>- System removes pending invitation<br>- No notification sent to invitee<br><br>**Staff Self-Registration Disabled**:<br>- If staff email not invited by owner<br>- Staff cannot add themselves to property<br>- All staff must be explicitly invited by property owner |
| **Postconditions** | **Success**: Staff invitation sent or staff permissions updated. Staff member receives access with configured permissions. Owner retains full control. Activity logged.<br><br>**Failure**: Invitation not sent. Permission changes not applied. Error messages displayed. Owner can retry. |
| **Nonfunctional Requirements** | **Performance**: Staff list loads within 2 seconds. Invitation email sent within 10 seconds. Permission changes apply immediately.<br><br>**Security**: Role-based access control strictly enforced. Only property owner can invite/remove staff. Staff cannot escalate own permissions. Invitation tokens expire after 7 days. Suspended staff immediately lose access. Activity audit trail maintained.<br><br>**Usability**: Clear permission descriptions for each role. Pre-configured role templates (Receptionist, Manager). Custom permission configuration supported. Bulk invitation for multiple staff (future enhancement). Staff list searchable and sortable.<br><br>**Notifications**: Email notifications for: invitation sent, invitation accepted, permissions changed, access suspended, staff removed. Notification templates professional and clear.<br><br>**Compliance**: Staff data privacy protected. Personal information encrypted. GDPR right to be forgotten supported (remove staff). |
| **Business Requirements** | BR-066: Only property owner can invite and manage staff members<br>BR-067: Staff invitation expires after 7 days<br>BR-068: One email address can be staff at multiple properties<br>BR-069: Owner role cannot be assigned to staff (owner status separate)<br>BR-070: Permission changes apply immediately without requiring staff re-login<br>BR-071: Removed staff retain user account but lose property access |
| **Frequency of Use** | Low to Medium (initial setup intensive, then occasional updates) |
| **Priority** | Medium |
| **Outstanding Questions** | 1. Should system support staff schedules/shifts management?<br>2. Should owner be able to set permission expiration dates (temporary access)?<br>3. Should staff receive daily/weekly activity summaries?<br>4. Should system support staff departments (housekeeping, front desk)?<br>5. Should staff be able to request permission changes from owner?<br>6. Should system track staff performance metrics (bookings processed, response times)?<br>7. Should owner be able to delegate staff management to a manager role? |

## Related Use Cases

- **UC-12: Process Booking** — Staff with appropriate permissions can process bookings
- **UC-17: Check In Guest** — Staff performs check-ins based on permissions
- **UC-18: Check Out Guest** — Staff performs check-outs based on permissions
- **UC-19: View Booking Details** — Staff views bookings based on read permissions

## Notes

- Role-based access control essential for security and delegation
- Receptionist role focuses on front-desk operations (check-in/out, view bookings)
- Manager role includes analytics and property configuration
- Owner retains ultimate control and cannot be demoted
- Email invitation system prevents unauthorized staff additions
- Permission granularity allows customization beyond role templates
- Suspended access useful for temporary leaves without data loss
- Audit trail important for accountability and troubleshooting

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Owner delegates property operations to staff with controlled permissions

✓ **C2: Avoids functional decomposition** — Complete staff management sequence from invitation through activation to permission control

✓ **C3: Maintains black box view** — No mention of permission tables, access tokens, role hierarchies; only describes staff management behavior

✓ **C4: Primary and secondary actors identified** — Primary: Owner (manages staff); Secondary: Email Service (sends invitations and notifications)
