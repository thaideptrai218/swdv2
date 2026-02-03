---
title: "Phase 05: Owner Dashboard"
status: pending
priority: P1
effort: 8d
---

# Phase 05: Owner Dashboard - Property, Booking, Reports UI

## Context Links

- [Competitor Analysis](../reports/researcher-260203-1204-hostel-saas-competitors.md)
- [Phase 04: Core API](./phase-04-core-api-trpc-routers-services.md)

## Overview

Build owner/staff dashboard for managing properties, rooms, beds, bookings, guests, and viewing reports. Responsive design with shadcn/ui components.

## Key Insights

- Dashboard is primary interaction point for paying customers
- Focus on booking calendar and availability at-a-glance
- Mobile-first: owners check bookings on phones
- Quick actions: check-in, check-out, cancel

## Requirements

### Functional
- Dashboard overview with key metrics
- Property list with CRUD operations
- Room/bed management with visual layout
- Booking calendar with drag-drop
- Booking list with filters and status updates
- Guest directory with booking history
- Reports: occupancy, revenue, bookings
- Staff user management (Owner only)
- Subscription status and billing

### Non-Functional
- Page load <2s
- Mobile responsive (all features)
- Offline indicator
- Real-time booking updates (SSE)

## Architecture

### Page Structure

```
/dashboard
├── /                     # Overview with metrics
├── /properties           # Property list
├── /properties/[id]      # Property detail + rooms
├── /properties/new       # Add property
├── /rooms/[id]           # Room detail + beds
├── /bookings             # Booking list
├── /bookings/[id]        # Booking detail
├── /bookings/calendar    # Calendar view
├── /guests               # Guest directory
├── /guests/[id]          # Guest profile
├── /reports              # Analytics
├── /settings             # Tenant settings
├── /team                 # Staff management
└── /billing              # Subscription
```

### Component Hierarchy

```
DashboardLayout
├── Sidebar (navigation)
├── Header (user menu, notifications)
└── Content
    ├── PageHeader (title, actions)
    └── PageContent (data tables, forms, charts)
```

## Related Code Files

### Create
- `apps/web/src/app/(dashboard)/layout.tsx`
- `apps/web/src/app/(dashboard)/page.tsx`
- `apps/web/src/app/(dashboard)/properties/page.tsx`
- `apps/web/src/app/(dashboard)/properties/[id]/page.tsx`
- `apps/web/src/app/(dashboard)/properties/new/page.tsx`
- `apps/web/src/app/(dashboard)/bookings/page.tsx`
- `apps/web/src/app/(dashboard)/bookings/[id]/page.tsx`
- `apps/web/src/app/(dashboard)/bookings/calendar/page.tsx`
- `apps/web/src/app/(dashboard)/guests/page.tsx`
- `apps/web/src/app/(dashboard)/reports/page.tsx`
- `apps/web/src/app/(dashboard)/settings/page.tsx`
- `apps/web/src/app/(dashboard)/team/page.tsx`
- `apps/web/src/app/(dashboard)/billing/page.tsx`
- `apps/web/src/components/dashboard/sidebar.tsx`
- `apps/web/src/components/dashboard/header.tsx`
- `apps/web/src/components/properties/property-card.tsx`
- `apps/web/src/components/properties/property-form.tsx`
- `apps/web/src/components/bookings/booking-table.tsx`
- `apps/web/src/components/bookings/booking-calendar.tsx`
- `apps/web/src/components/bookings/booking-status-badge.tsx`
- `apps/web/src/components/reports/occupancy-chart.tsx`
- `apps/web/src/components/reports/revenue-chart.tsx`

## Implementation Steps

### 1. Dashboard Layout (Day 1)

```typescript
// apps/web/src/app/(dashboard)/layout.tsx
import { Sidebar } from '@/components/dashboard/sidebar';
import { Header } from '@/components/dashboard/header';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex flex-1 flex-col overflow-hidden">
        <Header />
        <main className="flex-1 overflow-y-auto bg-gray-50 p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

### 2. Sidebar Navigation (Day 1)

```typescript
// apps/web/src/components/dashboard/sidebar.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import {
  LayoutDashboard,
  Building2,
  CalendarDays,
  Users,
  BarChart3,
  Settings,
  UserCog,
  CreditCard,
} from 'lucide-react';
import { cn } from '@/lib/utils';

const navigation = [
  { name: 'Overview', href: '/dashboard', icon: LayoutDashboard },
  { name: 'Properties', href: '/dashboard/properties', icon: Building2 },
  { name: 'Bookings', href: '/dashboard/bookings', icon: CalendarDays },
  { name: 'Guests', href: '/dashboard/guests', icon: Users },
  { name: 'Reports', href: '/dashboard/reports', icon: BarChart3 },
  { name: 'Team', href: '/dashboard/team', icon: UserCog },
  { name: 'Billing', href: '/dashboard/billing', icon: CreditCard },
  { name: 'Settings', href: '/dashboard/settings', icon: Settings },
];

export function Sidebar() {
  const pathname = usePathname();

  return (
    <aside className="hidden w-64 border-r bg-white lg:block">
      <div className="flex h-16 items-center border-b px-6">
        <span className="text-xl font-bold">HostelHub</span>
      </div>
      <nav className="space-y-1 p-4">
        {navigation.map((item) => (
          <Link
            key={item.name}
            href={item.href}
            className={cn(
              'flex items-center gap-3 rounded-lg px-3 py-2 text-sm',
              pathname === item.href
                ? 'bg-primary text-primary-foreground'
                : 'text-muted-foreground hover:bg-muted'
            )}
          >
            <item.icon className="h-5 w-5" />
            {item.name}
          </Link>
        ))}
      </nav>
    </aside>
  );
}
```

### 3. Dashboard Overview (Day 2)

```typescript
// apps/web/src/app/(dashboard)/page.tsx
import { trpc } from '@/trpc/server';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import {
  Building2,
  BedDouble,
  CalendarCheck,
  TrendingUp
} from 'lucide-react';

export default async function DashboardPage() {
  const stats = await trpc.stats.dashboard();

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">Dashboard</h1>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <StatCard
          title="Properties"
          value={stats.totalProperties}
          icon={Building2}
        />
        <StatCard
          title="Total Beds"
          value={stats.totalBeds}
          icon={BedDouble}
        />
        <StatCard
          title="Active Bookings"
          value={stats.activeBookings}
          icon={CalendarCheck}
        />
        <StatCard
          title="Occupancy Rate"
          value={`${stats.occupancyRate}%`}
          icon={TrendingUp}
        />
      </div>

      <div className="grid gap-6 lg:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle>Today's Check-ins</CardTitle>
          </CardHeader>
          <CardContent>
            <CheckInList bookings={stats.todayCheckIns} />
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Today's Check-outs</CardTitle>
          </CardHeader>
          <CardContent>
            <CheckOutList bookings={stats.todayCheckOuts} />
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

### 4. Property Management (Day 2-3)

```typescript
// apps/web/src/app/(dashboard)/properties/page.tsx
'use client';

import { trpc } from '@/trpc/client';
import { PropertyCard } from '@/components/properties/property-card';
import { Button } from '@/components/ui/button';
import { Plus } from 'lucide-react';
import Link from 'next/link';

export default function PropertiesPage() {
  const { data: properties, isLoading } = trpc.property.list.useQuery();

  if (isLoading) return <LoadingSkeleton />;

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Properties</h1>
        <Button asChild>
          <Link href="/dashboard/properties/new">
            <Plus className="mr-2 h-4 w-4" />
            Add Property
          </Link>
        </Button>
      </div>

      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {properties?.map((property) => (
          <PropertyCard key={property.id} property={property} />
        ))}
      </div>
    </div>
  );
}

// apps/web/src/components/properties/property-card.tsx
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { BedDouble, MapPin } from 'lucide-react';
import Link from 'next/link';

export function PropertyCard({ property }: { property: Property }) {
  const totalBeds = property.rooms.reduce(
    (sum, room) => sum + room.beds.length,
    0
  );

  return (
    <Link href={`/dashboard/properties/${property.id}`}>
      <Card className="hover:border-primary transition-colors">
        <CardHeader>
          <CardTitle className="flex items-center justify-between">
            {property.name}
            <Badge variant={property.isActive ? 'default' : 'secondary'}>
              {property.isActive ? 'Active' : 'Inactive'}
            </Badge>
          </CardTitle>
        </CardHeader>
        <CardContent>
          <div className="space-y-2 text-sm text-muted-foreground">
            <div className="flex items-center gap-2">
              <MapPin className="h-4 w-4" />
              {property.city}
            </div>
            <div className="flex items-center gap-2">
              <BedDouble className="h-4 w-4" />
              {property.rooms.length} rooms, {totalBeds} beds
            </div>
          </div>
        </CardContent>
      </Card>
    </Link>
  );
}
```

### 5. Booking Calendar (Day 4-5)

```typescript
// apps/web/src/components/bookings/booking-calendar.tsx
'use client';

import { useState } from 'react';
import { trpc } from '@/trpc/client';
import {
  startOfMonth,
  endOfMonth,
  eachDayOfInterval,
  format,
  isSameDay,
  isWithinInterval,
} from 'date-fns';

export function BookingCalendar() {
  const [currentMonth, setCurrentMonth] = useState(new Date());

  const { data: bookings } = trpc.booking.list.useQuery({
    from: startOfMonth(currentMonth),
    to: endOfMonth(currentMonth),
  });

  const days = eachDayOfInterval({
    start: startOfMonth(currentMonth),
    end: endOfMonth(currentMonth),
  });

  return (
    <div className="rounded-lg border bg-white">
      <div className="flex items-center justify-between border-b p-4">
        <button onClick={() => setCurrentMonth(addMonths(currentMonth, -1))}>
          Previous
        </button>
        <h2 className="font-semibold">
          {format(currentMonth, 'MMMM yyyy')}
        </h2>
        <button onClick={() => setCurrentMonth(addMonths(currentMonth, 1))}>
          Next
        </button>
      </div>

      <div className="grid grid-cols-7">
        {['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'].map((day) => (
          <div key={day} className="border-b p-2 text-center text-sm font-medium">
            {day}
          </div>
        ))}

        {days.map((day) => {
          const dayBookings = bookings?.filter((b) =>
            isWithinInterval(day, { start: b.checkIn, end: b.checkOut })
          );

          return (
            <div
              key={day.toISOString()}
              className="min-h-24 border-b border-r p-1"
            >
              <div className="text-sm text-muted-foreground">
                {format(day, 'd')}
              </div>
              <div className="space-y-1">
                {dayBookings?.slice(0, 3).map((booking) => (
                  <div
                    key={booking.id}
                    className="truncate rounded bg-primary/10 px-1 text-xs"
                  >
                    {booking.guest.name}
                  </div>
                ))}
                {dayBookings && dayBookings.length > 3 && (
                  <div className="text-xs text-muted-foreground">
                    +{dayBookings.length - 3} more
                  </div>
                )}
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

### 6. Booking List with Actions (Day 5-6)

```typescript
// apps/web/src/components/bookings/booking-table.tsx
'use client';

import { trpc } from '@/trpc/client';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { BookingStatusBadge } from './booking-status-badge';
import { BookingActions } from './booking-actions';
import { format } from 'date-fns';

export function BookingTable({ status }: { status?: string }) {
  const { data: bookings, refetch } = trpc.booking.list.useQuery({ status });
  const updateStatus = trpc.booking.updateStatus.useMutation({
    onSuccess: () => refetch(),
  });

  const handleStatusChange = (id: string, newStatus: string) => {
    updateStatus.mutate({ id, status: newStatus });
  };

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Guest</TableHead>
          <TableHead>Room/Beds</TableHead>
          <TableHead>Check-in</TableHead>
          <TableHead>Check-out</TableHead>
          <TableHead>Total</TableHead>
          <TableHead>Status</TableHead>
          <TableHead>Actions</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {bookings?.map((booking) => (
          <TableRow key={booking.id}>
            <TableCell>
              <div>
                <div className="font-medium">{booking.guest.name}</div>
                <div className="text-sm text-muted-foreground">
                  {booking.guest.email}
                </div>
              </div>
            </TableCell>
            <TableCell>
              {booking.bookingBeds.map((bb) => bb.bed.name).join(', ')}
            </TableCell>
            <TableCell>{format(booking.checkIn, 'MMM d, yyyy')}</TableCell>
            <TableCell>{format(booking.checkOut, 'MMM d, yyyy')}</TableCell>
            <TableCell>
              {new Intl.NumberFormat('vi-VN', {
                style: 'currency',
                currency: 'VND',
              }).format(Number(booking.totalPrice))}
            </TableCell>
            <TableCell>
              <BookingStatusBadge status={booking.status} />
            </TableCell>
            <TableCell>
              <BookingActions
                booking={booking}
                onStatusChange={handleStatusChange}
              />
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

### 7. Reports Page (Day 7)

```typescript
// apps/web/src/app/(dashboard)/reports/page.tsx
'use client';

import { trpc } from '@/trpc/client';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { OccupancyChart } from '@/components/reports/occupancy-chart';
import { RevenueChart } from '@/components/reports/revenue-chart';
import { DateRangePicker } from '@/components/ui/date-range-picker';
import { useState } from 'react';
import { subDays } from 'date-fns';

export default function ReportsPage() {
  const [dateRange, setDateRange] = useState({
    from: subDays(new Date(), 30),
    to: new Date(),
  });

  const { data: reports } = trpc.stats.reports.useQuery({
    from: dateRange.from,
    to: dateRange.to,
  });

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Reports</h1>
        <DateRangePicker
          value={dateRange}
          onChange={setDateRange}
        />
      </div>

      <div className="grid gap-6 lg:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle>Occupancy Rate</CardTitle>
          </CardHeader>
          <CardContent>
            <OccupancyChart data={reports?.occupancy} />
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Revenue</CardTitle>
          </CardHeader>
          <CardContent>
            <RevenueChart data={reports?.revenue} />
          </CardContent>
        </Card>
      </div>

      <Card>
        <CardHeader>
          <CardTitle>Booking Summary</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="grid gap-4 md:grid-cols-4">
            <div>
              <div className="text-2xl font-bold">{reports?.totalBookings}</div>
              <div className="text-sm text-muted-foreground">Total Bookings</div>
            </div>
            <div>
              <div className="text-2xl font-bold">{reports?.completedBookings}</div>
              <div className="text-sm text-muted-foreground">Completed</div>
            </div>
            <div>
              <div className="text-2xl font-bold">{reports?.cancelledBookings}</div>
              <div className="text-sm text-muted-foreground">Cancelled</div>
            </div>
            <div>
              <div className="text-2xl font-bold">
                {new Intl.NumberFormat('vi-VN', {
                  style: 'currency',
                  currency: 'VND',
                  maximumFractionDigits: 0,
                }).format(reports?.totalRevenue || 0)}
              </div>
              <div className="text-sm text-muted-foreground">Total Revenue</div>
            </div>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
```

### 8. Team Management (Day 8)

```typescript
// apps/web/src/app/(dashboard)/team/page.tsx
'use client';

import { trpc } from '@/trpc/client';
import { Button } from '@/components/ui/button';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Badge } from '@/components/ui/badge';
import { InviteUserDialog } from '@/components/team/invite-user-dialog';
import { useState } from 'react';

export default function TeamPage() {
  const { data: users, refetch } = trpc.user.list.useQuery();
  const [showInvite, setShowInvite] = useState(false);

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Team</h1>
        <Button onClick={() => setShowInvite(true)}>
          Invite Member
        </Button>
      </div>

      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Name</TableHead>
            <TableHead>Email</TableHead>
            <TableHead>Role</TableHead>
            <TableHead>Status</TableHead>
            <TableHead>Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {users?.map((user) => (
            <TableRow key={user.id}>
              <TableCell className="font-medium">{user.name}</TableCell>
              <TableCell>{user.email}</TableCell>
              <TableCell>
                <Badge variant="outline">{user.role}</Badge>
              </TableCell>
              <TableCell>
                <Badge variant={user.isActive ? 'default' : 'secondary'}>
                  {user.isActive ? 'Active' : 'Inactive'}
                </Badge>
              </TableCell>
              <TableCell>
                {/* Edit/Delete actions */}
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>

      <InviteUserDialog
        open={showInvite}
        onClose={() => setShowInvite(false)}
        onSuccess={refetch}
      />
    </div>
  );
}
```

## Todo List

- [ ] Create dashboard layout with sidebar
- [ ] Build sidebar navigation component
- [ ] Build header with user menu
- [ ] Create dashboard overview with stats
- [ ] Build property list page
- [ ] Build property detail page with rooms
- [ ] Create property form (add/edit)
- [ ] Build room management UI
- [ ] Create booking calendar component
- [ ] Build booking list with filters
- [ ] Create booking detail page
- [ ] Build booking status actions
- [ ] Create guest directory page
- [ ] Build reports page with charts
- [ ] Create team management page
- [ ] Build billing/subscription page
- [ ] Add mobile responsive styles
- [ ] Implement SSE for real-time updates

## Success Criteria

- [ ] All dashboard pages render without errors
- [ ] Property CRUD operations work
- [ ] Booking calendar displays correctly
- [ ] Booking status updates reflect immediately
- [ ] Reports show accurate data
- [ ] Mobile layout is usable
- [ ] Page transitions are smooth (<200ms)

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Complex calendar performance | Medium | Virtual scrolling, memoization |
| Form validation errors | Low | Zod schemas, error boundaries |
| Mobile UX issues | Medium | Test on real devices early |

## Security Considerations

- Role-based UI rendering (hide admin features from staff)
- Validate user can access requested resources
- Sanitize displayed user content
- Rate limit API calls from UI

## Next Steps

After completion, proceed to [Phase 06: Guest Platform](./phase-06-guest-platform-search-listing-booking-ui.md)
