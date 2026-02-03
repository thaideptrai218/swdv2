---
title: "Phase 09: Monitoring & Logging"
status: pending
priority: P2
effort: 3d
---

# Phase 09: Monitoring & Logging - ELK, Prometheus, Grafana

## Context Links

- [Tech Architecture Report](../reports/researcher-260203-1204-tech-architecture.md)
- [Phase 08: Real-time & Notifications](./phase-08-real-time-sse-notifications-alerts.md)

## Overview

Implement comprehensive monitoring with Prometheus metrics, Grafana dashboards, and ELK stack for centralized logging. Set up alerting via Discord webhooks for critical issues.

## Key Insights

- ELK for centralized logs: searchable, structured, retention policies
- Prometheus for metrics: RED method (Rate, Errors, Duration)
- Grafana for visualization: business + operational dashboards
- Discord webhooks: free, instant, mobile-accessible alerts

## Requirements

### Functional
- Structured logging across all services
- Application metrics (requests, latency, errors)
- Business metrics (bookings, revenue, active users)
- Health check endpoints
- Grafana dashboards
- Alert rules with thresholds
- Log search and filtering

### Non-Functional
- Log ingestion <5 seconds
- Metrics scrape every 15 seconds
- 30-day log retention
- Dashboard load <3 seconds
- 99.9% monitoring uptime

## Architecture

### Monitoring Stack

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  NestJS API │────>│  Prometheus │────>│   Grafana   │
└─────────────┘     └─────────────┘     └─────────────┘
       │                                       │
       │            ┌─────────────┐            │
       └───────────>│   Logstash  │            │
                    └─────────────┘            │
                          │                    │
                    ┌─────────────┐            │
                    │Elasticsearch│<───────────┘
                    └─────────────┘
                          │
                    ┌─────────────┐
                    │   Kibana    │
                    └─────────────┘
```

### Log Flow

```
App → Winston Logger → Stdout (JSON) → Fluentd/Filebeat → Logstash → Elasticsearch
                                                                          ↓
                                                                       Kibana
```

### Metrics Categories

```
Application:
- http_requests_total (counter)
- http_request_duration_seconds (histogram)
- http_request_errors_total (counter)

Business:
- bookings_created_total
- bookings_revenue_total
- active_subscriptions
- payment_success_rate

Infrastructure:
- nodejs_heap_size_bytes
- database_connections_active
- redis_connected_clients
- rabbitmq_queue_messages
```

## Related Code Files

### Create
- `apps/api/src/common/logger/logger.service.ts`
- `apps/api/src/common/logger/logger.module.ts`
- `apps/api/src/common/metrics/metrics.service.ts`
- `apps/api/src/common/metrics/metrics.controller.ts`
- `apps/api/src/common/metrics/metrics.middleware.ts`
- `apps/api/src/common/health/health.controller.ts`
- `infra/docker/docker-compose.monitoring.yml`
- `infra/prometheus/prometheus.yml`
- `infra/grafana/dashboards/*.json`
- `infra/grafana/provisioning/datasources.yml`
- `infra/logstash/pipeline/*.conf`
- `infra/alertmanager/alertmanager.yml`

## Implementation Steps

### 1. Structured Logging (Day 1)

```typescript
// apps/api/src/common/logger/logger.service.ts
import { Injectable, LoggerService as NestLoggerService } from '@nestjs/common';
import { createLogger, format, transports, Logger } from 'winston';

interface LogContext {
  requestId?: string;
  userId?: string;
  tenantId?: string;
  method?: string;
  path?: string;
  [key: string]: any;
}

@Injectable()
export class LoggerService implements NestLoggerService {
  private logger: Logger;

  constructor() {
    this.logger = createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format: format.combine(
        format.timestamp(),
        format.errors({ stack: true }),
        format.json()
      ),
      defaultMeta: {
        service: 'hostel-api',
        version: process.env.APP_VERSION,
      },
      transports: [
        new transports.Console({
          format: process.env.NODE_ENV === 'development'
            ? format.combine(format.colorize(), format.simple())
            : format.json(),
        }),
      ],
    });
  }

  log(message: string, context?: LogContext) {
    this.logger.info(message, context);
  }

  error(message: string, trace?: string, context?: LogContext) {
    this.logger.error(message, { trace, ...context });
  }

  warn(message: string, context?: LogContext) {
    this.logger.warn(message, context);
  }

  debug(message: string, context?: LogContext) {
    this.logger.debug(message, context);
  }

  // Structured log methods
  logRequest(request: any, context: LogContext) {
    this.log('Incoming request', {
      ...context,
      method: request.method,
      path: request.url,
      userAgent: request.headers['user-agent'],
    });
  }

  logResponse(request: any, response: any, duration: number, context: LogContext) {
    this.log('Response sent', {
      ...context,
      method: request.method,
      path: request.url,
      statusCode: response.statusCode,
      duration,
    });
  }

  logBusinessEvent(event: string, data: any, context: LogContext) {
    this.log(event, {
      ...context,
      eventType: 'business',
      data,
    });
  }
}
```

### 2. Logging Interceptor (Day 1)

```typescript
// apps/api/src/common/logger/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { LoggerService } from './logger.service';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: LoggerService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const startTime = Date.now();

    const ctx = {
      requestId: request.id,
      userId: request.user?.id,
      tenantId: request.user?.tenantId,
    };

    this.logger.logRequest(request, ctx);

    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - startTime;
          this.logger.logResponse(request, response, duration, ctx);
        },
        error: (error) => {
          const duration = Date.now() - startTime;
          this.logger.error(error.message, error.stack, {
            ...ctx,
            duration,
            errorName: error.name,
          });
        },
      })
    );
  }
}
```

### 3. Prometheus Metrics (Day 1)

```typescript
// apps/api/src/common/metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import {
  collectDefaultMetrics,
  Counter,
  Histogram,
  Gauge,
  Registry,
} from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly registry: Registry;

  // HTTP metrics
  readonly httpRequestsTotal: Counter;
  readonly httpRequestDuration: Histogram;
  readonly httpErrorsTotal: Counter;

  // Business metrics
  readonly bookingsCreated: Counter;
  readonly bookingsRevenue: Counter;
  readonly activeSubscriptions: Gauge;
  readonly paymentSuccessRate: Gauge;

  // Infrastructure metrics
  readonly databaseConnections: Gauge;
  readonly queueMessages: Gauge;

  constructor() {
    this.registry = new Registry();

    // HTTP metrics
    this.httpRequestsTotal = new Counter({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'path', 'status'],
      registers: [this.registry],
    });

    this.httpRequestDuration = new Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration in seconds',
      labelNames: ['method', 'path', 'status'],
      buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
      registers: [this.registry],
    });

    this.httpErrorsTotal = new Counter({
      name: 'http_errors_total',
      help: 'Total HTTP errors',
      labelNames: ['method', 'path', 'error'],
      registers: [this.registry],
    });

    // Business metrics
    this.bookingsCreated = new Counter({
      name: 'bookings_created_total',
      help: 'Total bookings created',
      labelNames: ['tenant'],
      registers: [this.registry],
    });

    this.bookingsRevenue = new Counter({
      name: 'bookings_revenue_total',
      help: 'Total booking revenue in VND',
      labelNames: ['tenant'],
      registers: [this.registry],
    });

    this.activeSubscriptions = new Gauge({
      name: 'active_subscriptions',
      help: 'Number of active subscriptions',
      labelNames: ['plan'],
      registers: [this.registry],
    });

    this.paymentSuccessRate = new Gauge({
      name: 'payment_success_rate',
      help: 'Payment success rate (0-1)',
      registers: [this.registry],
    });

    // Infrastructure
    this.databaseConnections = new Gauge({
      name: 'database_connections_active',
      help: 'Active database connections',
      registers: [this.registry],
    });

    this.queueMessages = new Gauge({
      name: 'rabbitmq_queue_messages',
      help: 'Messages in RabbitMQ queues',
      labelNames: ['queue'],
      registers: [this.registry],
    });
  }

  onModuleInit() {
    collectDefaultMetrics({ register: this.registry });
  }

  async getMetrics(): Promise<string> {
    return this.registry.metrics();
  }

  // Helper methods
  recordRequest(method: string, path: string, status: number, duration: number) {
    const labels = { method, path: this.normalizePath(path), status: String(status) };
    this.httpRequestsTotal.inc(labels);
    this.httpRequestDuration.observe(labels, duration / 1000);

    if (status >= 400) {
      this.httpErrorsTotal.inc({ ...labels, error: status >= 500 ? 'server' : 'client' });
    }
  }

  recordBooking(tenantId: string, revenue: number) {
    this.bookingsCreated.inc({ tenant: tenantId });
    this.bookingsRevenue.inc({ tenant: tenantId }, revenue);
  }

  private normalizePath(path: string): string {
    // Replace UUIDs and IDs with placeholders
    return path
      .replace(/[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}/gi, ':id')
      .replace(/\/\d+/g, '/:id');
  }
}

// apps/api/src/common/metrics/metrics.controller.ts
import { Controller, Get, Header } from '@nestjs/common';
import { MetricsService } from './metrics.service';
import { Public } from '@/modules/auth/decorators/public.decorator';

@Controller('metrics')
export class MetricsController {
  constructor(private metricsService: MetricsService) {}

  @Public()
  @Get()
  @Header('Content-Type', 'text/plain')
  async getMetrics(): Promise<string> {
    return this.metricsService.getMetrics();
  }
}
```

### 4. Health Checks (Day 2)

```typescript
// apps/api/src/common/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  PrismaHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator,
} from '@nestjs/terminus';
import { Public } from '@/modules/auth/decorators/public.decorator';
import { PrismaService } from '@/database/prisma.service';
import { RedisHealthIndicator } from './redis.health';
import { RabbitMQHealthIndicator } from './rabbitmq.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private prisma: PrismaHealthIndicator,
    private prismaService: PrismaService,
    private redis: RedisHealthIndicator,
    private rabbitmq: RabbitMQHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Public()
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.prisma.pingCheck('database', this.prismaService),
      () => this.redis.isHealthy('redis'),
      () => this.rabbitmq.isHealthy('rabbitmq'),
      () => this.memory.checkHeap('memory_heap', 500 * 1024 * 1024), // 500MB
      () => this.disk.checkStorage('storage', { path: '/', thresholdPercent: 0.9 }),
    ]);
  }

  @Public()
  @Get('ready')
  @HealthCheck()
  readiness() {
    return this.health.check([
      () => this.prisma.pingCheck('database', this.prismaService),
      () => this.redis.isHealthy('redis'),
    ]);
  }

  @Public()
  @Get('live')
  liveness() {
    return { status: 'ok' };
  }
}
```

### 5. Docker Compose Monitoring (Day 2)

```yaml
# infra/docker/docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:v2.47.0
    volumes:
      - ../prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
    ports:
      - '9090:9090'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.1.0
    volumes:
      - grafana_data:/var/lib/grafana
      - ../grafana/provisioning:/etc/grafana/provisioning
      - ../grafana/dashboards:/var/lib/grafana/dashboards
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - '3001:3000'
    networks:
      - monitoring

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - 'ES_JAVA_OPTS=-Xms512m -Xmx512m'
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - '9200:9200'
    networks:
      - monitoring

  logstash:
    image: docker.elastic.co/logstash/logstash:8.10.0
    volumes:
      - ../logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - '5044:5044'
    networks:
      - monitoring
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - '5601:5601'
    networks:
      - monitoring
    depends_on:
      - elasticsearch

volumes:
  prometheus_data:
  grafana_data:
  elasticsearch_data:

networks:
  monitoring:
    driver: bridge
```

### 6. Prometheus Config (Day 2)

```yaml
# infra/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - '/etc/prometheus/rules/*.yml'

scrape_configs:
  - job_name: 'hostel-api'
    static_configs:
      - targets: ['api:3001']
    metrics_path: '/metrics'

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['redis-exporter:9121']
```

### 7. Alert Rules (Day 3)

```yaml
# infra/prometheus/rules/alerts.yml
groups:
  - name: hostel-api
    rules:
      - alert: HighErrorRate
        expr: rate(http_errors_total[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: 'High error rate detected'
          description: 'Error rate is {{ $value | printf "%.2f" }} errors/sec'

      - alert: SlowResponses
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: 'Slow response times detected'
          description: 'P95 latency is {{ $value | printf "%.2f" }}s'

      - alert: HighMemoryUsage
        expr: nodejs_heap_size_used_bytes / nodejs_heap_size_total_bytes > 0.9
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: 'High memory usage'
          description: 'Heap usage is {{ $value | printf "%.0f" }}%'

      - alert: DatabaseConnectionsHigh
        expr: database_connections_active > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: 'Database connections nearing limit'

      - alert: QueueBacklog
        expr: rabbitmq_queue_messages > 1000
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: 'Message queue backlog detected'
          description: 'Queue has {{ $value }} pending messages'

  - name: business
    rules:
      - alert: NoBookingsLastHour
        expr: increase(bookings_created_total[1h]) == 0
        for: 1h
        labels:
          severity: info
        annotations:
          summary: 'No bookings in the last hour'

      - alert: PaymentFailureSpike
        expr: rate(payment_failures_total[15m]) > rate(payment_success_total[15m]) * 0.3
        for: 15m
        labels:
          severity: critical
        annotations:
          summary: 'High payment failure rate'
```

### 8. Discord Alert Integration (Day 3)

```typescript
// apps/api/src/common/alerting/alert.service.ts
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { DiscordService } from '@/modules/notification/discord.service';

interface AlertPayload {
  alertname: string;
  severity: 'info' | 'warning' | 'critical';
  summary: string;
  description?: string;
}

@Injectable()
export class AlertService {
  constructor(private discord: DiscordService) {}

  @OnEvent('alert.fired')
  async handleAlert(payload: AlertPayload) {
    const colors = {
      info: 0x3498db,
      warning: 0xf39c12,
      critical: 0xe74c3c,
    };

    await this.discord.send({
      title: `Alert: ${payload.alertname}`,
      description: payload.description,
      color: colors[payload.severity],
      fields: [
        { name: 'Summary', value: payload.summary },
        { name: 'Severity', value: payload.severity.toUpperCase(), inline: true },
        { name: 'Time', value: new Date().toISOString(), inline: true },
      ],
    });
  }
}
```

## Todo List

- [ ] Create structured logger service
- [ ] Implement logging interceptor
- [ ] Create Prometheus metrics service
- [ ] Implement metrics middleware
- [ ] Create metrics controller
- [ ] Build health check endpoints
- [ ] Create Docker Compose for monitoring stack
- [ ] Configure Prometheus scraping
- [ ] Create Grafana dashboards
- [ ] Configure Logstash pipeline
- [ ] Set up Kibana index patterns
- [ ] Define alert rules
- [ ] Integrate Discord alerting
- [ ] Test end-to-end monitoring
- [ ] Document runbooks

## Success Criteria

- [ ] All logs appear in Elasticsearch within 5s
- [ ] Prometheus scrapes metrics every 15s
- [ ] Grafana dashboards load in <3s
- [ ] Health checks report accurate status
- [ ] Alerts fire correctly on thresholds
- [ ] Discord receives alert notifications

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Monitoring overhead | Low | Efficient sampling, async logging |
| Elasticsearch disk usage | Medium | Configure retention, index lifecycle |
| Alert fatigue | Medium | Tune thresholds, add silence rules |

## Security Considerations

- Metrics endpoint internal only (not public)
- Sanitize PII from logs
- Grafana auth required
- Elasticsearch authentication in production
- Encrypt log transport

## Next Steps

After completion, proceed to [Phase 10: Testing & Deployment](./phase-10-testing-deployment-cicd-production.md)
