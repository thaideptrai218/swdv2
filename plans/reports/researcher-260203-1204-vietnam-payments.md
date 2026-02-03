# Vietnam Payment Integration Research Report

**Date:** 2026-02-03
**Context:** /home/welterial/projects/swdv2
**Focus:** SaaS subscription billing for Vietnam market

---

## Executive Summary

Vietnam payment landscape is fragmented. No native recurring billing. Manual reconciliation common. SePay/VietQR emerging as cost-effective solution for bank transfers. E-wallets (MoMo, ZaloPay, VNPay) dominate consumer payments but lack robust subscription APIs.

**Recommendation:** Hybrid approach - VietQR for bank transfers + VNPay for card payments + manual verification layer.

---

## 1. SePay/VietQR Integration

### Overview
- **VietQR:** National QR payment standard (2021+), bank-agnostic
- **SePay:** Payment aggregator providing VietQR automation APIs
- **Model:** Real-time bank transfer notifications via webhook

### Key Features
- One-time setup, reusable QR codes
- Instant webhook notifications (1-5 min delay)
- Transaction matching via reference code in transfer memo
- Lower fees than traditional gateways (0.5-1% vs 2-3%)

### Integration Flow
1. Generate QR code with unique ref code per invoice
2. Customer scans QR → transfers from any bank app
3. SePay webhook notifies payment received
4. Match ref code to subscription/invoice
5. Auto-activate subscription

### Limitations
- Manual intervention if memo missing/incorrect
- No automatic retry for failed payments
- Requires bank account reconciliation
- QR code expiry management needed

### Technical Requirements
- Webhook endpoint (HTTPS, public IP)
- Ref code generation system (8-12 chars, alphanumeric)
- Payment reconciliation database
- Cron job for expired payment checks (30-60 min timeout)

---

## 2. E-Wallet Alternatives

### VNPay
**Strengths:**
- Largest gateway in Vietnam (60%+ market share)
- Supports cards (Visa/Master/JCB), VNPay wallet, bank transfers
- Better docs than competitors (English available)
- Recurring token API (experimental, limited banks)

**Weaknesses:**
- High fees: 2-3% transaction + monthly/setup fees
- Recurring API not production-ready (2025 status)
- Complex onboarding (business license, contracts)
- Manual subscription management required

**Use Case:** Primary gateway for card payments, backup for e-wallet

### MoMo
**Strengths:**
- 30M+ users, highest e-wallet penetration
- Good mobile UX
- Instant payment confirmation

**Weaknesses:**
- No subscription/recurring API
- Business integration requires partnership deals
- High merchant fees (2.5-3%)
- Consumer-focused, not SaaS-friendly

**Use Case:** One-time payments only, not suitable for subscriptions

### ZaloPay
**Strengths:**
- Zalo ecosystem integration (100M users)
- Lower fees than MoMo (1.5-2.5%)
- Growing merchant adoption

**Weaknesses:**
- No recurring billing support
- Limited API documentation (mostly Vietnamese)
- Slower onboarding than VNPay

**Use Case:** Optional add-on for Zalo-centric customers

---

## 3. Subscription Billing Implementation

### Core Challenge
**Vietnam banking infrastructure does not support automated recurring charges.**

### Workarounds

#### A. Pseudo-Recurring via Reminders
1. Store customer's preferred payment method
2. Send payment reminder 3-7 days before renewal
3. Generate new invoice + payment link/QR
4. Customer manually approves payment
5. Grace period (3-7 days) before suspension

#### B. Prepaid Credit Model
1. Customer loads credits via one-time payment
2. Subscription deducts credits monthly
3. Auto-reminder when credits low
4. No forced renewals, customer-initiated top-ups

#### C. Tokenized Cards (Limited)
1. VNPay/OnePay token API for cards only
2. Store encrypted card token
3. Initiate charge via API
4. Success rate: 60-80% (OTP failures, expired cards)
5. Fallback to manual payment

### Recommended Flow
```
Day -7: Email reminder with payment link
Day -3: SMS reminder
Day 0: Subscription expires, enter grace period
Day +3: Account suspended, retain data
Day +7: Final reminder before account closure
Day +14: Archive account data
```

---

## 4. Recurring Payment Challenges

### Technical Challenges
- **No Direct Debit:** Banks don't expose APIs for merchant-initiated charges
- **OTP Requirements:** Card payments often require customer OTP approval
- **Token Expiry:** Stored card tokens expire, need re-authorization
- **Failed Payment Handling:** No retry mechanism, manual follow-up needed

### Regulatory Challenges
- **SBV Restrictions:** State Bank of Vietnam limits auto-debit capabilities
- **Consumer Protection:** Heavy bias toward customer approval for each charge
- **KYC Requirements:** Strict merchant verification for payment processors

### Operational Challenges
- **Reconciliation:** Manual matching of bank transfers to invoices
- **Refunds:** 7-14 day processing via bank transfer (no instant refunds)
- **Currency:** VND only, no multi-currency support
- **Tax Invoices:** Red invoices (Hóa đơn đỏ) required for B2B, manual issuance

---

## 5. Best Practices for Vietnamese SaaS

### Payment Flow Design
1. **Multiple Payment Options:** VietQR (primary) + VNPay (cards) + manual transfer
2. **Transparent Pricing:** Display fees clearly, absorb gateway costs when possible
3. **Flexible Billing Cycles:** Offer quarterly/annual for fewer transactions
4. **Grace Periods:** Minimum 3 days, recommend 7 days
5. **Manual Approval:** Accept that customers must approve each payment

### Technical Stack
```
Frontend: QR code display, payment status polling
Backend: Webhook handlers, ref code generator, reconciliation engine
Database: Invoices, payments, subscription status, transaction logs
Cron Jobs: Payment expiry checks (every 15 min), reminder emails
Monitoring: Payment success rate, webhook failures, reconciliation gaps
```

### User Experience
- **Pre-fill Transfer Memo:** Show exact ref code to copy-paste
- **Real-time Status:** Poll backend every 10s for payment confirmation
- **Mobile-First:** 80%+ payments happen on mobile
- **Fallback Options:** Always show manual bank transfer details
- **Customer Support:** Live chat for payment issues (common)

### Financial Management
- **Dual Accounting:** Track both gateway-confirmed and bank-confirmed payments
- **Reserve Fund:** Set aside 5-10% for refunds/disputes
- **Monthly Reconciliation:** Match gateway reports with bank statements
- **Tax Compliance:** Integrate with Vietnamese e-invoice systems (HDDTonline, MeInvoice)

### Security
- **Webhook Validation:** Verify signature/hash from payment gateway
- **HTTPS Only:** Required for PCI compliance
- **Rate Limiting:** Prevent ref code brute-forcing
- **Data Retention:** 5-year minimum for tax audits

---

## Implementation Roadmap

### Phase 1: Core Payment (Week 1-2)
- Integrate SePay/VietQR API
- Build webhook handler
- Implement ref code generation
- Create payment confirmation flow

### Phase 2: Multi-Gateway (Week 3)
- Add VNPay for card payments
- Unified payment interface
- Gateway fallback logic

### Phase 3: Subscription Management (Week 4-5)
- Invoice generation system
- Email/SMS reminder automation
- Grace period handling
- Subscription status management

### Phase 4: Reconciliation (Week 6)
- Manual payment matching UI
- Bank statement import
- Discrepancy reporting
- Refund workflow

---

## Cost Analysis

### Transaction Fees (per 100,000 VND payment)
- **VietQR/SePay:** 500-1,000 VND (0.5-1%)
- **VNPay Cards:** 2,000-3,000 VND (2-3%)
- **MoMo/ZaloPay:** 1,500-2,500 VND (1.5-2.5%)
- **Manual Transfer:** 0 VND (customer pays transfer fee)

### Setup Costs
- **VNPay:** 10-50M VND setup + 2-5M VND/month
- **SePay:** 5-10M VND setup + 1-2M VND/month
- **MoMo/ZaloPay:** Negotiable, partnership-based

### Recommendation
Start with SePay for 80% of transactions, VNPay for premium customers with cards.

---

## Critical Risks

1. **Payment Drop-off:** 30-50% don't complete manual renewal
2. **Webhook Failures:** Network issues cause missed payments
3. **Ref Code Errors:** Customer typos break reconciliation
4. **Regulatory Changes:** SBV can change rules without notice
5. **Gateway Downtime:** No SLA guarantees from local providers

### Mitigations
- Multiple payment reminders with escalating urgency
- Webhook retry mechanism + fallback polling
- OCR for bank statement parsing (reduce manual matching)
- Diversify across 2+ gateways
- Build manual payment verification UI

---

## Open Questions

1. What is the target customer segment (B2B vs B2C)? Different tax/invoice requirements.
2. Preferred billing cycle (monthly/quarterly/annual)? Affects reconciliation complexity.
3. Is international card support needed? Requires different gateway (Stripe, PayPal).
4. Acceptable payment success rate target? 70-80% realistic for Vietnam.
5. Budget for payment gateway fees? Determines SePay vs VNPay priority.
6. Need red invoice integration? B2B requires VAT invoice automation.
7. Expected transaction volume? Affects gateway tier pricing.

---

**Next Steps:** Review with product/finance teams, select primary gateway, design subscription UX flow accounting for manual approval requirement.