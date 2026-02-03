---
title: "Phase 06: Guest Platform"
status: pending
priority: P1
effort: 6d
---

# Phase 06: Guest Platform - Search, Listing, Booking UI

## Context Links

- [Competitor Analysis](../reports/researcher-260203-1204-hostel-saas-competitors.md)
- [Phase 05: Owner Dashboard](./phase-05-owner-dashboard-property-booking-reports-ui.md)

## Overview

Build public-facing guest platform for searching hostels, viewing listings, checking availability, and booking beds. Include guest authentication and booking history.

## Key Insights

- SEO critical for guest discovery (SSR required)
- Mobile-first: 70%+ traffic from phones
- Quick booking flow reduces abandonment
- Reviews and photos drive conversions

## Requirements

### Functional
- City/location search with autocomplete
- Date range and guest count filters
- Property listing with filters (price, amenities, room type)
- Property detail with photos, amenities, reviews
- Real-time availability check
- Bed selection and booking flow
- Guest registration/login
- Booking confirmation and history
- Review submission after stay

### Non-Functional
- SSR for SEO (property pages)
- LCP <2.5s on mobile
- Search results <1s
- Booking flow <3 steps

## Architecture

### Page Structure

```
/                         # Landing with search
/search                   # Search results
/hostels/[slug]           # Property detail
/hostels/[slug]/book      # Booking flow
/account                  # Guest account
├── /bookings             # Booking history
├── /profile              # Profile settings
└── /reviews              # My reviews
/login                    # Guest login
/register                 # Guest registration
```

### Search Flow

```
1. User enters city, dates, guests
2. Server fetches available properties (SSR or client)
3. Display results with availability indicators
4. Filter/sort on client
5. Click → Property detail page
6. Check specific room/bed availability
7. Select beds → Booking flow
```

## Related Code Files

### Create
- `apps/web/src/app/(public)/page.tsx`
- `apps/web/src/app/(public)/search/page.tsx`
- `apps/web/src/app/(public)/hostels/[slug]/page.tsx`
- `apps/web/src/app/(public)/hostels/[slug]/book/page.tsx`
- `apps/web/src/app/(guest)/account/page.tsx`
- `apps/web/src/app/(guest)/account/bookings/page.tsx`
- `apps/web/src/app/(guest)/account/profile/page.tsx`
- `apps/web/src/app/(auth)/login/page.tsx`
- `apps/web/src/app/(auth)/register/page.tsx`
- `apps/web/src/components/search/search-form.tsx`
- `apps/web/src/components/search/search-results.tsx`
- `apps/web/src/components/search/property-card.tsx`
- `apps/web/src/components/search/filters.tsx`
- `apps/web/src/components/property/property-gallery.tsx`
- `apps/web/src/components/property/property-info.tsx`
- `apps/web/src/components/property/room-list.tsx`
- `apps/web/src/components/property/reviews.tsx`
- `apps/web/src/components/booking/bed-selector.tsx`
- `apps/web/src/components/booking/booking-summary.tsx`
- `apps/web/src/components/booking/payment-form.tsx`

## Implementation Steps

### 1. Landing Page with Search (Day 1)

```typescript
// apps/web/src/app/(public)/page.tsx
import { SearchForm } from '@/components/search/search-form';

export default function HomePage() {
  return (
    <div className="relative">
      {/* Hero Section */}
      <section className="relative h-[70vh] bg-gradient-to-b from-primary/20 to-background">
        <div className="container flex h-full flex-col items-center justify-center">
          <h1 className="text-4xl font-bold tracking-tight sm:text-6xl">
            Find Your Perfect Hostel in Vietnam
          </h1>
          <p className="mt-4 text-lg text-muted-foreground">
            Book beds in the best hostels across Ho Chi Minh, Hanoi, Da Nang, and more
          </p>

          <div className="mt-8 w-full max-w-4xl">
            <SearchForm />
          </div>
        </div>
      </section>

      {/* Featured Cities */}
      <section className="container py-16">
        <h2 className="text-2xl font-bold">Popular Destinations</h2>
        <div className="mt-6 grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
          <CityCard city="Ho Chi Minh City" count={45} />
          <CityCard city="Hanoi" count={38} />
          <CityCard city="Da Nang" count={22} />
          <CityCard city="Nha Trang" count={18} />
        </div>
      </section>
    </div>
  );
}

// apps/web/src/components/search/search-form.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { DatePickerWithRange } from '@/components/ui/date-range-picker';
import { Search, MapPin, Users } from 'lucide-react';

export function SearchForm() {
  const router = useRouter();
  const [city, setCity] = useState('');
  const [dates, setDates] = useState<{ from?: Date; to?: Date }>({});
  const [guests, setGuests] = useState(1);

  const handleSearch = () => {
    const params = new URLSearchParams({
      city,
      checkIn: dates.from?.toISOString() || '',
      checkOut: dates.to?.toISOString() || '',
      guests: guests.toString(),
    });
    router.push(`/search?${params}`);
  };

  return (
    <div className="flex flex-col gap-4 rounded-lg bg-white p-4 shadow-lg sm:flex-row">
      <div className="flex flex-1 items-center gap-2 rounded-md border px-3">
        <MapPin className="h-5 w-5 text-muted-foreground" />
        <Input
          placeholder="Where are you going?"
          value={city}
          onChange={(e) => setCity(e.target.value)}
          className="border-0 focus-visible:ring-0"
        />
      </div>

      <DatePickerWithRange
        value={dates}
        onChange={setDates}
        className="flex-1"
      />

      <div className="flex items-center gap-2 rounded-md border px-3">
        <Users className="h-5 w-5 text-muted-foreground" />
        <Input
          type="number"
          min={1}
          max={20}
          value={guests}
          onChange={(e) => setGuests(parseInt(e.target.value))}
          className="w-16 border-0 focus-visible:ring-0"
        />
        <span className="text-sm text-muted-foreground">guests</span>
      </div>

      <Button onClick={handleSearch} size="lg">
        <Search className="mr-2 h-5 w-5" />
        Search
      </Button>
    </div>
  );
}
```

### 2. Search Results Page (Day 2)

```typescript
// apps/web/src/app/(public)/search/page.tsx
import { trpc } from '@/trpc/server';
import { SearchResults } from '@/components/search/search-results';
import { SearchFilters } from '@/components/search/filters';

interface SearchPageProps {
  searchParams: {
    city?: string;
    checkIn?: string;
    checkOut?: string;
    guests?: string;
    roomType?: string;
    minPrice?: string;
    maxPrice?: string;
  };
}

export default async function SearchPage({ searchParams }: SearchPageProps) {
  const results = await trpc.availability.search({
    city: searchParams.city,
    checkIn: new Date(searchParams.checkIn || Date.now()),
    checkOut: new Date(searchParams.checkOut || Date.now() + 86400000),
    guests: parseInt(searchParams.guests || '1'),
    roomType: searchParams.roomType,
    minPrice: searchParams.minPrice ? parseInt(searchParams.minPrice) : undefined,
    maxPrice: searchParams.maxPrice ? parseInt(searchParams.maxPrice) : undefined,
  });

  return (
    <div className="container py-8">
      <div className="flex gap-8">
        <aside className="hidden w-64 lg:block">
          <SearchFilters />
        </aside>

        <main className="flex-1">
          <div className="mb-6 flex items-center justify-between">
            <h1 className="text-xl font-semibold">
              {results.length} hostels in {searchParams.city || 'Vietnam'}
            </h1>
            <SortDropdown />
          </div>

          <SearchResults properties={results} />
        </main>
      </div>
    </div>
  );
}

// apps/web/src/components/search/search-results.tsx
import { PropertyCard } from './property-card';

export function SearchResults({ properties }: { properties: any[] }) {
  if (properties.length === 0) {
    return (
      <div className="py-16 text-center">
        <p className="text-lg text-muted-foreground">
          No hostels found matching your criteria
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {properties.map((property) => (
        <PropertyCard key={property.id} property={property} />
      ))}
    </div>
  );
}
```

### 3. Property Detail Page (Day 2-3)

```typescript
// apps/web/src/app/(public)/hostels/[slug]/page.tsx
import { trpc } from '@/trpc/server';
import { notFound } from 'next/navigation';
import { PropertyGallery } from '@/components/property/property-gallery';
import { PropertyInfo } from '@/components/property/property-info';
import { RoomList } from '@/components/property/room-list';
import { Reviews } from '@/components/property/reviews';
import { BookingSidebar } from '@/components/property/booking-sidebar';

export async function generateMetadata({ params }: { params: { slug: string } }) {
  const property = await trpc.property.getBySlug({ slug: params.slug });
  if (!property) return {};

  return {
    title: `${property.name} - HostelHub`,
    description: property.description,
    openGraph: {
      images: property.images[0],
    },
  };
}

export default async function PropertyPage({ params }: { params: { slug: string } }) {
  const property = await trpc.property.getBySlug({ slug: params.slug });

  if (!property) notFound();

  return (
    <div className="container py-8">
      <PropertyGallery images={property.images} />

      <div className="mt-8 grid gap-8 lg:grid-cols-3">
        <div className="lg:col-span-2">
          <PropertyInfo property={property} />

          <section className="mt-8">
            <h2 className="text-xl font-semibold">Rooms & Beds</h2>
            <RoomList rooms={property.rooms} />
          </section>

          <section className="mt-8">
            <h2 className="text-xl font-semibold">Reviews</h2>
            <Reviews propertyId={property.id} />
          </section>
        </div>

        <aside className="lg:col-span-1">
          <BookingSidebar property={property} />
        </aside>
      </div>
    </div>
  );
}

// apps/web/src/components/property/room-list.tsx
import { Badge } from '@/components/ui/badge';
import { BedDouble, Users } from 'lucide-react';

export function RoomList({ rooms }: { rooms: Room[] }) {
  return (
    <div className="mt-4 space-y-4">
      {rooms.map((room) => (
        <div key={room.id} className="rounded-lg border p-4">
          <div className="flex items-start justify-between">
            <div>
              <h3 className="font-medium">{room.name}</h3>
              <div className="mt-1 flex items-center gap-4 text-sm text-muted-foreground">
                <span className="flex items-center gap-1">
                  <BedDouble className="h-4 w-4" />
                  {room.beds.length} beds
                </span>
                <span className="flex items-center gap-1">
                  <Users className="h-4 w-4" />
                  {room.capacity} guests
                </span>
              </div>
            </div>
            <div className="text-right">
              <div className="text-lg font-semibold">
                {formatPrice(room.basePrice)}
              </div>
              <div className="text-sm text-muted-foreground">per night</div>
            </div>
          </div>

          <div className="mt-4 flex flex-wrap gap-2">
            {room.amenities.map((amenity) => (
              <Badge key={amenity} variant="secondary">
                {amenity}
              </Badge>
            ))}
          </div>
        </div>
      ))}
    </div>
  );
}
```

### 4. Booking Flow (Day 3-4)

```typescript
// apps/web/src/app/(public)/hostels/[slug]/book/page.tsx
'use client';

import { useState } from 'react';
import { useSearchParams } from 'next/navigation';
import { trpc } from '@/trpc/client';
import { BedSelector } from '@/components/booking/bed-selector';
import { BookingSummary } from '@/components/booking/booking-summary';
import { GuestForm } from '@/components/booking/guest-form';
import { PaymentForm } from '@/components/booking/payment-form';
import { Button } from '@/components/ui/button';

type Step = 'beds' | 'guest' | 'payment' | 'confirmation';

export default function BookingPage({ params }: { params: { slug: string } }) {
  const searchParams = useSearchParams();
  const [step, setStep] = useState<Step>('beds');
  const [selectedBeds, setSelectedBeds] = useState<string[]>([]);
  const [guestData, setGuestData] = useState<any>(null);

  const checkIn = new Date(searchParams.get('checkIn') || '');
  const checkOut = new Date(searchParams.get('checkOut') || '');

  const { data: availability } = trpc.availability.check.useQuery({
    propertySlug: params.slug,
    checkIn,
    checkOut,
  });

  const createBooking = trpc.booking.create.useMutation({
    onSuccess: () => setStep('confirmation'),
  });

  const handleConfirmBooking = async () => {
    await createBooking.mutateAsync({
      bedIds: selectedBeds,
      checkIn,
      checkOut,
      guestId: guestData.id,
    });
  };

  return (
    <div className="container max-w-4xl py-8">
      <BookingProgress step={step} />

      <div className="mt-8 grid gap-8 lg:grid-cols-3">
        <div className="lg:col-span-2">
          {step === 'beds' && (
            <BedSelector
              rooms={availability?.rooms || []}
              selectedBeds={selectedBeds}
              onSelect={setSelectedBeds}
              onContinue={() => setStep('guest')}
            />
          )}

          {step === 'guest' && (
            <GuestForm
              onSubmit={(data) => {
                setGuestData(data);
                setStep('payment');
              }}
              onBack={() => setStep('beds')}
            />
          )}

          {step === 'payment' && (
            <PaymentForm
              onConfirm={handleConfirmBooking}
              onBack={() => setStep('guest')}
              isLoading={createBooking.isPending}
            />
          )}

          {step === 'confirmation' && (
            <BookingConfirmation bookingId={createBooking.data?.id} />
          )}
        </div>

        <aside>
          <BookingSummary
            selectedBeds={selectedBeds}
            checkIn={checkIn}
            checkOut={checkOut}
            rooms={availability?.rooms || []}
          />
        </aside>
      </div>
    </div>
  );
}

// apps/web/src/components/booking/bed-selector.tsx
export function BedSelector({
  rooms,
  selectedBeds,
  onSelect,
  onContinue,
}: BedSelectorProps) {
  const toggleBed = (bedId: string) => {
    if (selectedBeds.includes(bedId)) {
      onSelect(selectedBeds.filter((id) => id !== bedId));
    } else {
      onSelect([...selectedBeds, bedId]);
    }
  };

  return (
    <div className="space-y-6">
      <h2 className="text-xl font-semibold">Select Your Beds</h2>

      {rooms.map((room) => (
        <div key={room.id} className="rounded-lg border p-4">
          <h3 className="font-medium">{room.name}</h3>

          <div className="mt-4 grid grid-cols-4 gap-2">
            {room.beds.map((bed) => (
              <button
                key={bed.id}
                onClick={() => bed.isAvailable && toggleBed(bed.id)}
                disabled={!bed.isAvailable}
                className={cn(
                  'rounded-lg border p-3 text-center transition-colors',
                  selectedBeds.includes(bed.id)
                    ? 'border-primary bg-primary/10'
                    : bed.isAvailable
                    ? 'hover:border-primary'
                    : 'cursor-not-allowed bg-muted opacity-50'
                )}
              >
                <div className="text-sm font-medium">{bed.name}</div>
                <div className="text-xs text-muted-foreground">
                  {formatPrice(bed.pricePerNight)}
                </div>
              </button>
            ))}
          </div>
        </div>
      ))}

      <Button
        onClick={onContinue}
        disabled={selectedBeds.length === 0}
        className="w-full"
      >
        Continue ({selectedBeds.length} beds selected)
      </Button>
    </div>
  );
}
```

### 5. Guest Account Pages (Day 5)

```typescript
// apps/web/src/app/(guest)/account/bookings/page.tsx
'use client';

import { trpc } from '@/trpc/client';
import { BookingCard } from '@/components/account/booking-card';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';

export default function MyBookingsPage() {
  const { data: bookings } = trpc.guest.myBookings.useQuery();

  const upcoming = bookings?.filter((b) =>
    new Date(b.checkIn) > new Date() && b.status !== 'CANCELLED'
  );
  const past = bookings?.filter((b) =>
    new Date(b.checkOut) < new Date() || b.status === 'CANCELLED'
  );

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">My Bookings</h1>

      <Tabs defaultValue="upcoming">
        <TabsList>
          <TabsTrigger value="upcoming">
            Upcoming ({upcoming?.length || 0})
          </TabsTrigger>
          <TabsTrigger value="past">
            Past ({past?.length || 0})
          </TabsTrigger>
        </TabsList>

        <TabsContent value="upcoming" className="mt-4">
          {upcoming?.length === 0 ? (
            <p className="text-muted-foreground">No upcoming bookings</p>
          ) : (
            <div className="space-y-4">
              {upcoming?.map((booking) => (
                <BookingCard key={booking.id} booking={booking} />
              ))}
            </div>
          )}
        </TabsContent>

        <TabsContent value="past" className="mt-4">
          <div className="space-y-4">
            {past?.map((booking) => (
              <BookingCard key={booking.id} booking={booking} showReview />
            ))}
          </div>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### 6. Guest Auth Pages (Day 6)

```typescript
// apps/web/src/app/(auth)/register/page.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { trpc } from '@/trpc/client';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import Link from 'next/link';

export default function RegisterPage() {
  const router = useRouter();
  const [form, setForm] = useState({
    email: '',
    password: '',
    name: '',
    phone: '',
  });

  const register = trpc.auth.guestRegister.useMutation({
    onSuccess: () => router.push('/account'),
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    register.mutate(form);
  };

  return (
    <div className="container flex min-h-screen items-center justify-center">
      <div className="w-full max-w-md space-y-6">
        <div className="text-center">
          <h1 className="text-2xl font-bold">Create Account</h1>
          <p className="text-muted-foreground">
            Sign up to book hostels and manage your trips
          </p>
        </div>

        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="name">Full Name</Label>
            <Input
              id="name"
              value={form.name}
              onChange={(e) => setForm({ ...form, name: e.target.value })}
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="email">Email</Label>
            <Input
              id="email"
              type="email"
              value={form.email}
              onChange={(e) => setForm({ ...form, email: e.target.value })}
              required
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="password">Password</Label>
            <Input
              id="password"
              type="password"
              value={form.password}
              onChange={(e) => setForm({ ...form, password: e.target.value })}
              required
              minLength={8}
            />
          </div>

          <div className="space-y-2">
            <Label htmlFor="phone">Phone (optional)</Label>
            <Input
              id="phone"
              type="tel"
              value={form.phone}
              onChange={(e) => setForm({ ...form, phone: e.target.value })}
            />
          </div>

          <Button type="submit" className="w-full" disabled={register.isPending}>
            {register.isPending ? 'Creating account...' : 'Create Account'}
          </Button>
        </form>

        <p className="text-center text-sm text-muted-foreground">
          Already have an account?{' '}
          <Link href="/login" className="text-primary hover:underline">
            Sign in
          </Link>
        </p>
      </div>
    </div>
  );
}
```

## Todo List

- [ ] Create landing page with hero search
- [ ] Build search form with city autocomplete
- [ ] Build search results page with SSR
- [ ] Create property card component
- [ ] Build search filters sidebar
- [ ] Create property detail page with SSR
- [ ] Build property gallery component
- [ ] Build room list component
- [ ] Create reviews section
- [ ] Build booking sidebar
- [ ] Create bed selector component
- [ ] Build guest info form
- [ ] Create payment form (integrate Phase 07)
- [ ] Build booking confirmation page
- [ ] Create guest account layout
- [ ] Build my bookings page
- [ ] Build profile settings page
- [ ] Create login page
- [ ] Create registration page
- [ ] Add SEO metadata to all pages
- [ ] Optimize for Core Web Vitals

## Success Criteria

- [ ] Search returns relevant results <1s
- [ ] Property pages render server-side
- [ ] Booking flow completes in 3 steps
- [ ] Guest can view booking history
- [ ] Mobile experience is smooth
- [ ] Lighthouse score >80 on mobile

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| SEO issues | High | Test with Search Console, structured data |
| Slow search | Medium | Add caching, optimize queries |
| Booking abandonment | High | Simple flow, save progress |

## Security Considerations

- Rate limit search to prevent scraping
- Validate guest inputs
- Secure payment data handling (Phase 07)
- Prevent booking manipulation

## Next Steps

After completion, proceed to [Phase 07: Payment Integration](./phase-07-payment-integration-sepay-vnpay-subscriptions.md)
