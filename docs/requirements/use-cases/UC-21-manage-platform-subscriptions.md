# UC-21: Manage Platform Subscriptions

| Field | Description |
|-------|-------------|
| **Use Case Name** | Manage Platform Subscriptions |
| **Use Case ID** | UC-21 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Summary** | Platform Administrator views all subscriptions, adjusts plans for support cases, processes refunds, extends trials, and resolves billing issues. System updates subscription status and feature access. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A0: Platform Administrator. Secondary: None |
| **Preconditions** | Administrator authenticated. System operational. |
| **Trigger** | Administrator clicks "Subscriptions" in admin dashboard, or receives billing support ticket |
| **Main Sequence** | 1. Administrator navigates to subscriptions management<br>2. System displays subscription list with filters (Active, Trial, Expired, Cancelled)<br>3. Administrator searches subscription (owner email, property name, subscription ID)<br>4. System displays subscription details: plan, billing cycle, payment history, status<br>5. Administrator performs action: upgrade/downgrade plan, extend trial, process refund, cancel subscription, waive fees<br>6. System updates subscription and feature access<br>7. System displays confirmation with changes applied |
| **Alternative Sequences** | **Extend Trial**: Admin adds days to trial period, updates expiration date<br>**Process Refund**: Admin initiates refund, reason documented, amount returned to payment method<br>**Waive Fees**: Admin applies credit to account, reason documented for audit<br>**Cancel Subscription**: Admin cancels with immediate effect or end of period, documented reason |
| **Postconditions** | **Success**: Subscription updated. Feature access adjusted. Changes logged. Owner sees updated plan.<br>**Failure**: Error displayed. Subscription unchanged. Admin can retry. |
| **Nonfunctional Requirements** | **Performance**: Subscription list loads <3s. Actions complete <2s. Feature access updates immediately.<br>**Security**: Only admins modify subscriptions. All changes logged with reason. Refunds require approval workflow.<br>**Audit**: Revenue impact tracked. Reason documented for all manual changes. |
| **Business Requirements** | BR-116: Subscription changes require documented reason<br>BR-117: Refunds processed within 5 business days<br>BR-118: Trial extensions limited to 14 days maximum<br>BR-119: Manual plan changes logged for revenue reconciliation |
| **Frequency of Use** | Medium (support cases, special arrangements, billing issues) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system support bulk subscription operations?<br>2. Should refunds require approval from finance team?<br>3. Should admin changes trigger automated accounting entries?<br>4. What approval workflow for large refunds? |

## Related Use Cases
- **UC-15: Manage Subscription** — Owners manage own subscriptions
- **UC-09: Register Hostel** — New properties start with trial

## Notes
- Essential for customer support and retention
- Manual overrides necessary for special cases
- Audit trail critical for revenue reconciliation
- Refund processing impacts financial reporting

## Acceptance Criteria Validation
✓ **C1**: Admin resolves billing issues effectively
✓ **C2**: Complete management with feature adjustments
✓ **C3**: Black box view maintained
✓ **C4**: Primary (Admin) only actor
