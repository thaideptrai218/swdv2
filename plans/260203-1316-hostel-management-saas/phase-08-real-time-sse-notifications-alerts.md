---
title: "Phase 08: Real-time & Notifications"
status: pending
priority: P2
effort: 3d
---

# Phase 08: Real-time & Notifications - SSE, Queue, Alerts

## Context Links

- [Tech Architecture Report](../reports/researcher-260203-1204-tech-architecture.md)
- [Phase 07: Payment Integration](./phase-07-payment-integration-sepay-vnpay-subscriptions.md)

## Overview

Implement real-time updates via Server-Sent Events (SSE) for booking notifications. Set up RabbitMQ for async task processing. Integrate email/SMS notifications and Discord alerts for ops.

## Key Insights

- SSE over WebSocket: simpler, HTTP-based, sufficient for one-way updates
- RabbitMQ for decoupling: notifications, reports, cleanup jobs
- Discord webhooks: free, instant alerts for operators
- Email via Resend or SendGrid; SMS via FPT/Viettel Gateway

## Requirements

### Functional
- Real-time booking updates in dashboard
- Email notifications (booking confirmation, reminders)
- SMS notifications (check-in reminders, payment due)
- Discord alerts (new bookings, payment issues, system errors)
- In-app notification center
- Notification preferences per user

### Non-Functional
- SSE connection stable for 8+ hours
- Notification delivery <30 seconds
- Queue processing <5 seconds
- 99% email delivery rate

## Architecture

### SSE Flow

```
NestJS Event → SSE Controller → HTTP Stream → Frontend EventSource
                    ↓
         Tenant-scoped channels (booking:tenant123)
```

### Queue Architecture

```
Producer (Service) → RabbitMQ Exchange → Queue → Consumer (Worker)
                          ↓
                 Routing keys:
                 - notification.email
                 - notification.sms
                 - notification.discord
                 - task.report
                 - task.cleanup
```

### Notification Flow

```
1. Event occurs (booking created)
2. Emit internal event
3. Notification service picks up
4. Route to appropriate queue
5. Worker sends via provider
6. Log delivery status
```

## Related Code Files

### Create
- `apps/api/src/modules/sse/sse.controller.ts`
- `apps/api/src/modules/sse/sse.service.ts`
- `apps/api/src/modules/notification/notification.module.ts`
- `apps/api/src/modules/notification/notification.service.ts`
- `apps/api/src/modules/notification/email.service.ts`
- `apps/api/src/modules/notification/sms.service.ts`
- `apps/api/src/modules/notification/discord.service.ts`
- `apps/api/src/modules/notification/notification.processor.ts`
- `apps/api/src/modules/notification/templates/*.ts`
- `apps/web/src/hooks/use-sse.ts`
- `apps/web/src/components/notifications/notification-center.tsx`
- `apps/web/src/components/notifications/notification-item.tsx`

## Implementation Steps

### 1. SSE Controller (Day 1)

```typescript
// apps/api/src/modules/sse/sse.controller.ts
import { Controller, Sse, Req, MessageEvent } from '@nestjs/common';
import { Observable, interval, map, filter } from 'rxjs';
import { SseService } from './sse.service';
import { CurrentUser } from '@/modules/auth/decorators/current-user.decorator';

@Controller('sse')
export class SseController {
  constructor(private sseService: SseService) {}

  @Sse('bookings')
  bookingStream(
    @CurrentUser() user: { id: string; tenantId: string }
  ): Observable<MessageEvent> {
    return this.sseService.getBookingUpdates(user.tenantId).pipe(
      map((event) => ({
        type: event.type,
        data: JSON.stringify(event.payload),
      }))
    );
  }

  @Sse('notifications')
  notificationStream(
    @CurrentUser() user: { id: string }
  ): Observable<MessageEvent> {
    return this.sseService.getNotifications(user.id).pipe(
      map((notification) => ({
        type: 'notification',
        data: JSON.stringify(notification),
      }))
    );
  }
}

// apps/api/src/modules/sse/sse.service.ts
import { Injectable } from '@nestjs/common';
import { Subject, Observable, filter } from 'rxjs';
import { OnEvent } from '@nestjs/event-emitter';

interface BookingEvent {
  type: 'booking.created' | 'booking.updated' | 'booking.cancelled';
  tenantId: string;
  payload: any;
}

@Injectable()
export class SseService {
  private bookingSubject = new Subject<BookingEvent>();
  private notificationSubject = new Subject<{ userId: string; data: any }>();

  getBookingUpdates(tenantId: string): Observable<BookingEvent> {
    return this.bookingSubject.pipe(
      filter((event) => event.tenantId === tenantId)
    );
  }

  getNotifications(userId: string): Observable<any> {
    return this.notificationSubject.pipe(
      filter((event) => event.userId === userId),
      map((event) => event.data)
    );
  }

  @OnEvent('booking.created')
  handleBookingCreated(payload: any) {
    this.bookingSubject.next({
      type: 'booking.created',
      tenantId: payload.tenantId,
      payload,
    });
  }

  @OnEvent('booking.updated')
  handleBookingUpdated(payload: any) {
    this.bookingSubject.next({
      type: 'booking.updated',
      tenantId: payload.tenantId,
      payload,
    });
  }

  emitNotification(userId: string, data: any) {
    this.notificationSubject.next({ userId, data });
  }
}
```

### 2. SSE Client Hook (Day 1)

```typescript
// apps/web/src/hooks/use-sse.ts
'use client';

import { useEffect, useCallback, useRef } from 'react';
import { useQueryClient } from '@tanstack/react-query';

interface UseSSEOptions {
  endpoint: string;
  onMessage?: (event: MessageEvent) => void;
  onError?: (error: Event) => void;
  enabled?: boolean;
}

export function useSSE({ endpoint, onMessage, onError, enabled = true }: UseSSEOptions) {
  const eventSourceRef = useRef<EventSource | null>(null);
  const queryClient = useQueryClient();

  const connect = useCallback(() => {
    if (!enabled) return;

    const token = localStorage.getItem('accessToken');
    const url = `${process.env.NEXT_PUBLIC_API_URL}${endpoint}?token=${token}`;

    const eventSource = new EventSource(url);

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      onMessage?.(data);

      // Invalidate relevant queries based on event type
      if (data.type?.startsWith('booking.')) {
        queryClient.invalidateQueries({ queryKey: ['booking'] });
      }
    };

    eventSource.onerror = (error) => {
      console.error('SSE error:', error);
      onError?.(error);

      // Reconnect after 5 seconds
      setTimeout(() => {
        eventSource.close();
        connect();
      }, 5000);
    };

    eventSourceRef.current = eventSource;
  }, [endpoint, enabled, onMessage, onError, queryClient]);

  useEffect(() => {
    connect();

    return () => {
      eventSourceRef.current?.close();
    };
  }, [connect]);

  return {
    close: () => eventSourceRef.current?.close(),
    reconnect: connect,
  };
}

// Usage in dashboard
export function BookingDashboard() {
  useSSE({
    endpoint: '/sse/bookings',
    onMessage: (event) => {
      if (event.type === 'booking.created') {
        toast.success(`New booking from ${event.data.guestName}`);
      }
    },
  });

  // ... rest of component
}
```

### 3. Notification Service (Day 2)

```typescript
// apps/api/src/modules/notification/notification.service.ts
import { Injectable } from '@nestjs/common';
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';
import { PrismaService } from '@/database/prisma.service';
import { OnEvent } from '@nestjs/event-emitter';

type NotificationType =
  | 'BOOKING_CONFIRMED'
  | 'BOOKING_CANCELLED'
  | 'CHECKIN_REMINDER'
  | 'PAYMENT_RECEIVED'
  | 'PAYMENT_DUE'
  | 'SUBSCRIPTION_EXPIRING';

interface NotificationPayload {
  type: NotificationType;
  recipientId: string;
  recipientEmail: string;
  recipientPhone?: string;
  data: Record<string, any>;
  channels: ('email' | 'sms' | 'push' | 'discord')[];
}

@Injectable()
export class NotificationService {
  constructor(
    private prisma: PrismaService,
    @InjectQueue('notification') private notificationQueue: Queue,
  ) {}

  async send(payload: NotificationPayload) {
    // Store notification in DB
    const notification = await this.prisma.notification.create({
      data: {
        userId: payload.recipientId,
        type: payload.type,
        data: payload.data,
        isRead: false,
      },
    });

    // Queue for each channel
    for (const channel of payload.channels) {
      await this.notificationQueue.add(channel, {
        ...payload,
        notificationId: notification.id,
      });
    }

    return notification;
  }

  @OnEvent('booking.confirmed')
  async onBookingConfirmed(booking: any) {
    await this.send({
      type: 'BOOKING_CONFIRMED',
      recipientId: booking.guestId,
      recipientEmail: booking.guest.email,
      recipientPhone: booking.guest.phone,
      data: {
        bookingId: booking.id,
        propertyName: booking.property.name,
        checkIn: booking.checkIn,
        checkOut: booking.checkOut,
        totalPrice: booking.totalPrice,
      },
      channels: ['email'],
    });

    // Also notify hostel owner via Discord
    await this.notificationQueue.add('discord', {
      type: 'NEW_BOOKING',
      data: booking,
    });
  }

  @OnEvent('payment.completed')
  async onPaymentCompleted(payment: any) {
    await this.send({
      type: 'PAYMENT_RECEIVED',
      recipientId: payment.booking.guestId,
      recipientEmail: payment.booking.guest.email,
      data: {
        amount: payment.amount,
        bookingId: payment.bookingId,
      },
      channels: ['email'],
    });
  }

  async markAsRead(notificationId: string, userId: string) {
    return this.prisma.notification.updateMany({
      where: { id: notificationId, userId },
      data: { isRead: true },
    });
  }

  async getUnreadCount(userId: string): Promise<number> {
    return this.prisma.notification.count({
      where: { userId, isRead: false },
    });
  }
}
```

### 4. Email Service (Day 2)

```typescript
// apps/api/src/modules/notification/email.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { Resend } from 'resend';

interface EmailOptions {
  to: string;
  subject: string;
  template: string;
  data: Record<string, any>;
}

@Injectable()
export class EmailService {
  private readonly logger = new Logger(EmailService.name);
  private readonly resend: Resend;
  private readonly fromEmail: string;

  constructor(private config: ConfigService) {
    this.resend = new Resend(config.get('RESEND_API_KEY'));
    this.fromEmail = config.get('EMAIL_FROM');
  }

  async send(options: EmailOptions): Promise<boolean> {
    try {
      const html = this.renderTemplate(options.template, options.data);

      await this.resend.emails.send({
        from: this.fromEmail,
        to: options.to,
        subject: options.subject,
        html,
      });

      this.logger.log(`Email sent to ${options.to}: ${options.subject}`);
      return true;
    } catch (error) {
      this.logger.error(`Failed to send email to ${options.to}`, error);
      return false;
    }
  }

  private renderTemplate(template: string, data: Record<string, any>): string {
    const templates: Record<string, (data: any) => string> = {
      BOOKING_CONFIRMED: (d) => `
        <h1>Booking Confirmed!</h1>
        <p>Your booking at ${d.propertyName} has been confirmed.</p>
        <ul>
          <li>Check-in: ${formatDate(d.checkIn)}</li>
          <li>Check-out: ${formatDate(d.checkOut)}</li>
          <li>Total: ${formatPrice(d.totalPrice)}</li>
        </ul>
        <p>Booking ID: ${d.bookingId}</p>
      `,
      PAYMENT_RECEIVED: (d) => `
        <h1>Payment Received</h1>
        <p>We've received your payment of ${formatPrice(d.amount)}.</p>
        <p>Booking ID: ${d.bookingId}</p>
      `,
      SUBSCRIPTION_EXPIRING: (d) => `
        <h1>Subscription Expiring Soon</h1>
        <p>Your ${d.plan} subscription will expire in ${d.daysRemaining} days.</p>
        <p><a href="${d.renewUrl}">Renew Now</a></p>
      `,
    };

    return templates[template]?.(data) || '';
  }
}
```

### 5. Discord Service (Day 2)

```typescript
// apps/api/src/modules/notification/discord.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import axios from 'axios';

interface DiscordEmbed {
  title: string;
  description?: string;
  color?: number;
  fields?: { name: string; value: string; inline?: boolean }[];
  timestamp?: string;
}

@Injectable()
export class DiscordService {
  private readonly logger = new Logger(DiscordService.name);
  private readonly webhookUrl: string;

  constructor(private config: ConfigService) {
    this.webhookUrl = config.get('DISCORD_WEBHOOK_URL');
  }

  async send(embed: DiscordEmbed): Promise<boolean> {
    try {
      await axios.post(this.webhookUrl, {
        embeds: [embed],
      });
      return true;
    } catch (error) {
      this.logger.error('Failed to send Discord notification', error);
      return false;
    }
  }

  async sendBookingAlert(booking: any) {
    await this.send({
      title: 'New Booking',
      color: 0x00ff00, // Green
      fields: [
        { name: 'Guest', value: booking.guest.name, inline: true },
        { name: 'Property', value: booking.property.name, inline: true },
        { name: 'Dates', value: `${booking.checkIn} - ${booking.checkOut}` },
        { name: 'Amount', value: formatPrice(booking.totalPrice), inline: true },
      ],
      timestamp: new Date().toISOString(),
    });
  }

  async sendPaymentAlert(payment: any, type: 'success' | 'failed') {
    await this.send({
      title: type === 'success' ? 'Payment Received' : 'Payment Failed',
      color: type === 'success' ? 0x00ff00 : 0xff0000,
      fields: [
        { name: 'Amount', value: formatPrice(payment.amount), inline: true },
        { name: 'Method', value: payment.method, inline: true },
        { name: 'Booking', value: payment.bookingId.slice(0, 8) },
      ],
      timestamp: new Date().toISOString(),
    });
  }

  async sendErrorAlert(error: Error, context: string) {
    await this.send({
      title: 'System Error',
      description: `\`\`\`${error.message}\`\`\``,
      color: 0xff0000,
      fields: [
        { name: 'Context', value: context },
        { name: 'Stack', value: `\`\`\`${error.stack?.slice(0, 500)}\`\`\`` },
      ],
      timestamp: new Date().toISOString(),
    });
  }
}
```

### 6. Notification Processor (Day 3)

```typescript
// apps/api/src/modules/notification/notification.processor.ts
import { Process, Processor } from '@nestjs/bull';
import { Logger } from '@nestjs/common';
import { Job } from 'bull';
import { EmailService } from './email.service';
import { SmsService } from './sms.service';
import { DiscordService } from './discord.service';
import { PrismaService } from '@/database/prisma.service';

@Processor('notification')
export class NotificationProcessor {
  private readonly logger = new Logger(NotificationProcessor.name);

  constructor(
    private emailService: EmailService,
    private smsService: SmsService,
    private discordService: DiscordService,
    private prisma: PrismaService,
  ) {}

  @Process('email')
  async handleEmail(job: Job) {
    const { recipientEmail, type, data, notificationId } = job.data;

    const success = await this.emailService.send({
      to: recipientEmail,
      subject: this.getEmailSubject(type),
      template: type,
      data,
    });

    await this.updateDeliveryStatus(notificationId, 'email', success);
  }

  @Process('sms')
  async handleSms(job: Job) {
    const { recipientPhone, type, data, notificationId } = job.data;

    if (!recipientPhone) {
      this.logger.warn('No phone number for SMS notification');
      return;
    }

    const success = await this.smsService.send({
      to: recipientPhone,
      message: this.getSmsMessage(type, data),
    });

    await this.updateDeliveryStatus(notificationId, 'sms', success);
  }

  @Process('discord')
  async handleDiscord(job: Job) {
    const { type, data } = job.data;

    switch (type) {
      case 'NEW_BOOKING':
        await this.discordService.sendBookingAlert(data);
        break;
      case 'PAYMENT_SUCCESS':
        await this.discordService.sendPaymentAlert(data, 'success');
        break;
      case 'PAYMENT_FAILED':
        await this.discordService.sendPaymentAlert(data, 'failed');
        break;
      case 'ERROR':
        await this.discordService.sendErrorAlert(data.error, data.context);
        break;
    }
  }

  private getEmailSubject(type: string): string {
    const subjects: Record<string, string> = {
      BOOKING_CONFIRMED: 'Your Booking is Confirmed!',
      PAYMENT_RECEIVED: 'Payment Received',
      CHECKIN_REMINDER: 'Check-in Reminder',
      SUBSCRIPTION_EXPIRING: 'Your Subscription is Expiring',
    };
    return subjects[type] || 'Notification';
  }

  private getSmsMessage(type: string, data: any): string {
    const messages: Record<string, (d: any) => string> = {
      CHECKIN_REMINDER: (d) =>
        `Reminder: Check-in tomorrow at ${d.propertyName}. Booking: ${d.bookingId.slice(0, 8)}`,
      PAYMENT_DUE: (d) =>
        `Payment due: ${formatPrice(d.amount)}. Pay now: ${d.paymentUrl}`,
    };
    return messages[type]?.(data) || '';
  }

  private async updateDeliveryStatus(
    notificationId: string,
    channel: string,
    success: boolean
  ) {
    // Update notification metadata
    await this.prisma.notification.update({
      where: { id: notificationId },
      data: {
        metadata: {
          [`${channel}Sent`]: success,
          [`${channel}SentAt`]: new Date().toISOString(),
        },
      },
    });
  }
}
```

### 7. Notification Center Component (Day 3)

```typescript
// apps/web/src/components/notifications/notification-center.tsx
'use client';

import { useState } from 'react';
import { trpc } from '@/trpc/client';
import { useSSE } from '@/hooks/use-sse';
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover';
import { Button } from '@/components/ui/button';
import { Bell } from 'lucide-react';
import { NotificationItem } from './notification-item';
import { ScrollArea } from '@/components/ui/scroll-area';

export function NotificationCenter() {
  const [open, setOpen] = useState(false);
  const { data: notifications, refetch } = trpc.notification.list.useQuery();
  const { data: unreadCount } = trpc.notification.unreadCount.useQuery();

  const markAsRead = trpc.notification.markAsRead.useMutation({
    onSuccess: () => refetch(),
  });

  // Real-time notifications
  useSSE({
    endpoint: '/sse/notifications',
    onMessage: () => refetch(),
  });

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button variant="ghost" size="icon" className="relative">
          <Bell className="h-5 w-5" />
          {unreadCount > 0 && (
            <span className="absolute -right-1 -top-1 flex h-5 w-5 items-center justify-center rounded-full bg-red-500 text-xs text-white">
              {unreadCount > 9 ? '9+' : unreadCount}
            </span>
          )}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-80 p-0" align="end">
        <div className="border-b p-4">
          <h3 className="font-semibold">Notifications</h3>
        </div>
        <ScrollArea className="h-96">
          {notifications?.length === 0 ? (
            <div className="p-8 text-center text-muted-foreground">
              No notifications
            </div>
          ) : (
            <div className="divide-y">
              {notifications?.map((notification) => (
                <NotificationItem
                  key={notification.id}
                  notification={notification}
                  onRead={() => markAsRead.mutate({ id: notification.id })}
                />
              ))}
            </div>
          )}
        </ScrollArea>
      </PopoverContent>
    </Popover>
  );
}
```

## Todo List

- [ ] Create SSE controller with tenant-scoped streams
- [ ] Build SSE service with event handling
- [ ] Create useSSE hook for frontend
- [ ] Setup RabbitMQ queues
- [ ] Implement notification service
- [ ] Create email service with templates
- [ ] Create SMS service
- [ ] Create Discord webhook service
- [ ] Build notification processor
- [ ] Create notification center component
- [ ] Add notification preferences
- [ ] Test real-time updates
- [ ] Test notification delivery
- [ ] Configure retry policies

## Success Criteria

- [ ] SSE streams update dashboard in real-time
- [ ] Booking events trigger notifications
- [ ] Emails delivered within 30 seconds
- [ ] Discord alerts appear for new bookings
- [ ] Failed notifications retry automatically
- [ ] Notification center shows unread count

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| SSE connection drops | Medium | Auto-reconnect logic |
| Email deliverability | Medium | Use verified sender, SPF/DKIM |
| Queue buildup | Medium | Monitor queue length, scale workers |

## Security Considerations

- SSE endpoints require authentication
- Validate tenant context for all events
- Rate limit notification sending
- Sanitize Discord message content
- Don't expose sensitive data in notifications

## Next Steps

After completion, proceed to [Phase 09: Monitoring & Logging](./phase-09-monitoring-logging-elk-prometheus-grafana.md)
