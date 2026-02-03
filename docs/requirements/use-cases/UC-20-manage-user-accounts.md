# UC-20: Manage User Accounts

| Field | Description |
|-------|-------------|
| **Use Case Name** | Manage User Accounts |
| **Use Case ID** | UC-20 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Summary** | Platform Administrator views all user accounts, suspends or deactivates accounts, resets passwords, verifies properties, and resolves account issues. System sends notifications via Email Service. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A0: Platform Administrator. Secondary: A5: Email Service |
| **Preconditions** | Administrator authenticated with admin permissions. System operational. Email Service available. |
| **Trigger** | Administrator clicks "User Management" in admin dashboard, or receives support ticket about account issue |
| **Main Sequence** | 1. Administrator navigates to user management page<br>2. System displays user list with filters (All, Travelers, Owners, Staff, Suspended)<br>3. Administrator searches for user (name, email, ID)<br>4. System displays user profile with: account details, properties owned, booking history, account status, login activity<br>5. Administrator performs action: view details, suspend account, reset password, verify property, delete account, or send notification<br>6. System executes action and logs activity<br>7. System sends notification email to user via Email Service (if applicable)<br>8. System displays confirmation message |
| **Alternative Sequences** | **Suspend Account**: Admin suspends for policy violation, System blocks login, sends email notification<br>**Reset Password**: Admin generates temporary password, sends to user email, requires change on next login<br>**Verify Property**: Admin approves property registration, updates status to Active, notifies owner<br>**Delete Account**: Admin confirms, System anonymizes data (GDPR), removes PII, retains transactional records |
| **Postconditions** | **Success**: User account updated. Notification sent. Action logged. Changes reflected system-wide.<br>**Failure**: Error displayed. Action not applied. Admin can retry. |
| **Nonfunctional Requirements** | **Performance**: User list loads <3s for 10K users. Search returns results <1s. Actions complete <2s.<br>**Security**: Admin actions logged immutably. PII access logged. Only admins access user management. Suspend effective immediately.<br>**Audit**: All actions logged with admin ID, timestamp, reason, before/after state. |
| **Business Requirements** | BR-111: Only platform administrators can manage user accounts<br>BR-112: Account suspension requires documented reason<br>BR-113: Deleted accounts anonymized per GDPR (PII removed, bookings retained)<br>BR-114: Admin actions logged for compliance audit<br>BR-115: Password resets expire after 24 hours |
| **Frequency of Use** | Medium (daily for support issues, occasional for policy enforcement) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support bulk account operations?<br>2. Should admins be able to impersonate users for troubleshooting?<br>3. Should automated suspension trigger for fraud detection?<br>4. What retention period for deleted account data? |

## Related Use Cases
- **UC-01: Register Account** — Users register accounts managed here
- **UC-09: Register Hostel** — Property verification performed here

## Notes
- Critical for platform governance and compliance
- GDPR compliance requires proper data deletion
- Audit trail essential for dispute resolution
- Suspension immediate to prevent further policy violations

## Acceptance Criteria Validation
✓ **C1**: Admin manages platform users effectively
✓ **C2**: Complete management sequence with notifications
✓ **C3**: Black box view maintained (no database mentions)
✓ **C4**: Primary (Admin), Secondary (Email Service) identified
