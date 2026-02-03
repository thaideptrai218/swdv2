# UC-15: Manage Subscription

| Field | Description |
|-------|-------------|
| **Use Case Name** | Manage Subscription |
| **Use Case ID** | UC-15 |
| **Created By** | System Analyst |
| **Created Date** | 2026-02-04 |
| **Last Updated By** | System Analyst |
| **Last Updated Date** | 2026-02-04 |
| **Summary** | Hostel Owner views current subscription plan, upgrades or downgrades plan, updates payment method, and reviews billing history. System processes payments via Payment Gateway and adjusts feature access based on active plan. |
| **Dependency** | Include: None. Extend: None |
| **Actors** | Primary: A1: Hostel Owner. Secondary: A4: Payment Gateway |
| **Preconditions** | Owner authenticated. Owner has active property. Payment Gateway operational. System operational. |
| **Trigger** | Owner clicks "Subscription" or "Billing" button in account settings, or System prompts owner when trial period ending |
| **Main Sequence** | 1. Owner navigates to subscription management page<br>2. System retrieves current subscription details<br>3. System displays subscription dashboard<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Current plan name and pricing<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Billing cycle (monthly/annual)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Next billing date<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Plan features included<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Current usage vs plan limits<br>4. System displays available subscription plans<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. **Free Plan**: Single property, basic features, 5% commission<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. **Starter Plan** (₫299,000/month): Enhanced features, 3.5% commission, email support<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. **Professional Plan** (₫599,000/month): Advanced features, 2.5% commission, priority support, analytics<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. **Enterprise Plan** (₫1,499,000/month): Premium features, 1.5% commission, dedicated account manager, API access<br>5. Owner selects plan to view details<br>6. System displays plan comparison table<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Features by plan (checkmarks for included features)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Commission rates<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Support level<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Property limits<br>&nbsp;&nbsp;&nbsp;&nbsp;6.5. Staff account limits<br>7. Owner clicks "Upgrade" or "Downgrade" button<br>8. System displays plan change confirmation<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. New plan details<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Pro-rated amount (if mid-cycle upgrade)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Next billing amount and date<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Feature changes summary<br>9. Owner confirms plan change<br>10. System processes payment via Payment Gateway<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. If upgrade with immediate charge: Processes pro-rated payment<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. If downgrade: Schedules change for next billing cycle<br>11. Payment Gateway processes transaction<br>12. Payment Gateway sends confirmation to System<br>13. System updates subscription plan<br>14. System adjusts feature access immediately<br>15. System sends confirmation email with receipt<br>16. System displays success message "Subscription updated to [Plan Name]" |
| **Alternative Sequences** | **Step 10**: If payment fails (insufficient funds, expired card), Payment Gateway returns error, System displays payment failure message, Owner can update payment method and retry<br><br>**Free Trial Flow**:<br>- New property owners start with 14-day Professional Plan trial<br>- Trial period countdown displayed in dashboard<br>- 7 days before expiration: System sends reminder email<br>- 1 day before expiration: System sends urgent reminder<br>- On expiration: System downgrades to Free Plan if no payment method added<br>- Owner retains data but loses premium features<br><br>**Update Payment Method**:<br>- Owner clicks "Update Payment Method"<br>- System displays payment method form<br>- Owner enters new credit card details<br>- System sends to Payment Gateway for validation<br>- Payment Gateway tokenizes card (PCI compliant)<br>- System saves payment method token<br>- System displays "Payment method updated successfully"<br><br>**Cancel Subscription**:<br>- Owner clicks "Cancel Subscription"<br>- System displays cancellation confirmation with consequences<br>- Owner selects cancellation reason from dropdown<br>- Owner confirms cancellation<br>- System schedules cancellation for end of current billing period<br>- System sends cancellation confirmation email<br>- Owner retains access until period end<br>- At period end: System downgrades to Free Plan<br>- All premium features disabled<br><br>**View Billing History**:<br>- Owner clicks "Billing History" tab<br>- System displays invoice list (date, amount, plan, status, receipt link)<br>- Owner downloads invoice PDF<br>- System generates invoice with VAT details<br><br>**Annual Billing Discount**:<br>- Owner toggles "Billed Annually" switch<br>- System displays annual pricing (15% discount vs monthly)<br>- Owner confirms annual billing<br>- System processes upfront annual payment<br>- Next billing date set to 12 months ahead<br><br>**Reactivate Cancelled Subscription**:<br>- Owner previously cancelled subscription<br>- Owner clicks "Reactivate" on preferred plan<br>- System processes immediate payment<br>- Features restored immediately<br>- Billing cycle restarts<br><br>**Feature Usage Limits Exceeded**:<br>- Owner reaches plan limit (e.g., maximum properties)<br>- System prevents action with upgrade prompt<br>- "Upgrade to Starter plan to add more properties"<br>- Owner can upgrade immediately from prompt |
| **Postconditions** | **Success**: Subscription plan updated. Payment processed. Features adjusted. Owner receives confirmation and receipt. Billing cycle updated.<br><br>**Failure**: Plan not changed. Payment not processed. Error message displayed. Owner can retry or contact support. |
| **Nonfunctional Requirements** | **Performance**: Subscription page loads within 2 seconds. Payment processing completes within 10 seconds. Plan change applies immediately after payment confirmation.<br><br>**Security**: Payment card data never stored (tokenized by Payment Gateway). PCI DSS compliance maintained. Subscription changes logged for audit. Secure payment form (HTTPS, encryption).<br><br>**Financial**: Pro-rated charges calculated accurately for mid-cycle upgrades. Refunds processed per policy (typically no refunds, credit applied to next billing). VAT/tax calculated based on owner location. Currency support for Vietnamese Dong (₫).<br><br>**Reliability**: Payment retry logic for failed transactions. Grace period before feature restrictions (3 days past due). Billing email notifications reliable (99%+ delivery rate).<br><br>**Usability**: Plan comparison table clear and scannable. Pricing transparent with no hidden fees. Upgrade path obvious from feature limit prompts. Billing history easily accessible. Invoices downloadable for accounting. |
| **Business Requirements** | BR-078: Free plan available indefinitely with feature limitations<br>BR-079: Pro-rated charges for mid-cycle upgrades; downgrades apply next billing cycle<br>BR-080: 14-day free trial of Professional plan for new properties<br>BR-081: Annual billing offers 15% discount vs monthly<br>BR-082: Failed payment grace period: 3 days before feature restrictions<br>BR-083: Commission rates reduce with higher subscription tiers<br>BR-084: All plans allow unlimited bookings (no transaction limits) |
| **Frequency of Use** | Low (initial selection intensive, then occasional upgrades/updates) |
| **Priority** | High |
| **Outstanding Questions** | 1. Should system offer multi-property discount packages?<br>2. Should system support add-ons (extra staff accounts, premium support) separately from plan tiers?<br>3. Should owner be able to prepay for multiple months with discount?<br>4. Should system offer refunds for cancellations (pro-rated, full, none)?<br>5. Should system support payment methods beyond credit cards (bank transfer, PayPal)?<br>6. Should system offer loyalty discounts for long-term customers?<br>7. Should system support referral credits (discount for referring other owners)? |

## Related Use Cases

- **UC-09: Register Hostel** — New properties start with free trial subscription
- **UC-13: Manage Staff** — Staff account limits enforced by subscription plan
- **UC-14: View Analytics** — Advanced analytics available in higher-tier plans

## Notes

- SaaS pricing model enables predictable revenue
- Commission-based + subscription hybrid optimizes for growth
- Free plan reduces entry barriers for small hostels
- Trial period allows owners to test premium features
- Pro-rated billing fairness encourages mid-cycle upgrades
- Annual billing discount improves cash flow predictability
- Payment Gateway integration ensures PCI compliance
- Feature gates based on subscription enforce upgrade paths
- Transparent pricing builds trust
- Billing history supports owner accounting needs

## Acceptance Criteria Validation

✓ **C1: Delivers useful result to primary actor** — Owner manages subscription with clear plan options and immediate feature adjustments

✓ **C2: Avoids functional decomposition** — Complete subscription management sequence from plan selection through payment to feature activation

✓ **C3: Maintains black box view** — No mention of payment tokens, subscription tables, feature flags; only describes subscription behavior

✓ **C4: Primary and secondary actors identified** — Primary: Owner (manages subscription); Secondary: Payment Gateway (processes payments)
