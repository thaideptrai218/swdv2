# Object Structuring: Manage Subscription

**Use Case**: docs/requirements/use-cases/UC-15-manage-subscription.md
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

**Analysis**: Owner initiates via account settings or trial expiration prompt, views subscription plans, selects upgrade/downgrade, updates payment method, views billing history through web interface.

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| SubscriptionManagementUI | «user interaction» | Maps from HostelOwner external user. Handles subscription dashboard display (current plan, billing cycle, next date), plan comparison table, upgrade/downgrade confirmation, payment method form, billing history table, invoice download, trial countdown display, feature limit prompts. |

---

## 2. Entity Objects (Data)

**Data References**:
- Step 2-3: "Retrieves current subscription details" → Subscription entity
- Step 4: "Available subscription plans" → Plan definitions (may be Subscription plans or system configuration)
- Step 7-9: "Plan change confirmation" → Subscription entity (plan, billing)
- Step 10-12: "Payment processing" → Payment entity
- Step 13-14: "Updates subscription, adjusts features" → Subscription entity (plan, status), feature access updates
- Alternative: "Free trial" → Subscription entity (trial period tracking)
- Alternative: "Update payment method" → Payment entity (payment method token)
- Alternative: "Billing history" → Payment entity (invoices)

**Referenced Data**:
- **Subscription**: Owner's subscription
  - Attributes: subscriptionId, userId, planType, startDate, endDate, status (Active/Trial/Past Due/Cancelled), billingCycle (Monthly/Annual), nextBillingDate
- **Payment**: Subscription payments and invoices
  - Attributes: paymentId, amount, paymentDate, status, subscriptionId, invoiceUrl
- **User**: Owner account
  - Attributes: userId, role (Owner)
- **Hostel**: Property limits enforced by subscription
  - Attributes: hostelId, ownerId (limited by plan)

**Objects**:
| Entity | In Phase 1? | Status | Usage |
|--------|-------------|--------|-------|
| Subscription | Yes | ✅ | Primary entity managed (plan changes, status updates) |
| Payment | Yes | ✅ | Payment processing, billing history, invoices |
| User | Yes | ✅ | Owner reference |
| Hostel | Yes | ✅ | Property limits enforced by subscription tier |

---

## 3. Boundary Objects (Output)

**Output Type**:
- [x] Display/Screen (dashboard, plan comparison, confirmation)
- [x] Printer (invoice download)
- [ ] External device
- [x] Same as input (reusing SubscriptionManagementUI)

**Analysis**:
- Steps 3-6, 8, 16: Display subscription dashboard, plan comparison, confirmations
- Step 10-12: Payment processing via Payment Gateway
- Step 15: Send confirmation email with receipt
- Alternative: Download invoice PDFs

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| SubscriptionManagementUI | «user interaction» | Reused for output - displays subscription dashboard, plan comparison table (checkmarks for features), pro-rated amount calculations, billing history table, trial countdown, feature limit warnings, success messages, invoice PDFs |
| PaymentGatewayProxy | «proxy» | Maps from PaymentGateway external system. Processes subscription payments (step 10-12): pro-rated charges for upgrades, scheduled changes for downgrades, payment method tokenization (PCI compliant), transaction processing. Reused from UC-05. |
| EmailServiceProxy | «proxy» | Maps from EmailService external system. Sends subscription confirmation with receipt (step 15), trial expiration reminders (7 days, 1 day before), cancellation confirmations, payment failure notices. Reused from multiple UCs. |

---

## 4. Control Objects

**State-Dependent?**: [x] YES / [ ] NO

**Justification**: Subscription has state-dependent lifecycle:
- **Trial** state: 14-day free trial countdown for new properties (BR-080)
- **Active** state: Subscription paid and features enabled
- **Past Due** state: Payment failed, 3-day grace period before restrictions (BR-082)
- **Cancelled** state: Scheduled cancellation at period end
- **Downgraded** state: Downgrade scheduled for next billing cycle
- Trial expiration creates time-based state transition
- Payment failure triggers grace period countdown
- Different behavior based on subscription state (feature access, payment retry)

**Control Objects**:
| Object | Stereotype | Phase 4? | Justification |
|--------|------------|----------|---------------|
| SubscriptionControl | «state-dependent control» | ⚠️ Yes | Orchestrates subscription lifecycle through states: trial (14-day countdown) → active (features enabled) → past due (3-day grace) → restricted/downgraded. Manages plan changes (upgrades immediate, downgrades scheduled), payment processing (pro-rated charges), feature gates (property limits, staff limits, analytics access), trial expiration handling. Time-based state transitions for trial and grace periods (BR-080, BR-082). |

---

## 5. Application Logic

**Complex Rules**:
- [x] Multi-condition validation (payment validation, feature limits)
- [x] Algorithms (pro-rated calculation, commission rate determination)
- [x] Cross-entity validation (property count vs plan limit)

**Identified Logic**:
- **Pro-rated calculation**: For mid-cycle upgrades, calculate days remaining, pro-rate charge (BR-079)
- **Plan comparison**: Feature matrix, commission rates per tier (BR-083), property/staff limits
- **Feature gates**: Enforce property limits, staff account limits, analytics access based on plan
- **Trial management**: 14-day countdown (BR-080), expiration warnings (7 days, 1 day), auto-downgrade
- **Grace period**: 3 days past due before feature restrictions (BR-082)
- **Annual discount**: 15% off vs monthly pricing (BR-081)

**Objects**:
| Object | Stereotype | Justification |
|--------|------------|---------------|
| SubscriptionPlanManager | «service» | Manages subscription plans: defines plan tiers (Free, Starter ₫299K/mo, Professional ₫599K/mo, Enterprise ₫1,499K/mo), features per plan, commission rates (5% Free → 1.5% Enterprise per BR-083), property/staff limits, support level. Provides plan comparison table. |
| ProRatedCalculator | «algorithm» | Calculates pro-rated charges for mid-cycle upgrades (BR-079): (days remaining / total days) × new plan price. Downgrades apply next billing cycle (no immediate charge). Annual billing 15% discount (BR-081). Accurate to VND (Vietnamese Dong). |
| FeatureGateManager | «service» | Enforces feature access based on subscription plan: property count limits (varies by tier), staff account limits, analytics access (Professional+), API access (Enterprise only), support level. Displays upgrade prompts when limits reached. Immediate effect on plan changes (BR-070). |
| TrialManager | «service» | Manages 14-day trial period (BR-080): tracks trial countdown, sends reminder emails (7 days, 1 day before expiration), auto-downgrades to Free plan if no payment method added at expiration, preserves data but disables premium features. |
| PaymentRetryManager | «service» | Handles failed payments: implements 3-day grace period (BR-082), retry logic for failed transactions, sends payment failure notifications, schedules feature restrictions after grace period, tracks billing status (Active/Past Due). |

---

## Object Summary

| Object | Stereotype | Category | Phase 1 Link | Purpose |
|--------|------------|----------|--------------|---------|
| SubscriptionManagementUI | «user interaction» | Boundary (I/O) | HostelOwner | Subscription management interface |
| PaymentGatewayProxy | «proxy» | Boundary (Output) | PaymentGateway | Payment processing |
| EmailServiceProxy | «proxy» | Boundary (Output) | EmailService | Confirmation and notification emails |
| SubscriptionControl | «state-dependent control» | Control | N/A | Subscription lifecycle orchestration |
| Subscription | «entity» | Entity | Subscription | Subscription record |
| Payment | «entity» | Entity | Payment | Payment transactions and invoices |
| User | «entity» | Entity | User | Owner account |
| Hostel | «entity» | Entity | Hostel | Property limits enforcement |
| SubscriptionPlanManager | «service» | Application Logic | N/A | Plan definitions and comparison |
| ProRatedCalculator | «algorithm» | Application Logic | N/A | Pro-rated charge calculation |
| FeatureGateManager | «service» | Application Logic | N/A | Feature access enforcement |
| TrialManager | «service» | Application Logic | N/A | Trial period management |
| PaymentRetryManager | «service» | Application Logic | N/A | Failed payment handling |

**Total**: 13 objects (3 boundary, 4 entity, 1 control, 5 logic)

---

## Phase 3 Preparation

**Objects for Communication Diagram**:
- **Primary Actor**: HostelOwner
- **Secondary Actors**: PaymentGateway, EmailService
- **Boundary**: SubscriptionManagementUI, PaymentGatewayProxy, EmailServiceProxy
- **Control**: SubscriptionControl
- **Entity**: Subscription, Payment, User, Hostel
- **Logic**: SubscriptionPlanManager, ProRatedCalculator, FeatureGateManager, TrialManager, PaymentRetryManager

**Key Interactions (Upgrade Plan)**:
1. HostelOwner → SubscriptionManagementUI: Navigate to subscription
2. SubscriptionManagementUI → SubscriptionControl: Request current subscription
3. SubscriptionControl → Subscription: Retrieve current plan
4. SubscriptionControl → SubscriptionPlanManager: Get available plans
5. SubscriptionPlanManager: Return plan definitions with features and pricing
6. SubscriptionControl → SubscriptionManagementUI: Display dashboard and plans
7. HostelOwner → SubscriptionManagementUI: Select plan, click "Upgrade"
8. SubscriptionManagementUI → SubscriptionControl: Process upgrade
9. SubscriptionControl → ProRatedCalculator: Calculate pro-rated amount
10. ProRatedCalculator: (days remaining / total days) × new plan price
11. SubscriptionControl → SubscriptionManagementUI: Display confirmation
12. HostelOwner → SubscriptionManagementUI: Confirm upgrade
13. SubscriptionControl → PaymentGatewayProxy: Process payment
14. PaymentGatewayProxy → PaymentGateway: Charge pro-rated amount
15. PaymentGateway → PaymentGatewayProxy: Payment confirmation
16. SubscriptionControl → Payment: Create payment record
17. SubscriptionControl → Subscription: Update plan and billing cycle
18. SubscriptionControl → FeatureGateManager: Apply new feature access
19. FeatureGateManager: Enable features immediately
20. SubscriptionControl → EmailServiceProxy: Send confirmation with receipt
21. SubscriptionControl → SubscriptionManagementUI: Display success

**Trial Flow**:
1. New property registered (UC-09)
2. SubscriptionControl → TrialManager: Start 14-day trial
3. TrialManager → Subscription: Create with status = Trial, Professional plan
4. SubscriptionControl → FeatureGateManager: Enable Professional features
5. TrialManager: Schedule countdown (7 days, 1 day reminders)
6. At 7 days before: TrialManager → EmailServiceProxy: Send reminder
7. At 1 day before: TrialManager → EmailServiceProxy: Send urgent reminder
8. At expiration: TrialManager checks payment method
9. If no payment: TrialManager → Subscription: Update plan = Free, status = Active
10. FeatureGateManager: Disable premium features, retain data

---

## Phase 4 Requirements

**State-Dependent Controls**: 1

⚠️ **Statechart needed for**: SubscriptionControl

**States Identified**:
1. **NoSubscription**: Owner account created, no subscription yet
2. **TrialActive**: 14-day Professional trial (BR-080)
3. **TrialExpiring**: 7 days or less remaining
4. **TrialExpired**: Trial ended, no payment method
5. **Active**: Subscription paid, features enabled
6. **Upgrading**: Processing mid-cycle upgrade
7. **DowngradeScheduled**: Downgrade scheduled for next billing cycle
8. **PastDue**: Payment failed, in 3-day grace period (BR-082)
9. **Restricted**: Grace period expired, features limited
10. **Cancelled**: Cancellation scheduled for period end
11. **CancelledDowngraded**: Period ended, downgraded to Free
12. **PaymentMethodUpdating**: Updating payment card

**Events**:
- createSubscription
- startTrial
- trialWarning7Days / trialWarning1Day
- trialExpired
- paymentMethodAdded
- upgradePlan
- paymentSuccess / paymentFailed
- planUpgraded
- downgradePlan
- downgradeScheduled
- billingCycleEnd
- planDowngraded
- paymentDue
- paymentRetryFailed
- gracePeriodExpired
- featuresRestricted
- cancelSubscription
- cancellationScheduled
- updatePaymentMethod

---

## Validation

- [x] At least 1 boundary object (3 identified)
- [x] Boundary → External mapping (1:1):
  - SubscriptionManagementUI ← HostelOwner
  - PaymentGatewayProxy ← PaymentGateway
  - EmailServiceProxy ← EmailService
- [x] Entities in Phase 1:
  - Subscription ✅
  - Payment ✅
  - User ✅
  - Hostel ✅
- [x] Control justified (state-dependent subscription lifecycle with trial and grace periods)
- [x] State-dependent flagged (SubscriptionControl)
- [x] Logic justified (plan management, pro-rated calculation, feature gates, trial, payment retry)

---

## Alternative Flows Mapping

**Step 10 Alternative** (Payment fails):
- PaymentGatewayProxy receives error from PaymentGateway
- SubscriptionControl → PastDue state
- PaymentRetryManager starts 3-day grace period (BR-082)
- SubscriptionManagementUI displays payment failure message
- Owner can update payment method and retry

**Free Trial Flow** (BR-080):
- See Key Interactions - Trial Flow above

**Update Payment Method**:
- Owner clicks "Update Payment Method"
- SubscriptionControl → PaymentMethodUpdating state
- SubscriptionManagementUI displays secure payment form
- Owner enters new credit card details
- PaymentGatewayProxy tokenizes card (PCI compliant, never stored)
- SubscriptionControl → Subscription: Save payment method token
- EmailServiceProxy sends confirmation

**Cancel Subscription**:
- Owner clicks "Cancel Subscription"
- SubscriptionControl → CancellationConfirmation
- SubscriptionManagementUI displays consequences
- Owner selects reason, confirms
- SubscriptionControl → Cancelled state (scheduled for period end)
- EmailServiceProxy sends cancellation confirmation
- Owner retains access until period end
- At period end: SubscriptionControl → CancelledDowngraded state
- FeatureGateManager downgrades to Free plan

**View Billing History**:
- Owner clicks "Billing History" tab
- SubscriptionControl → Payment: Query invoice history
- SubscriptionManagementUI displays invoice table
- Owner downloads invoice PDF with VAT details

**Annual Billing Discount** (BR-081):
- Owner toggles "Billed Annually" switch
- ProRatedCalculator: Annual price = Monthly × 12 × 0.85 (15% discount)
- SubscriptionManagementUI displays annual pricing
- Owner confirms annual billing
- PaymentGatewayProxy processes upfront annual payment
- Subscription.nextBillingDate set to +12 months

**Feature Usage Limits Exceeded**:
- Owner attempts action exceeding plan limit (e.g., add property beyond limit)
- FeatureGateManager blocks action
- SubscriptionManagementUI displays upgrade prompt
- "Upgrade to Starter plan to add more properties"
- Owner can upgrade immediately from prompt

---

## Business Rules

- **BR-078**: Free plan available indefinitely (SubscriptionPlanManager defines Free tier)
- **BR-079**: Pro-rated upgrades, downgrades next cycle (ProRatedCalculator and SubscriptionControl enforce)
- **BR-080**: 14-day Professional trial for new properties (TrialManager enforces)
- **BR-081**: Annual billing 15% discount (ProRatedCalculator applies)
- **BR-082**: 3-day grace period for failed payments (PaymentRetryManager enforces)
- **BR-083**: Commission rates reduce with higher tiers (SubscriptionPlanManager defines)
- **BR-084**: Unlimited bookings all plans (FeatureGateManager no transaction limits)

---

## Notes

- **State-dependent**: Trial countdown, payment grace period create time-based states
- **SaaS pricing**: Subscription + commission hybrid model
- **Free plan**: Reduces entry barriers (BR-078)
- **Trial period**: 14 days Professional trial allows testing (BR-080)
- **Pro-rated fairness**: Mid-cycle upgrades fair charging (BR-079)
- **Annual discount**: 15% encourages commitment (BR-081)
- **Payment Gateway**: PCI compliance via tokenization
- **Feature gates**: Subscription tier enforces limits
- **Grace period**: 3 days prevents immediate restriction (BR-082)
- **Transparent pricing**: No hidden fees, clear comparison
