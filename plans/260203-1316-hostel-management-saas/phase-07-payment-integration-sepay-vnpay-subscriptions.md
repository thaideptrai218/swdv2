---
title: "Phase 07: Payment Integration"
status: pending
priority: P1
effort: 5d
---

# Phase 07: Payment Integration - SePay, VNPay, Subscriptions

## Context Links

- [Vietnam Payments Report](../reports/researcher-260203-1204-vietnam-payments.md)
- [Phase 06: Guest Platform](./phase-06-guest-platform-search-listing-booking-ui.md)

## Overview

Integrate SePay (VietQR) and VNPay for both guest bookings and owner subscriptions. Handle Vietnam's manual payment approval flow with webhooks, reconciliation, and grace periods.

## Key Insights

- Vietnam has NO automated recurring billing - manual approval each cycle
- VietQR via SePay: lowest fees (0.5-1%), requires ref code matching
- VNPay: card payments, higher fees (2-3%), better for international
- Grace periods essential: 30-50% don't complete renewal on time

## Requirements

### Functional
- Generate VietQR codes for payments
- VNPay card payment flow
- Webhook handlers for payment confirmation
- Subscription invoice generation
- Payment status tracking
- Renewal reminders (email/SMS)
- Grace period handling
- Refund workflow
- Payment reconciliation UI

### Non-Functional
- Webhook processing <5s
- Payment confirmation <30s after bank transfer
- 99.9% webhook reliability
- PCI-compliant card handling (via VNPay)

## Architecture

### Payment Flow (Guest Booking)

```
1. Guest completes booking → Status: PENDING
2. Display VietQR code with unique ref code
3. Guest transfers from bank app
4. SePay webhook → Match ref code → Update payment
5. Payment COMPLETED → Booking CONFIRMED
6. Send confirmation email
```

### Subscription Flow (Owner)

```
Day -7: Generate invoice, send reminder email
Day -3: Send SMS reminder
Day 0: Subscription expires, enter grace period
Day +3: Send warning, limited features
Day +7: Suspend account
Day +14: Final notice before data archive
```

### Webhook Architecture

```
Payment Gateway → HTTPS Webhook → Queue (RabbitMQ) → Handler
                                       ↓
                              Retry on failure (3x)
                                       ↓
                              Dead letter queue → Manual review
```

## Related Code Files

### Create
- `apps/api/src/modules/payment/payment.module.ts`
- `apps/api/src/modules/payment/payment.service.ts`
- `apps/api/src/modules/payment/sepay.service.ts`
- `apps/api/src/modules/payment/vnpay.service.ts`
- `apps/api/src/modules/payment/webhook.controller.ts`
- `apps/api/src/modules/payment/payment.processor.ts`
- `apps/api/src/modules/subscription/subscription.service.ts`
- `apps/api/src/modules/subscription/invoice.service.ts`
- `apps/api/src/modules/subscription/reminder.service.ts`
- `apps/web/src/components/payment/qr-code-display.tsx`
- `apps/web/src/components/payment/payment-status.tsx`
- `apps/web/src/components/payment/vnpay-redirect.tsx`
- `apps/web/src/app/(dashboard)/billing/page.tsx`
- `packages/shared/src/schemas/payment.schema.ts`

## Implementation Steps

### 1. Payment Module Setup (Day 1)

```typescript
// apps/api/src/modules/payment/payment.module.ts
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';
import { PaymentService } from './payment.service';
import { SepayService } from './sepay.service';
import { VnpayService } from './vnpay.service';
import { WebhookController } from './webhook.controller';
import { PaymentProcessor } from './payment.processor';

@Module({
  imports: [
    BullModule.registerQueue({
      name: 'payment',
    }),
  ],
  controllers: [WebhookController],
  providers: [
    PaymentService,
    SepayService,
    VnpayService,
    PaymentProcessor,
  ],
  exports: [PaymentService],
})
export class PaymentModule {}
```

### 2. SePay VietQR Integration (Day 1-2)

```typescript
// apps/api/src/modules/payment/sepay.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as crypto from 'crypto';

interface QRCodeParams {
  amount: number;
  refCode: string;
  description: string;
}

@Injectable()
export class SepayService {
  private readonly logger = new Logger(SepayService.name);
  private readonly bankCode: string;
  private readonly accountNumber: string;
  private readonly accountName: string;
  private readonly webhookSecret: string;

  constructor(private config: ConfigService) {
    this.bankCode = config.get('SEPAY_BANK_CODE');
    this.accountNumber = config.get('SEPAY_ACCOUNT_NUMBER');
    this.accountName = config.get('SEPAY_ACCOUNT_NAME');
    this.webhookSecret = config.get('SEPAY_WEBHOOK_SECRET');
  }

  generateRefCode(): string {
    // Format: HS + timestamp + random (12 chars total)
    const timestamp = Date.now().toString(36).toUpperCase();
    const random = crypto.randomBytes(3).toString('hex').toUpperCase();
    return `HS${timestamp}${random}`.slice(0, 12);
  }

  generateQRData(params: QRCodeParams): string {
    // VietQR format
    const template = 'compact2';
    return `https://img.vietqr.io/image/${this.bankCode}-${this.accountNumber}-${template}.png?amount=${params.amount}&addInfo=${params.refCode}&accountName=${encodeURIComponent(this.accountName)}`;
  }

  generateQRContent(params: QRCodeParams): object {
    return {
      bankCode: this.bankCode,
      accountNumber: this.accountNumber,
      accountName: this.accountName,
      amount: params.amount,
      description: params.refCode,
      qrDataURL: this.generateQRData(params),
    };
  }

  verifyWebhookSignature(payload: string, signature: string): boolean {
    const expectedSignature = crypto
      .createHmac('sha256', this.webhookSecret)
      .update(payload)
      .digest('hex');

    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(expectedSignature)
    );
  }

  parseWebhookPayload(payload: any): {
    refCode: string;
    amount: number;
    transactionId: string;
    paidAt: Date;
  } {
    return {
      refCode: payload.transferContent || payload.description,
      amount: payload.transferAmount,
      transactionId: payload.id,
      paidAt: new Date(payload.transferTime),
    };
  }
}
```

### 3. VNPay Integration (Day 2)

```typescript
// apps/api/src/modules/payment/vnpay.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as crypto from 'crypto';
import * as qs from 'qs';

interface VnpayPaymentParams {
  orderId: string;
  amount: number;
  orderInfo: string;
  returnUrl: string;
  ipAddress: string;
}

@Injectable()
export class VnpayService {
  private readonly tmnCode: string;
  private readonly hashSecret: string;
  private readonly paymentUrl: string;

  constructor(private config: ConfigService) {
    this.tmnCode = config.get('VNPAY_TMN_CODE');
    this.hashSecret = config.get('VNPAY_HASH_SECRET');
    this.paymentUrl = config.get('VNPAY_PAYMENT_URL');
  }

  createPaymentUrl(params: VnpayPaymentParams): string {
    const date = new Date();
    const createDate = this.formatDate(date);
    const expireDate = this.formatDate(new Date(date.getTime() + 15 * 60 * 1000));

    const vnpParams: Record<string, string | number> = {
      vnp_Version: '2.1.0',
      vnp_Command: 'pay',
      vnp_TmnCode: this.tmnCode,
      vnp_Locale: 'vn',
      vnp_CurrCode: 'VND',
      vnp_TxnRef: params.orderId,
      vnp_OrderInfo: params.orderInfo,
      vnp_OrderType: 'other',
      vnp_Amount: params.amount * 100, // VNPay uses smallest unit
      vnp_ReturnUrl: params.returnUrl,
      vnp_IpAddr: params.ipAddress,
      vnp_CreateDate: createDate,
      vnp_ExpireDate: expireDate,
    };

    // Sort and create signature
    const sortedParams = this.sortObject(vnpParams);
    const signData = qs.stringify(sortedParams, { encode: false });
    const signature = crypto
      .createHmac('sha512', this.hashSecret)
      .update(signData)
      .digest('hex');

    return `${this.paymentUrl}?${signData}&vnp_SecureHash=${signature}`;
  }

  verifyReturnUrl(query: Record<string, string>): {
    isValid: boolean;
    orderId: string;
    transactionId: string;
    responseCode: string;
  } {
    const secureHash = query.vnp_SecureHash;
    delete query.vnp_SecureHash;
    delete query.vnp_SecureHashType;

    const sortedParams = this.sortObject(query);
    const signData = qs.stringify(sortedParams, { encode: false });
    const expectedHash = crypto
      .createHmac('sha512', this.hashSecret)
      .update(signData)
      .digest('hex');

    return {
      isValid: secureHash === expectedHash,
      orderId: query.vnp_TxnRef,
      transactionId: query.vnp_TransactionNo,
      responseCode: query.vnp_ResponseCode,
    };
  }

  private formatDate(date: Date): string {
    return date.toISOString().replace(/[-:T.Z]/g, '').slice(0, 14);
  }

  private sortObject(obj: Record<string, any>): Record<string, any> {
    return Object.keys(obj)
      .sort()
      .reduce((result, key) => {
        result[key] = obj[key];
        return result;
      }, {} as Record<string, any>);
  }
}
```

### 4. Payment Service (Day 2-3)

```typescript
// apps/api/src/modules/payment/payment.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';
import { SepayService } from './sepay.service';
import { VnpayService } from './vnpay.service';
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class PaymentService {
  private readonly logger = new Logger(PaymentService.name);

  constructor(
    private prisma: PrismaService,
    private sepay: SepayService,
    private vnpay: VnpayService,
    @InjectQueue('payment') private paymentQueue: Queue,
    private eventEmitter: EventEmitter2,
  ) {}

  async createBookingPayment(bookingId: string, method: 'VIETQR' | 'VNPAY_CARD') {
    const booking = await this.prisma.booking.findUnique({
      where: { id: bookingId },
      include: { guest: true },
    });

    if (!booking) throw new Error('Booking not found');

    const refCode = this.sepay.generateRefCode();

    const payment = await this.prisma.payment.create({
      data: {
        bookingId,
        amount: booking.totalPrice,
        method,
        status: 'PENDING',
        refCode,
        provider: method === 'VIETQR' ? 'sepay' : 'vnpay',
      },
    });

    if (method === 'VIETQR') {
      const qrData = this.sepay.generateQRContent({
        amount: Number(booking.totalPrice),
        refCode,
        description: `Booking ${bookingId.slice(0, 8)}`,
      });

      return { payment, qrData };
    }

    // VNPay redirect
    return { payment };
  }

  async handleSepayWebhook(payload: any, signature: string) {
    // Verify signature
    const isValid = this.sepay.verifyWebhookSignature(
      JSON.stringify(payload),
      signature
    );

    if (!isValid) {
      this.logger.warn('Invalid SePay webhook signature');
      return;
    }

    // Queue for processing
    await this.paymentQueue.add('process-sepay', payload, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 1000 },
    });
  }

  async processSepayPayment(payload: any) {
    const { refCode, amount, transactionId, paidAt } =
      this.sepay.parseWebhookPayload(payload);

    // Find payment by ref code
    const payment = await this.prisma.payment.findUnique({
      where: { refCode },
      include: { booking: true },
    });

    if (!payment) {
      this.logger.warn(`Payment not found for ref code: ${refCode}`);
      // Store for manual reconciliation
      await this.storeUnmatchedPayment(payload);
      return;
    }

    // Verify amount matches
    if (Number(payment.amount) !== amount) {
      this.logger.warn(`Amount mismatch: expected ${payment.amount}, got ${amount}`);
      await this.prisma.payment.update({
        where: { id: payment.id },
        data: {
          status: 'FAILED',
          metadata: { error: 'Amount mismatch', received: amount },
        },
      });
      return;
    }

    // Update payment and booking
    await this.prisma.$transaction([
      this.prisma.payment.update({
        where: { id: payment.id },
        data: {
          status: 'COMPLETED',
          providerRef: transactionId,
          paidAt,
        },
      }),
      this.prisma.booking.update({
        where: { id: payment.bookingId },
        data: { status: 'CONFIRMED' },
      }),
    ]);

    // Emit event for notifications
    this.eventEmitter.emit('payment.completed', {
      paymentId: payment.id,
      bookingId: payment.bookingId,
    });
  }

  private async storeUnmatchedPayment(payload: any) {
    // Store in separate table for manual review
    await this.prisma.$executeRaw`
      INSERT INTO unmatched_payments (payload, created_at)
      VALUES (${JSON.stringify(payload)}::jsonb, NOW())
    `;
  }
}
```

### 5. Webhook Controller (Day 3)

```typescript
// apps/api/src/modules/payment/webhook.controller.ts
import { Controller, Post, Body, Headers, RawBodyRequest, Req } from '@nestjs/common';
import { Public } from '@/modules/auth/decorators/public.decorator';
import { PaymentService } from './payment.service';
import { Request } from 'express';

@Controller('webhooks')
export class WebhookController {
  constructor(private paymentService: PaymentService) {}

  @Public()
  @Post('sepay')
  async handleSepay(
    @Req() req: RawBodyRequest<Request>,
    @Headers('x-sepay-signature') signature: string,
  ) {
    const rawBody = req.rawBody?.toString() || '';
    await this.paymentService.handleSepayWebhook(
      JSON.parse(rawBody),
      signature
    );
    return { received: true };
  }

  @Public()
  @Post('vnpay/ipn')
  async handleVnpayIPN(@Body() body: any) {
    await this.paymentService.handleVnpayIPN(body);
    return { RspCode: '00', Message: 'Confirm Success' };
  }
}
```

### 6. Subscription Service (Day 4)

```typescript
// apps/api/src/modules/subscription/subscription.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';
import { Cron, CronExpression } from '@nestjs/schedule';
import { InvoiceService } from './invoice.service';
import { ReminderService } from './reminder.service';

const PLAN_PRICES = {
  BASIC: 500000,    // 500k VND/month
  PRO: 1200000,     // 1.2M VND/month
  ENTERPRISE: 3000000, // 3M VND/month
};

@Injectable()
export class SubscriptionService {
  private readonly logger = new Logger(SubscriptionService.name);

  constructor(
    private prisma: PrismaService,
    private invoiceService: InvoiceService,
    private reminderService: ReminderService,
  ) {}

  async createSubscription(tenantId: string, plan: 'BASIC' | 'PRO' | 'ENTERPRISE') {
    const startDate = new Date();
    const endDate = new Date();
    endDate.setMonth(endDate.getMonth() + 1);

    const subscription = await this.prisma.subscription.create({
      data: {
        tenantId,
        plan,
        status: 'ACTIVE',
        startDate,
        endDate,
      },
    });

    // Create first invoice
    await this.invoiceService.createInvoice({
      tenantId,
      amount: PLAN_PRICES[plan],
      dueDate: startDate,
    });

    return subscription;
  }

  // Run every day at 9 AM Vietnam time
  @Cron('0 9 * * *', { timeZone: 'Asia/Ho_Chi_Minh' })
  async processSubscriptionReminders() {
    const now = new Date();
    const in7Days = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);
    const in3Days = new Date(now.getTime() + 3 * 24 * 60 * 60 * 1000);

    // 7-day reminders
    const expiringIn7Days = await this.prisma.subscription.findMany({
      where: {
        status: 'ACTIVE',
        endDate: {
          gte: now,
          lte: in7Days,
        },
      },
      include: { tenant: true },
    });

    for (const sub of expiringIn7Days) {
      await this.reminderService.sendRenewalReminder(sub.tenant, 7);
      await this.invoiceService.createRenewalInvoice(sub);
    }

    // 3-day reminders
    const expiringIn3Days = await this.prisma.subscription.findMany({
      where: {
        status: 'ACTIVE',
        endDate: {
          gte: now,
          lte: in3Days,
        },
      },
      include: { tenant: true },
    });

    for (const sub of expiringIn3Days) {
      await this.reminderService.sendRenewalReminder(sub.tenant, 3, 'sms');
    }
  }

  // Run every hour to check expired subscriptions
  @Cron(CronExpression.EVERY_HOUR)
  async processExpiredSubscriptions() {
    const now = new Date();
    const gracePeriodEnd = new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000);

    // Mark as past due
    await this.prisma.subscription.updateMany({
      where: {
        status: 'ACTIVE',
        endDate: { lt: now },
      },
      data: { status: 'PAST_DUE' },
    });

    // Suspend after grace period
    await this.prisma.subscription.updateMany({
      where: {
        status: 'PAST_DUE',
        endDate: { lt: gracePeriodEnd },
      },
      data: { status: 'SUSPENDED' },
    });
  }

  async renewSubscription(subscriptionId: string, paymentId: string) {
    const subscription = await this.prisma.subscription.findUnique({
      where: { id: subscriptionId },
    });

    if (!subscription) throw new Error('Subscription not found');

    const newEndDate = new Date(subscription.endDate);
    newEndDate.setMonth(newEndDate.getMonth() + 1);

    return this.prisma.subscription.update({
      where: { id: subscriptionId },
      data: {
        status: 'ACTIVE',
        endDate: newEndDate,
      },
    });
  }
}
```

### 7. QR Code Display Component (Day 5)

```typescript
// apps/web/src/components/payment/qr-code-display.tsx
'use client';

import { useState, useEffect } from 'react';
import { trpc } from '@/trpc/client';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Copy, Check, RefreshCw } from 'lucide-react';
import Image from 'next/image';

interface QRCodeDisplayProps {
  paymentId: string;
  qrData: {
    qrDataURL: string;
    bankCode: string;
    accountNumber: string;
    accountName: string;
    amount: number;
    description: string;
  };
}

export function QRCodeDisplay({ paymentId, qrData }: QRCodeDisplayProps) {
  const [copied, setCopied] = useState(false);

  // Poll for payment status
  const { data: status } = trpc.payment.getStatus.useQuery(
    { paymentId },
    {
      refetchInterval: 5000, // Check every 5 seconds
      enabled: true,
    }
  );

  useEffect(() => {
    if (status?.status === 'COMPLETED') {
      // Redirect to success page
      window.location.href = `/booking/success?paymentId=${paymentId}`;
    }
  }, [status, paymentId]);

  const copyToClipboard = async (text: string) => {
    await navigator.clipboard.writeText(text);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  const formatPrice = (amount: number) => {
    return new Intl.NumberFormat('vi-VN', {
      style: 'currency',
      currency: 'VND',
    }).format(amount);
  };

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle className="text-center">Scan to Pay</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="flex justify-center">
          <Image
            src={qrData.qrDataURL}
            alt="VietQR Payment Code"
            width={250}
            height={250}
            className="rounded-lg"
          />
        </div>

        <div className="space-y-2 text-sm">
          <div className="flex justify-between">
            <span className="text-muted-foreground">Bank</span>
            <span className="font-medium">{qrData.bankCode}</span>
          </div>
          <div className="flex justify-between">
            <span className="text-muted-foreground">Account</span>
            <span className="font-medium">{qrData.accountNumber}</span>
          </div>
          <div className="flex justify-between">
            <span className="text-muted-foreground">Name</span>
            <span className="font-medium">{qrData.accountName}</span>
          </div>
          <div className="flex justify-between">
            <span className="text-muted-foreground">Amount</span>
            <span className="font-bold text-lg">
              {formatPrice(qrData.amount)}
            </span>
          </div>
          <div className="flex items-center justify-between">
            <span className="text-muted-foreground">Transfer Note</span>
            <div className="flex items-center gap-2">
              <code className="rounded bg-muted px-2 py-1 font-mono">
                {qrData.description}
              </code>
              <Button
                variant="ghost"
                size="sm"
                onClick={() => copyToClipboard(qrData.description)}
              >
                {copied ? (
                  <Check className="h-4 w-4 text-green-500" />
                ) : (
                  <Copy className="h-4 w-4" />
                )}
              </Button>
            </div>
          </div>
        </div>

        <div className="flex items-center justify-center gap-2 text-sm text-muted-foreground">
          <RefreshCw className="h-4 w-4 animate-spin" />
          Waiting for payment confirmation...
        </div>

        <p className="text-center text-xs text-muted-foreground">
          Important: Include the transfer note exactly as shown above.
          Payment will be confirmed automatically within 1-5 minutes.
        </p>
      </CardContent>
    </Card>
  );
}
```

### 8. Billing Page (Day 5)

```typescript
// apps/web/src/app/(dashboard)/billing/page.tsx
'use client';

import { trpc } from '@/trpc/client';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { QRCodeDisplay } from '@/components/payment/qr-code-display';
import { format } from 'date-fns';

export default function BillingPage() {
  const { data: subscription } = trpc.subscription.current.useQuery();
  const { data: invoices } = trpc.invoice.list.useQuery();

  const createPayment = trpc.invoice.createPayment.useMutation();

  const handlePayInvoice = async (invoiceId: string) => {
    const result = await createPayment.mutateAsync({ invoiceId });
    // Show QR code modal
  };

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">Billing</h1>

      {/* Current Plan */}
      <Card>
        <CardHeader>
          <CardTitle>Current Plan</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="flex items-center justify-between">
            <div>
              <div className="text-2xl font-bold">{subscription?.plan}</div>
              <div className="text-sm text-muted-foreground">
                {subscription?.status === 'ACTIVE' ? (
                  <>Renews on {format(subscription.endDate, 'MMM d, yyyy')}</>
                ) : (
                  <Badge variant="destructive">{subscription?.status}</Badge>
                )}
              </div>
            </div>
            <Button variant="outline">Change Plan</Button>
          </div>
        </CardContent>
      </Card>

      {/* Invoices */}
      <Card>
        <CardHeader>
          <CardTitle>Invoices</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="space-y-4">
            {invoices?.map((invoice) => (
              <div
                key={invoice.id}
                className="flex items-center justify-between rounded-lg border p-4"
              >
                <div>
                  <div className="font-medium">
                    {format(invoice.createdAt, 'MMMM yyyy')}
                  </div>
                  <div className="text-sm text-muted-foreground">
                    Due: {format(invoice.dueDate, 'MMM d, yyyy')}
                  </div>
                </div>
                <div className="flex items-center gap-4">
                  <div className="text-right">
                    <div className="font-bold">
                      {formatPrice(invoice.amount)}
                    </div>
                    <Badge
                      variant={
                        invoice.status === 'PAID' ? 'default' : 'secondary'
                      }
                    >
                      {invoice.status}
                    </Badge>
                  </div>
                  {invoice.status === 'PENDING' && (
                    <Button onClick={() => handlePayInvoice(invoice.id)}>
                      Pay Now
                    </Button>
                  )}
                </div>
              </div>
            ))}
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
```

## Todo List

- [ ] Create payment module structure
- [ ] Implement SePay VietQR service
- [ ] Implement VNPay service
- [ ] Create payment service with booking integration
- [ ] Build webhook controller with signature verification
- [ ] Create payment processor (queue handler)
- [ ] Implement subscription service
- [ ] Create invoice service
- [ ] Build reminder service (email/SMS)
- [ ] Create QR code display component
- [ ] Build payment status polling
- [ ] Create billing page for owners
- [ ] Implement VNPay redirect flow
- [ ] Add unmatched payment reconciliation
- [ ] Write payment integration tests
- [ ] Test webhook reliability

## Success Criteria

- [ ] VietQR payment completes within 5 minutes
- [ ] VNPay redirect flow works end-to-end
- [ ] Webhooks process with signature verification
- [ ] Subscription reminders sent on schedule
- [ ] Grace period handling works correctly
- [ ] Unmatched payments captured for review

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Webhook failures | High | Queue with retries, dead letter |
| Ref code typos | Medium | Clear UI, copy button |
| Gateway downtime | Medium | Multiple payment options |
| Amount mismatch | Medium | Store for manual reconciliation |

## Security Considerations

- Verify webhook signatures cryptographically
- Use HTTPS for all payment endpoints
- Never log full card numbers
- Rate limit payment creation
- Encrypt sensitive payment metadata
- PCI compliance via VNPay (no card storage)

## Next Steps

After completion, proceed to [Phase 08: Real-time & Notifications](./phase-08-real-time-sse-notifications-alerts.md)
