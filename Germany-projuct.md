# Customer App — Full Architecture Design

> **Project:** On-Demand Home Salon Platform (Germany)
> **App:** Customer Mobile App
> **Stack:** Flutter + Supabase (Auth, Postgres, Realtime, Storage, Edge Functions)
> **Version:** v1.0 MVP Architecture
> **Date:** 2026-09-05
> **Builds on:** Partner App architecture (mirror for data model, inverse for UX flow)

---

## 0. How Customer App Fits in the System

```
┌──────────────────┐         ┌──────────────────┐
│  CUSTOMER APP    │◄───────►│   SUPABASE       │
│  (Flutter)       │  REST   │   - Postgres     │
│                  │  + RT   │   - Realtime     │
│  9 screens       │  + Auth │   - Auth         │
│  Browse+Book+Pay │  + RLS  │   - Storage      │
└────────┬─────────┘         │   - Edge Funcs   │
         │                   └────────┬─────────┘
         │                            │
         │ WebSocket (via Realtime)   │
         ▼                            ▼
┌──────────────────┐         ┌──────────────────┐
│  PARTNER APP     │◄───────►│  DISPATCH        │
│  (Flutter)       │         │  Edge Function   │
│                  │         │  (find partner)  │
└──────────────────┘         └──────────────────┘
                                      │
                                      ▼
                             ┌──────────────────┐
                             │  STRIPE          │
                             │  (Payment)       │
                             └──────────────────┘
```

**Key Insight:** Customer App is mostly **read-heavy + state viewer**, while Partner App is **transaction-heavy + action taker**. So Supabase Realtime shines here — the customer mostly *watches* the partner state change in real time.

---

## 1. Tech Stack (Flutter + Supabase)

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  
  # State management
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  
  # Supabase
  supabase_flutter: ^2.5.0        # Official client
  
  # Routing
  go_router: ^14.0.0              # Declarative routing
  
  # Maps & Location
  google_maps_flutter: ^2.6.0
  geolocator: ^11.0.0
  geocoding: ^3.0.0
  
  # Payments
  flutter_stripe: ^11.0.0         # Apple Pay, Google Pay, Cards
  
  # Local cache
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  shared_preferences: ^2.2.0
  
  # UI helpers
  cached_network_image: ^3.3.0
  flutter_svg: ^2.0.0
  shimmer: ^3.0.0
  intl: ^0.19.0                   # Date/currency formatting
  uuid: ^4.3.0
  
  # Push + Realtime
  firebase_core: ^2.27.0
  firebase_messaging: ^14.7.0
  flutter_local_notifications: ^17.0.0
  
  # Forms & validation
  flutter_form_builder: ^9.0.0
  form_builder_validators: ^10.0.0
  
  # Misc
  url_launcher: ^6.2.0
  share_plus: ^9.0.0
  package_info_plus: ^5.0.0

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.4.0
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  supabase: ^1.0.0                # CLI for type generation
```

---

## 2. Customer App State Machine

Unlike the Partner App (which has clear action states), the Customer App is mostly a **view + one-time commit** app. The state machine is simpler:

```
                    ┌──────────────┐
                    │ UNAUTHENTICATED│ ◄──── App start / Logout
                    └──────┬───────┘
                           │ [Login/OTP verified]
                           ▼
                    ┌──────────────┐
        ┌──────────►│  AUTHENTICATED│◄──────────┐
        │           │  (Browsing)   │           │
        │           └──────┬───────┘           │
        │                  │ [Select service]   │
        │                  ▼                   │
        │           ┌──────────────┐           │
        │     ┌────►│  CONFIGURING │           │
        │     │     │ (Service+Slot)│           │
        │     │     └──────┬───────┘           │
        │     │            │ [Pay]             │
        │     │            ▼                   │
        │     │     ┌──────────────┐           │
        │     │     │   PAYMENT    │           │
        │     │     │  PROCESSING  │           │
        │     │     └──────┬───────┘           │
        │     │            │ [Success]         │
        │     │            ▼                   │
        │     │     ┌──────────────┐           │
        │     │     │   BOOKING    │           │
        │     │     │   ACTIVE     │           │
        │     │     │  (tracking)  │           │
        │     │     └──────┬───────┘           │
        │     │            │ [Service done]    │
        │     │            ▼                   │
        │     │     ┌──────────────┐           │
        │     │     │   RATING     │           │
        │     │     │  PENDING     │           │
        │     │     └──────┬───────┘           │
        │     │            │ [Submit]          │
        │     │            ▼                   │
        │     │     ┌──────────────┐           │
        └─────┴─────│  COMPLETED   │           │
                  │  (Receipt)   │           │
                  └──────┬───────┘           │
                         │ [Book again] ─────┘
```

**Key states for the Customer App:**
1. `UNAUTHENTICATED` — login screen
2. `AUTHENTICATED_IDLE` — default, browsing home
3. `CONFIGURING` — selected a service, choosing addons/slot/address
4. `PAYMENT_PROCESSING` — Stripe sheet open
5. `BOOKING_ACTIVE` — paid, watching partner approach
6. `RATING_PENDING` — service done, modal to rate
7. `COMPLETED` — rated, receipt shown
8. `ERROR` — payment failed, PIN invalid, etc. (overlay state)

**State persistence:** Riverpod `StateNotifier` for app-level state. Active booking ID stored in `SharedPreferences` for cold-start recovery (if app killed while tracking).

---

## 3. Screen-by-Screen Architecture

### Screen 1: Splash & Onboarding / Login

**Purpose:** Authenticate user, capture location, set up push notifications.

**UI Components:**
- Logo (top, centered)
- Phone input (with country code — DE default +49)
- "Continue" button (sends OTP)
- OTP input (6 digits, auto-fill from SMS)
- Social login: Google + Apple
- Language toggle (DE/EN)

**Sub-screens (in onboarding flow):**
```
Step 1: Phone + OTP
Step 2: First name + Last name
Step 3: Location permission (precise)
Step 4: Notification permission
Step 5: Address setup (or "skip for now")
```

**Data captured:**
```dart
class CustomerProfile {
  String id;            // UUID
  String phone;
  String? email;
  String firstName;
  String? lastName;
  String? defaultAddressId;
  String languagePreference;  // 'de' | 'en'
  DateTime createdAt;
}
```

**Supabase Implementation:**

```dart
// Phone auth with Supabase
await supabase.auth.signInWithOtp(
  phone: '+49${phoneNumber}',
);

// Verify OTP
final response = await supabase.auth.verifyOTP(
  phone: '+49${phoneNumber}',
  token: otpCode,
  type: OtpType.sms,
);

// After auth, ensure customer profile exists
await supabase.from('customers').upsert({
  'id': supabase.auth.currentUser!.id,
  'phone': supabase.auth.currentUser!.phone,
  'first_name': firstName,
  'language': 'de',
});
```

**Location Permission UX:**
- On first booking (not on signup) — ask for "While using app" location
- This avoids the "creepy" feeling of asking location immediately
- Use `Permission_handler` package

**Edge Cases:**
- Existing customer with old session → silent re-auth
- Phone already registered → log in (not register)
- OTP timeout (60s) → "Resend code" button

---

### Screen 2: Home Screen (Service Selection)

**Purpose:** Main browse/discovery screen. Customer picks service category.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  📍 Berlin, Charlottenburg ▼   👤  │  ← Header
├─────────────────────────────────────────┤
│  🔍  Search services...                │  ← Search bar
├─────────────────────────────────────────┤
│  Categories                            │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐│
│  │ ✂️   │ │ 🧔  │ │ 💇‍♀️ │ │ 👶  ││
│  │ Men  │ │Beard │ │Women │ │Kids ││
│  └──────┘ └──────┘ └──────┘ └──────┘│
├─────────────────────────────────────────┤
│  Popular near you                      │
│  ┌─────────────────────────────────┐  │
│  │ [img] Men's Haircut             │  │
│  │       45 min • €35 • ⭐ 4.8    │  │
│  └─────────────────────────────────┘  │
│  ┌─────────────────────────────────┐  │
│  │ [img] Haircut + Beard Combo     │  │
│  │       60 min • €50 • ⭐ 4.9    │  │
│  └─────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  Why choose us?                        │
│  ✓ Professional cleanup                │
│  ✓ Verified partners                   │
│  ✓ Cashless payment                   │
└─────────────────────────────────────────┘
```

**Data Requirements:**
- `current_city` (from user location)
- `service_categories[]` (static, hardcoded in app)
- `popular_services[]` (from Supabase, filtered by city)
- `user_recent_searches[]` (from local cache)

**Supabase Query:**
```dart
// Fetch services active in user's city
final services = await supabase
  .from('services')
  .select('''
    id, name, slug, description, base_price_cents, 
    duration_min, image_url, category
  ''')
  .eq('is_active', true)
  .order('display_order')
  .execute();
```

**Data Model (services table):**
```sql
CREATE TABLE services (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    slug                VARCHAR(50) UNIQUE NOT NULL,  -- 'mens-haircut'
    category            ENUM('men', 'women', 'kids', 'beard', 'combo'),
    name_de             VARCHAR(100) NOT NULL,
    name_en             VARCHAR(100) NOT NULL,
    description_de      TEXT,
    description_en      TEXT,
    base_price_cents    INT NOT NULL,
    duration_min        INT NOT NULL,
    image_url           TEXT,
    display_order       INT DEFAULT 0,
    is_active           BOOLEAN DEFAULT TRUE,
    created_at          TIMESTAMPTZ DEFAULT NOW()
);

-- Example data
INSERT INTO services (slug, category, name_de, name_en, base_price_cents, duration_min) VALUES
  ('mens-haircut',     'men',   'Herrenhaarschnitt',     'Men''s Haircut',     3500, 45),
  ('beard-trim',       'beard', 'Bartpflege',            'Beard Trim',         2000, 30),
  ('mens-haircut-beard','combo','Herrenschnitt + Bart',  'Men''s Cut + Beard', 5000, 60),
  ('womens-cut-style', 'women', 'Damenschnitt + Styling','Women''s Cut/Style', 7500, 75),
  ('kids-haircut',     'kids',  'Kinderschnitt',         'Kids Haircut',       2500, 30);
```

**Add-ons (separate table, related to services):**
```sql
CREATE TABLE service_addons (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    service_id      UUID REFERENCES services(id),  -- NULL = available for all
    name_de         VARCHAR(100) NOT NULL,
    name_en         VARCHAR(100) NOT NULL,
    price_cents     INT NOT NULL,
    duration_min    INT DEFAULT 0,
    is_active       BOOLEAN DEFAULT TRUE
);

-- Example
INSERT INTO service_addons VALUES
  (gen_random_uuid(), NULL, 'Premium Bartöl',          'Premium Beard Oil',     500, 0, TRUE),
  (gen_random_uuid(), NULL, 'Kopfhautmassage',         'Scalp Massage',         800, 10, TRUE),
  (gen_random_uuid(), NULL, 'Augenbrauen zupfen',      'Eyebrow Threading',     600, 10, TRUE);
```

**UX Details:**
- Search bar → debounced 300ms → fetch from services table
- "Popular near you" → based on user's postal code + recent bookings
- Categories → image grid, tap → list filtered by category
- Service card → tap → goes to Screen 3 (Service Detail)

---

### Screen 3: Service Detail & Add-ons

**Purpose:** Show full service info, allow add-on selection, set quantity.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  ← Back                       Share 🔗│
├─────────────────────────────────────────┤
│  [       Service Hero Image        ]   │
├─────────────────────────────────────────┤
│  Men's Haircut                         │
│  ⭐ 4.8 (124 bookings)                 │
│  ⏱  45 minutes                         │
│  💰 €35.00                             │
├─────────────────────────────────────────┤
│  Description                           │
│  Professional haircut at your home,    │
│  including consultation, wash, cut,    │
│  and styling. We bring all equipment.  │
├─────────────────────────────────────────┤
│  What's included                       │
│  ✓ Professional consultation           │
│  ✓ Disposable cape (hygienic)          │
│  ✓ Floor mat & sheet                  │
│  ✓ Portable vacuum cleanup            │
│  ✓ Waste bag disposal                 │
├─────────────────────────────────────────┤
│  Add-ons (optional)                    │
│  ┌─────────────────────────────────┐  │
│  │ ☐ Premium Beard Oil      +€5.00 │  │
│  │ ☐ Scalp Massage         +€8.00 │  │
│  │ ☐ Eyebrow Threading     +€6.00 │  │
│  └─────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  Total: €35.00                         │
│  [    CONTINUE TO BOOK →  ]           │
└─────────────────────────────────────────┘
```

**Data Flow:**
- Receives `serviceId` as route param
- Loads service + addons in single query (use Supabase embedded resources)
- Local state: `selectedAddons: Set<String>`, `quantity: int`
- "Continue" → navigates to Screen 4 (Schedule)

**Supabase Query (with relationships):**
```dart
final service = await supabase
  .from('services')
  .select('''
    *,
    service_addons(*)
  ''')
  .eq('id', serviceId)
  .single();
```

**Analytics to Track:**
- `service_viewed` (with addons shown)
- `addon_added` (which addon, with which base service)
- `service_to_schedule` (conversion event)

---

### Screen 4: Cart & Schedule Slot

**Purpose:** Choose address, date, and time slot.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  ← Back          Booking Summary        │
├─────────────────────────────────────────┤
│  ✂️  Men's Haircut                    │
│  + Beard Oil (+€5)                     │
│  + Scalp Massage (+€8)                 │
│  Duration: ~65 min                     │
├─────────────────────────────────────────┤
│  📍 Service Location                   │
│  ┌─────────────────────────────────┐  │
│  │ 🏠 Home                         │  │
│  │    Karl-Marx-Allee 92, Berlin   │  │
│  │    3rd floor, Apt 3B           │  │
│  │    [Change address]            │  │
│  └─────────────────────────────────┘  │
│  [📍 Use current location]             │
│  [➕ Add new address]                  │
├─────────────────────────────────────────┤
│  📅 Date                              │
│  ┌──┬──┬──┬──┬──┬──┬──┐              │
│  │Mo│Tu│We│Th│Fr│Sa│Su│              │
│  └──┴──┴──┴──┴──┴──┴──┘              │
│       Selected: Wed, Sep 7             │
├─────────────────────────────────────────┤
│  🕐 Time slot                         │
│  [09:00] [10:00] [11:00] [12:00]      │
│  [13:00] [14:00] [15:00] [16:00] ✓   │
│  [17:00] [18:00] [19:00] [20:00]      │
│  Earliest available: in 35 min        │
├─────────────────────────────────────────┤
│  Service Area Verified ✓              │
│  (within our partner coverage)        │
├─────────────────────────────────────────┤
│  💰 Subtotal:     €48.00              │
│  💳 Service fee:   €2.00              │
│  ─────────────────────                │
│  Total:            €50.00              │
│  [    CONTINUE TO PAYMENT →  ]         │
└─────────────────────────────────────────┘
```

**Data Requirements:**
- `selected_address` (from saved addresses)
- `selected_date` (date only)
- `selected_slot` (time)
- Computed: `earliest_available` (real-time calculation)
- Computed: `service_area_valid` (geo-fence check)

**Addresses Data Model:**
```sql
CREATE TABLE customer_addresses (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id     UUID NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
    label           VARCHAR(50),  -- 'Home', 'Work', 'Other'
    street          VARCHAR(255) NOT NULL,
    additional_info TEXT,         -- 'Apt 3B, Buzzer 204'
    postal_code     VARCHAR(10) NOT NULL,
    city            VARCHAR(100) NOT NULL,
    country         VARCHAR(2) DEFAULT 'DE',
    lat             DOUBLE PRECISION NOT NULL,
    lng             DOUBLE PRECISION NOT NULL,
    is_default      BOOLEAN DEFAULT FALSE,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_addresses_customer ON customer_addresses(customer_id);

-- RLS: customer can only see/modify their own addresses
ALTER TABLE customer_addresses ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can manage own addresses" ON customer_addresses
  FOR ALL USING (customer_id = auth.uid());
```

**Slot Calculation (Edge Function or client-side):**

The hard part: showing only slots that have at least 1 available partner.

```dart
// Client-side approximation
Future<List<TimeSlot>> getAvailableSlots(DateTime date, String addressId) async {
  // 1. Get all slots for the day (8 AM to 9 PM, 1-hour intervals)
  final allSlots = generateHourlySlots(date);
  
  // 2. For each slot, check if any partner is available
  // This is a heavy query — best done via Supabase Edge Function
  final response = await supabase.functions.invoke('get-available-slots', 
    body: { 'date': date.toIso8601String(), 'address_id': addressId }
  );
  
  return (response.data as List).map(TimeSlot.fromJson).toList();
}
```

**Edge Function (Supabase):**
```typescript
// supabase/functions/get-available-slots/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

serve(async (req) => {
  const { date, address_id } = await req.json();
  const supabase = createClient(/* ... */);
  
  // Get address coordinates
  const { data: address } = await supabase
    .from('customer_addresses')
    .select('lat, lng')
    .eq('id', address_id)
    .single();
  
  // Get online partners within 10km
  const { data: partners } = await supabase
    .from('partners')
    .select('id, current_lat, current_lng, working_hours')
    .eq('is_online', true)
    .eq('status', 'active');
  
  // Filter by distance
  const nearbyPartners = partners.filter(p => 
    haversineDistance(p.current_lat, p.current_lng, address.lat, address.lng) < 10
  );
  
  // Generate slots
  const slots = generateSlots(date, nearbyPartners);
  
  return new Response(JSON.stringify(slots), { headers: { 'Content-Type': 'application/json' } });
});
```

**UX Details:**
- "Earliest available" badge — finds the next available slot in next 2 hours
- Selected slot must have at least 1 partner available — otherwise greyed out
- Address map preview (small thumbnail)
- "Use current location" → requests location → reverse geocodes to address

---

### Screen 5: Checkout & Payment

**Purpose:** Collect payment, optional tip, confirm booking.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  ← Back          Checkout              │
├─────────────────────────────────────────┤
│  Booking Details                       │
│  ✂️  Men's Haircut                    │
│  📅  Wed, Sep 7 at 16:00              │
│  📍  Karl-Marx-Allee 92, Berlin       │
├─────────────────────────────────────────┤
│  Price Breakdown                       │
│  Service:              €35.00          │
│  Beard Oil:             €5.00          │
│  Scalp Massage:         €8.00          │
│  Platform fee:          €2.00          │
│  ─────────────────────                 │
│  Subtotal:             €50.00          │
├─────────────────────────────────────────┤
│  💚 Tip your partner (optional)       │
│  [10%]  [15%]  [20%]  [Custom]        │
│  Tip: €5.00                            │
├─────────────────────────────────────────┤
│  Payment Method                        │
│  ┌─────────────────────────────────┐  │
│  │ 💳 Visa **** 4242   [Default]  │  │
│  │    Expires 12/26                │  │
│  └─────────────────────────────────┘  │
│  [Change payment method]               │
│  + Add new card                        │
│  [Apple Pay]  [Google Pay]             │
├─────────────────────────────────────────┤
│  Total charged: €55.00                 │
│                                         │
│  ☐ I agree to Terms of Service        │
│  ☐ I agree to Cancellation Policy     │
│                                         │
│  [   PAY €55.00 & BOOK   ]            │
└─────────────────────────────────────────┘
```

**Data Flow:**
1. User confirms
2. Client validates all checkboxes ticked
3. Create `pending_payment` record in DB
4. Open Stripe Payment Sheet
5. On success → confirm booking → trigger dispatch
6. On failure → show error → allow retry

**Stripe Integration (Flutter):**

```dart
// 1. Create payment intent via Supabase Edge Function
final paymentIntent = await supabase.functions.invoke(
  'create-payment-intent',
  body: {
    'amount_cents': totalAmount,  // 5500
    'currency': 'eur',
    'customer_id': customerId,
    'booking_data': { /* service, slot, address */ }
  }
);

// 2. Initialize Stripe payment sheet
await Stripe.instance.initPaymentSheet(
  paymentSheetParameters: SetupPaymentSheetParameters(
    paymentIntentClientSecret: paymentIntent.data['client_secret'],
    customerEphemeralKey: paymentIntent.data['ephemeral_key'],
    customerId: paymentIntent.data['customer_id'],
    merchantDisplayName: 'SalonAtHome',
  ),
);

// 3. Present payment sheet
await Stripe.instance.presentPaymentSheet();

// 4. On success → create booking
final booking = await supabase.from('bookings').insert({...}).select().single();
```

**Payment Method Storage:**
- Use Stripe Customer object (not local card storage)
- Save `stripe_customer_id` and `default_payment_method_id` in `customers` table
- User can manage cards via Stripe-hosted page or in-app via `payment_sheet`

**Tip Logic:**
- Tip is added to total but **goes 100% to partner** (no platform cut on tips)
- Stored separately in `bookings.tip_cents`
- Visible to partner in their wallet breakdown

**Idempotency:**
- Use `idempotency_key` (UUID generated client-side) for payment intent
- If user taps "Pay" twice → only one charge
- Bookings table has UNIQUE constraint on `idempotency_key`

**Tax & Invoice:**
- 19% VAT included in prices (already factored in platform fee)
- After payment, generate PDF invoice (stored in Supabase Storage)
- Send to customer email + available in Profile → History

---

### Screen 6: Live Booking Status / Tracking

**Purpose:** Real-time view of partner approach + service progress.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  ← Back        Booking #SAL-2847       │
├─────────────────────────────────────────┤
│  Status: Partner En Route 🚗           │
│  ETA: 12 min                           │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐  │
│  │                                  │  │
│  │    [    Map with route       ]   │  │
│  │    Partner: 🚗                   │  │
│  │    You: 📍                       │  │
│  │                                  │  │
│  └─────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  Your Partner                          │
│  ┌──────┐  Raj Kumar                   │
│  │ img  │  ⭐ 4.8 (124 jobs)           │
│  └──────┘  3 years experience          │
│  [📞 Call]  [💬 Chat]                  │
├─────────────────────────────────────────┤
│  Timeline                              │
│  ✅ Payment confirmed      15:32      │
│  ✅ Partner assigned        15:33      │
│  ✅ Partner en route        15:42      │
│  ⏳ Arriving soon           ~16:00     │
│  ⏳ Service starts                      │
│  ⏳ Service completes                   │
├─────────────────────────────────────────┤
│  Booking Details                       │
│  ✂️  Men's Haircut + Beard            │
│  📅  Wed, Sep 7 at 16:00              │
│  📍  Karl-Marx-Allee 92, 3rd floor    │
│  💰  Paid: €55.00                      │
│                                         │
│  [   CANCEL BOOKING   ]               │  ← only if before partner arrives
└─────────────────────────────────────────┘
```

**Real-time Updates (Supabase Realtime):**

```dart
// Subscribe to booking changes
final channel = supabase
  .channel('booking-${bookingId}')
  .on('postgres_changes', 
    event: PostgresChangeEvent.update,
    schema: 'public',
    table: 'bookings',
    filter: 'id=eq.${bookingId}',
    callback: (payload) {
      final booking = Booking.fromJson(payload.newRecord);
      ref.read(bookingProvider.notifier).update(booking);
    }
  )
  .on('postgres_changes',
    event: PostgresChangeEvent.update,
    schema: 'public',
    table: 'partners',
    filter: 'id=eq.${partnerId}',
    callback: (payload) {
      // Partner location update → move marker on map
      final partner = Partner.fromJson(payload.newRecord);
      ref.read(partnerLocationProvider.notifier).update(partner);
    }
  )
  .subscribe();
```

**Map Implementation:**
- `google_maps_flutter` with custom marker for partner (animated car icon)
- Route polyline from partner's current location to customer's address
- ETA calculation: done in partner app, sent via DB update

**Timeline Updates (triggered by booking state changes):**
- `pending` → Payment confirmed
- `assigned` → Partner assigned
- `en_route` → Partner started travel
- `at_location` → Partner arrived
- `in_service` → Service started (PIN verified)
- `completed` → Service done
- `cancelled` → Cancelled
- `disputed` → Issue raised

**Cancellation UX:**
- Free cancellation until partner accepts (full refund)
- 50% fee if cancelled after partner en route
- No refund if cancelled after partner arrives (unless emergency)

**Push Notifications (for major events):**
- "Payment successful"
- "Partner Raj has been assigned"
- "Raj is on the way — ETA 12 min"
- "Raj has arrived"
- "Service in progress"
- "Service complete — please rate"

---

### Screen 7: In-Service PIN Verification

**Purpose:** Customer gets a 4-digit PIN to share with partner. This is the customer's view of the verification.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  ← Back        Service in Progress      │
├─────────────────────────────────────────┤
│  Status: Partner has arrived ✓         │
├─────────────────────────────────────────┤
│  Your start PIN:                       │
│                                         │
│      ┌───┐ ┌───┐ ┌───┐ ┌───┐          │
│      │ 4 │ │ 8 │ │ 2 │ │ 0 │          │
│      └───┘ └───┘ └───┘ └───┘          │
│                                         │
│  Share this PIN with your partner to   │
│  confirm they've arrived and start     │
│  the service.                          │
├─────────────────────────────────────────┤
│  ⏱  PIN expires in 4:32                │
│                                         │
│  Don't recognize this PIN?             │
│  This PIN is unique to this booking.   │
├─────────────────────────────────────────┤
│  Safety tips                           │
│  ✓ Confirm partner's name before       │
│    sharing PIN                         │
│  ✓ Service starts only after PIN       │
│    is entered in partner's app         │
│  ✓ You'll get a receipt after service  │
└─────────────────────────────────────────┘
```

**Key Design Decision:** Customer sees PIN but **does not enter it themselves**. The partner enters it in their app after the customer shares it verbally/visually.

**PIN Generation:**
- Generated at booking confirmation (after payment)
- 4 random digits, never reused
- Stored in `bookings.start_pin` (hashed with bcrypt for security)
- Expires 5 minutes after partner marks "arrived"
- New PIN generated if expired (regeneration limit: 3 per booking)

**Why this design:**
- Customer doesn't need to type on partner's phone (hygiene + privacy)
- Verifies partner is physically at location (not a no-show)
- Customer feels in control — "I have the PIN, partner can't start without it"

**Realtime Transition:**
- When partner enters PIN correctly → booking state → `in_service`
- Customer screen auto-transitions to "Service in progress" view

---

### Screen 8: Review, Rating & Tip

**Purpose:** After service complete, rate the partner and adjust tip.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│        Service Complete! 🎉             │
├─────────────────────────────────────────┤
│  How was your experience?              │
│                                         │
│      ⭐ ⭐ ⭐ ⭐ ⭐                     │  ← Interactive stars
│                                         │
│  Rate specific aspects:                │
│  Punctuality:     ⭐⭐⭐⭐⭐           │
│  Cleanliness:     ⭐⭐⭐⭐⭐           │
│  Quality:         ⭐⭐⭐⭐⭐           │
│  Communication:   ⭐⭐⭐⭐⭐           │
├─────────────────────────────────────────┤
│  Add a photo (optional)                │
│  [📷 Before/After]                     │
├─────────────────────────────────────────┤
│  Comments (optional)                   │
│  ┌─────────────────────────────────┐  │
│  │ Great service! Very clean...    │  │
│  └─────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  Adjust tip                            │
│  Current tip: €5.00                    │
│  [€0] [€3] [€5] [€10] [€20] [Custom]  │
├─────────────────────────────────────────┤
│  [    SUBMIT REVIEW    ]              │
│  Skip for now                          │
└─────────────────────────────────────────┘
```

**Trigger:**
- Shown when booking state changes to `completed`
- Modal opens automatically (with option to "Skip for now" and rate later)
- Push notification: "How was your service? Rate Raj ⭐"

**Data Model:**
```sql
CREATE TABLE ratings (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    booking_id      UUID NOT NULL REFERENCES bookings(id) UNIQUE,
    customer_id     UUID NOT NULL REFERENCES customers(id),
    partner_id      UUID NOT NULL REFERENCES partners(id),
    stars_overall   INT CHECK (stars_overall BETWEEN 1 AND 5),
    stars_punctuality INT CHECK (stars_punctuality BETWEEN 1 AND 5),
    stars_cleanliness INT CHECK (stars_cleanliness BETWEEN 1 AND 5),
    stars_quality   INT CHECK (stars_quality BETWEEN 1 AND 5),
    stars_communication INT CHECK (stars_communication BETWEEN 1 AND 5),
    comment         TEXT,
    photo_urls      TEXT[],
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE ratings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can rate own bookings" ON ratings
  FOR INSERT WITH CHECK (customer_id = auth.uid());
```

**Tip Adjustment:**
- Show original tip
- Allow increasing or removing tip (not decreasing below original)
- Tip change updates `bookings.tip_cents` (if not yet paid out to partner)
- Triggers re-credit to partner wallet

**Triggering Partner Rating Update:**
```sql
-- Trigger function (Supabase)
CREATE OR REPLACE FUNCTION update_partner_rating()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE partners SET
    rating_avg = (
      SELECT AVG(stars_overall) FROM ratings WHERE partner_id = NEW.partner_id
    ),
    rating_count = (
      SELECT COUNT(*) FROM ratings WHERE partner_id = NEW.partner_id
    )
  WHERE id = NEW.partner_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER on_rating_created
  AFTER INSERT ON ratings
  FOR EACH ROW
  EXECUTE FUNCTION update_partner_rating();
```

**Low Rating Follow-up:**
- 1-2 stars → auto-trigger support ticket creation
- 1 star → priority review by admin
- Customer may be asked to provide photo evidence

---

### Screen 9: Profile & Booking History

**Purpose:** Manage account, addresses, payment methods, view history.

**UI Layout:**
```
┌─────────────────────────────────────────┐
│  👤 Priya Sharma                       │
│     +49 176 •• •• 5678   [Edit]       │
├─────────────────────────────────────────┤
│  [📋 Bookings] [📍 Addresses] [💳 Pay] │
├─────────────────────────────────────────┤
│  Upcoming                              │
│  ┌─────────────────────────────────┐  │
│  │ Wed, Sep 7 • 16:00              │  │
│  │ Men's Haircut + Beard           │  │
│  │ Raj Kumar  •  Status: Active    │  │
│  │ [View] [Cancel]                 │  │
│  └─────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  Past                                  │
│  ┌─────────────────────────────────┐  │
│  │ Mon, Aug 29 • 11:00             │  │
│  │ Women's Cut & Style             │  │
│  │ Anil M.  •  €78.00              │  │
│  │ ⭐ 5  •  [Re-book] [Receipt]   │  │
│  └─────────────────────────────────┘  │
│  [Show more]                           │
├─────────────────────────────────────────┤
│  Settings                              │
│  • Language (DE/EN)                    │
│  • Notifications                       │
│  • Privacy & Data (GDPR export)        │
│  • Terms of Service                    │
│  • Help & Support                      │
│  • Logout                              │
└─────────────────────────────────────────┘
```

**Tabs:**

**Tab 1: Bookings**
- Upcoming (next 30 days)
- Past (paginated, oldest first)
- Filter: All / Completed / Cancelled
- Each booking → detail view with:
  - Full receipt (PDF download)
  - Partner info
  - Re-book button (1-tap repeat)
  - Support button (if dispute)

**Tab 2: Addresses**
- List of saved addresses
- Add new (with map picker)
- Edit / Delete / Set as default
- Default address highlighted

**Tab 3: Payment**
- Saved cards (via Stripe)
- Default card
- Add new card
- Transaction history
- Download tax invoice (Jahresbescheinigung for Steuererklärung)

**Settings:**
- Language: DE / EN
- Push notifications: master toggle + per-type
- Email notifications
- GDPR: Export my data / Delete account
- App version, build number
- Logout (with confirmation)

---

## 4. Supabase Database Schema (Customer-Facing)

```sql
-- Customers
CREATE TABLE customers (
    id                      UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    phone                   VARCHAR(20) UNIQUE NOT NULL,
    email                   VARCHAR(255),
    first_name              VARCHAR(100) NOT NULL,
    last_name               VARCHAR(100),
    language                VARCHAR(5) DEFAULT 'de',
    stripe_customer_id      TEXT UNIQUE,
    default_payment_method_id TEXT,
    fcm_token               TEXT,
    status                  ENUM('active', 'suspended', 'deleted') DEFAULT 'active',
    total_bookings          INT DEFAULT 0,
    total_spent_cents       BIGINT DEFAULT 0,
    created_at              TIMESTAMPTZ DEFAULT NOW(),
    last_active_at          TIMESTAMPTZ
);

ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can view own profile" ON customers
  FOR SELECT USING (id = auth.uid());
CREATE POLICY "Users can update own profile" ON customers
  FOR UPDATE USING (id = auth.uid());

-- Customer Addresses (already shown in Screen 4)

-- Services (already shown in Screen 2)
-- Service Addons (already shown in Screen 2)

-- Bookings (THE CENTRAL TABLE)
CREATE TABLE bookings (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    short_code          VARCHAR(20) UNIQUE NOT NULL,  -- 'SAL-2847'
    customer_id         UUID NOT NULL REFERENCES customers(id),
    partner_id          UUID REFERENCES partners(id),
    address_id          UUID NOT NULL REFERENCES customer_addresses(id),
    
    -- Service details
    service_id          UUID NOT NULL REFERENCES services(id),
    addons              JSONB DEFAULT '[]',  -- [{addon_id, price_cents}]
    service_total_cents INT NOT NULL,  -- sum of base + addons
    duration_estimate_min INT NOT NULL,
    
    -- Pricing
    service_fee_cents   INT NOT NULL DEFAULT 200,
    tip_cents           INT DEFAULT 0,
    total_charged_cents INT NOT NULL,
    
    -- Schedule
    scheduled_at        TIMESTAMPTZ NOT NULL,
    
    -- Status
    state               ENUM('pending_payment', 'paid', 'broadcasting', 
                              'assigned', 'en_route', 'at_location',
                              'in_service', 'completed', 'cancelled',
                              'disputed', 'refunded') DEFAULT 'pending_payment',
    
    -- Payment
    stripe_payment_intent_id TEXT,
    payment_status      ENUM('pending', 'succeeded', 'failed', 'refunded'),
    
    -- PIN
    start_pin           TEXT,  -- bcrypt hashed
    pin_generated_at    TIMESTAMPTZ,
    pin_attempts        INT DEFAULT 0,
    
    -- Timestamps
    paid_at             TIMESTAMPTZ,
    assigned_at         TIMESTAMPTZ,
    en_route_at         TIMESTAMPTZ,
    arrived_at          TIMESTAMPTZ,
    started_at          TIMESTAMPTZ,
    completed_at        TIMESTAMPTZ,
    cancelled_at        TIMESTAMPTZ,
    
    -- Cancellation
    cancelled_by        ENUM('customer', 'partner', 'admin', 'system'),
    cancellation_reason TEXT,
    refund_cents        INT DEFAULT 0,
    
    -- Idempotency
    idempotency_key     UUID UNIQUE,
    
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    updated_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_bookings_customer ON bookings(customer_id, created_at DESC);
CREATE INDEX idx_bookings_partner_state ON bookings(partner_id, state);
CREATE INDEX idx_bookings_state_scheduled ON bookings(state, scheduled_at);
CREATE INDEX idx_bookings_short_code ON bookings(short_code);

ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can view own bookings" ON bookings
  FOR SELECT USING (customer_id = auth.uid());
CREATE POLICY "Users can create own bookings" ON bookings
  FOR INSERT WITH CHECK (customer_id = auth.uid());

-- Booking State Transitions (audit trail)
CREATE TABLE booking_transitions (
    id              BIGSERIAL PRIMARY KEY,
    booking_id      UUID NOT NULL REFERENCES bookings(id),
    from_state      VARCHAR(30),
    to_state        VARCHAR(30) NOT NULL,
    actor_type      ENUM('customer', 'partner', 'admin', 'system'),
    actor_id        UUID,
    metadata        JSONB,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_transitions_booking ON booking_transitions(booking_id, created_at);

-- Ratings (already shown in Screen 8)

-- Support Tickets
CREATE TABLE support_tickets (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id     UUID NOT NULL REFERENCES customers(id),
    booking_id      UUID REFERENCES bookings(id),
    subject         VARCHAR(255) NOT NULL,
    body            TEXT NOT NULL,
    status          ENUM('open', 'in_progress', 'resolved', 'closed') DEFAULT 'open',
    priority        ENUM('low', 'normal', 'high', 'urgent') DEFAULT 'normal',
    assigned_admin  UUID,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE support_tickets ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can manage own tickets" ON support_tickets
  FOR ALL USING (customer_id = auth.uid());

-- Push Tokens
CREATE TABLE push_tokens (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL,
    user_type       ENUM('customer', 'partner'),
    fcm_token       TEXT NOT NULL,
    device_type     VARCHAR(20),  -- 'ios' | 'android'
    app_version     VARCHAR(20),
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    last_used_at    TIMESTAMPTZ,
    UNIQUE(user_id, fcm_token)
);
```

---

## 5. Supabase Realtime Subscriptions

The Customer App's "Live Tracking" screen is the killer feature. Here's how Realtime works:

**Channels to Subscribe:**

```dart
class RealtimeService {
  RealtimeChannel subscribeToBooking(String bookingId, String partnerId) {
    final channel = supabase.channel('booking-$bookingId');
    
    // 1. Booking state changes
    channel.onPostgresChanges(
      event: PostgresChangeEvent.update,
      schema: 'public',
      table: 'bookings',
      filter: 'id=eq.$bookingId',
      callback: (payload) {
        // Update UI with new state
      },
    );
    
    // 2. Partner location updates (every 15s when en_route)
    channel.onPostgresChanges(
      event: PostgresChangeEvent.update,
      schema: 'public',
      table: 'partners',
      filter: 'id=eq.$partnerId',
      callback: (payload) {
        // Update map marker position
      },
    );
    
    // 3. New chat messages
    channel.onPostgresChanges(
      event: PostgresChangeEvent.insert,
      schema: 'public',
      table: 'chat_messages',
      filter: 'booking_id=eq.$bookingId',
      callback: (payload) {
        // Add message to chat thread
      },
    );
    
    return channel.subscribe();
  }
}
```

**Performance Tip:** Don't subscribe to full partner table. Add a `last_location_update` column and filter by `last_location_update > NOW() - INTERVAL '5 minutes'` to avoid stale data.

```sql
-- Add to partners table
ALTER TABLE partners ADD COLUMN last_location_update TIMESTAMPTZ;

-- Index for realtime
CREATE INDEX idx_partners_realtime ON partners(id) 
  WHERE is_online = true AND last_location_update > NOW() - INTERVAL '5 minutes';
```

---

## 6. Key User Flows

### Flow 1: First-Time Booking (Happy Path)

```
1. App opens → Onboarding (phone + OTP)
2. Grant location permission
3. Home screen → Tap "Men's Haircut"
4. Service detail → Add Beard Oil → Tap Continue
5. Schedule screen → Select today + 16:00 → Tap Continue
6. Checkout → Select card → Tap Pay
7. Stripe sheet opens → User pays
8. Booking created in DB (state: 'paid')
9. Edge function: 'dispatch-job' triggered
10. Dispatch finds partner → state: 'assigned' → push: "Raj assigned"
11. Partner accepts → state: 'en_route' → push: "Raj is on the way"
12. Partner arrives → state: 'at_location' → customer sees PIN
13. Partner enters PIN → state: 'in_service' → push: "Service started"
14. Partner completes → state: 'completed' → rating modal opens
15. Customer rates → rating saved → push: "Thanks for rating!"
16. Booking moves to "Past" tab
17. 7 days later → tip + payout released to partner
```

### Flow 2: Cancellation (Before Partner Assigned)

```
1. Customer on tracking screen → Tap "Cancel Booking"
2. Confirmation modal: "Cancel booking? You'll get a full refund."
3. Customer confirms
4. Booking state → 'cancelled' (cancelled_by: 'customer')
5. Stripe refund triggered (full amount)
6. Push: "Booking cancelled, €50 refunded"
7. Screen returns to Home
```

### Flow 3: Cancellation (After Partner En Route)

```
1. Customer on tracking screen → Tap "Cancel Booking"
2. Modal: "Your partner is on the way. 50% fee may apply."
3. Customer confirms
4. Backend checks: if state IN ('en_route', 'at_location'):
   - cancellation_fee = total * 0.5
   - refund_cents = total - fee
5. Stripe partial refund
6. Push: "Cancelled. €25 refunded, €25 partner fee"
7. Partner notified + gets compensation
```

### Flow 4: Partner No-Show

```
1. Partner assigned → 15 min no-show timer starts
2. Customer sees: "Partner is taking longer than expected. [Call] [Cancel]"
3. If 15 min passes without 'en_route' update:
   - Auto-cancel booking
   - Full refund
   - Partner flagged for review
4. Auto-trigger re-dispatch to next available partner
5. Customer notified: "Looking for another partner..."
6. If found within 10 min → new partner assigned
7. If not found → full refund + apology coupon
```

### Flow 5: Dispute (Customer Unhappy)

```
1. After service → Customer gives 1-2 stars
2. Modal: "We're sorry. Can you tell us what went wrong?"
3. Customer selects issue type:
   - Late arrival
   - Rude behavior
   - Poor quality
   - Didn't clean up
   - Safety concern
4. Optional: upload photos
5. Support ticket auto-created (priority: high)
6. Admin reviews within 4 hours
7. Resolution:
   - Partial refund (50-100%)
   - Discount coupon for next booking
   - Partner warning/penalty
```

---

## 7. Flutter Project Structure

```
lib/
├── main.dart
├── app.dart                          # Root app widget
├── bootstrap.dart                    # Supabase init, FCM setup
│
├── core/
│   ├── config/
│   │   ├── supabase_config.dart
│   │   └── stripe_config.dart
│   ├── constants/
│   │   ├── app_constants.dart
│   │   └── route_names.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   ├── app_colors.dart
│   │   └── app_text_styles.dart
│   ├── utils/
│   │   ├── currency_formatter.dart
│   │   ├── date_formatter.dart
│   │   └── validators.dart
│   ├── errors/
│   │   ├── exceptions.dart
│   │   └── error_handler.dart
│   └── extensions/
│       ├── string_extensions.dart
│       └── datetime_extensions.dart
│
├── data/
│   ├── models/
│   │   ├── customer.dart
│   │   ├── service.dart
│   │   ├── service_addon.dart
│   │   ├── address.dart
│   │   ├── booking.dart
│   │   ├── booking_state.dart
│   │   ├── partner.dart
│   │   ├── rating.dart
│   │   ├── support_ticket.dart
│   │   └── payment_method.dart
│   ├── repositories/
│   │   ├── auth_repository.dart
│   │   ├── customer_repository.dart
│   │   ├── service_repository.dart
│   │   ├── address_repository.dart
│   │   ├── booking_repository.dart
│   │   ├── payment_repository.dart
│   │   ├── rating_repository.dart
│   │   └── support_repository.dart
│   ├── services/
│   │   ├── supabase_service.dart       # Singleton client wrapper
│   │   ├── stripe_service.dart
│   │   ├── fcm_service.dart
│   │   ├── location_service.dart
│   │   ├── realtime_service.dart       # Channel management
│   │   ├── storage_service.dart        # Hive + SharedPrefs
│   │   └── analytics_service.dart
│   └── remote/
│       ├── auth_remote.dart
│       ├── booking_remote.dart
│       ├── service_remote.dart
│       └── edge_functions.dart         # Call Supabase Functions
│
├── features/
│   ├── auth/
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   │   ├── splash_screen.dart
│   │   │   │   ├── phone_login_screen.dart
│   │   │   │   ├── otp_verify_screen.dart
│   │   │   │   └── onboarding_screen.dart
│   │   │   └── widgets/
│   │   │       └── phone_input.dart
│   │   └── state/
│   │       └── auth_provider.dart
│   │
│   ├── home/
│   │   ├── presentation/
│   │   │   ├── screens/home_screen.dart
│   │   │   └── widgets/
│   │   │       ├── service_category_card.dart
│   │   │       ├── popular_service_card.dart
│   │   │       └── search_bar.dart
│   │   └── state/
│   │
│   ├── service/
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   │   ├── service_detail_screen.dart
│   │   │   │   └── schedule_screen.dart
│   │   │   └── widgets/
│   │   │       ├── addon_tile.dart
│   │   │       ├── time_slot_picker.dart
│   │   │       └── date_strip.dart
│   │   └── state/
│   │
│   ├── checkout/
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   │   ├── checkout_screen.dart
│   │   │   │   ├── payment_success_screen.dart
│   │   │   │   └── payment_failed_screen.dart
│   │   │   └── widgets/
│   │   │       ├── price_breakdown.dart
│   │   │       ├── tip_selector.dart
│   │   │       └── payment_method_card.dart
│   │   └── state/
│   │
│   ├── tracking/
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   │   ├── tracking_screen.dart
│   │   │   │   └── pin_display_screen.dart
│   │   │   └── widgets/
│   │   │       ├── booking_map.dart
│   │   │       ├── booking_timeline.dart
│   │   │       ├── partner_info_card.dart
│   │   │       └── live_eta.dart
│   │   └── state/
│   │       ├── booking_provider.dart
│   │       └── partner_location_provider.dart
│   │
│   ├── rating/
│   │   ├── presentation/
│   │   │   ├── screens/rating_screen.dart
│   │   │   └── widgets/
│   │   │       ├── star_rating.dart
│   │   │       └── aspect_rating.dart
│   │   └── state/
│   │
│   ├── profile/
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   │   ├── profile_screen.dart
│   │   │   │   ├── booking_history_screen.dart
│   │   │   │   ├── addresses_screen.dart
│   │   │   │   ├── payment_methods_screen.dart
│   │   │   │   ├── booking_detail_screen.dart
│   │   │   │   └── settings_screen.dart
│   │   │   └── widgets/
│   │   │       ├── booking_tile.dart
│   │   │       ├── address_tile.dart
│   │   │       └── payment_tile.dart
│   │   └── state/
│   │
│   ├── support/
│   │   ├── presentation/
│   │   │   ├── screens/
│   │   │   │   ├── support_screen.dart
│   │   │   │   └── ticket_detail_screen.dart
│   │   │   └── widgets/
│   │   └── state/
│   │
│   └── chat/
│       ├── presentation/
│       │   ├── screens/chat_screen.dart
│       │   └── widgets/
│       │       ├── message_bubble.dart
│       │       └── message_input.dart
│       └── state/
│
├── shared/
│   ├── widgets/
│   │   ├── primary_button.dart
│   │   ├── loading_indicator.dart
│   │   ├── error_view.dart
│   │   ├── empty_view.dart
│   │   └── app_scaffold.dart
│   └── dialogs/
│       ├── cancel_booking_dialog.dart
│       ├── logout_dialog.dart
│       └── error_dialog.dart
│
└── routes/
    ├── app_router.dart
    └── route_paths.dart
```

---

## 8. Riverpod Providers (State Management)

```dart
// ============== Auth ==============
final supabaseClientProvider = Provider<SupabaseClient>((ref) => Supabase.instance.client);

final authStateProvider = StreamProvider<AuthState>((ref) {
  return ref.watch(supabaseClientProvider).auth.onAuthStateChange;
});

final currentUserProvider = Provider<User?>((ref) {
  return ref.watch(authStateProvider).value?.session?.user;
});

final customerProfileProvider = FutureProvider<Customer?>((ref) async {
  final user = ref.watch(currentUserProvider);
  if (user == null) return null;
  
  final response = await ref.watch(supabaseClientProvider)
    .from('customers')
    .select()
    .eq('id', user.id)
    .single();
  return Customer.fromJson(response);
});

// ============== Services ==============
final servicesProvider = FutureProvider<List<Service>>((ref) async {
  final response = await ref.watch(supabaseClientProvider)
    .from('services')
    .select('*, service_addons(*)')
    .eq('is_active', true)
    .order('display_order');
  return (response as List).map(Service.fromJson).toList();
});

final serviceByIdProvider = FutureProvider.family<Service, String>((ref, id) async {
  final services = await ref.watch(servicesProvider.future);
  return services.firstWhere((s) => s.id == id);
});

// ============== Booking Flow ==============
class BookingDraft {
  final Service service;
  final List<ServiceAddon> addons;
  final CustomerAddress address;
  final DateTime scheduledAt;
  final int tipCents;
  
  int get subtotalCents => service.basePriceCents + 
    addons.fold(0, (sum, a) => sum + a.priceCents);
  int get totalCents => subtotalCents + 200 + tipCents; // +200 service fee
}

final bookingDraftProvider = StateProvider<BookingDraft?>((ref) => null);

final activeBookingProvider = StreamProvider<Booking?>((ref) {
  final user = ref.watch(currentUserProvider);
  if (user == null) return Stream.value(null);
  
  // Stream from bookings table
  return ref.watch(supabaseClientProvider)
    .from('bookings')
    .stream(primaryKey: ['id'])
    .eq('customer_id', user.id)
    .inFilter('state', ['paid', 'broadcasting', 'assigned', 'en_route', 
                         'at_location', 'in_service'])
    .order('created_at', ascending: false)
    .limit(1)
    .map((rows) => rows.isEmpty ? null : Booking.fromJson(rows.first));
});

// ============== Realtime ==============
final bookingChannelProvider = StreamProvider.family<RealtimeChannel, String>(
  (ref, bookingId) {
    final channel = ref.watch(supabaseClientProvider).channel('booking-$bookingId');
    
    channel.onPostgresChanges(
      event: PostgresChangeEvent.update,
      schema: 'public',
      table: 'bookings',
      filter: 'id=eq.$bookingId',
      callback: (payload) {
        // Trigger refresh of activeBookingProvider
        ref.invalidate(activeBookingProvider);
      },
    );
    
    return Stream.value(channel.subscribe());
  },
);

// ============== Addresses ==============
final addressesProvider = FutureProvider<List<CustomerAddress>>((ref) async {
  final user = ref.watch(currentUserProvider);
  if (user == null) return [];
  
  final response = await ref.watch(supabaseClientProvider)
    .from('customer_addresses')
    .select()
    .eq('customer_id', user.id)
    .order('is_default', ascending: false);
  return (response as List).map(CustomerAddress.fromJson).toList();
});
```

---

## 9. Edge Functions (Supabase)

### `create-payment-intent`

```typescript
// supabase/functions/create-payment-intent/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";
import Stripe from "https://esm.sh/stripe@13.0.0?target=deno";

const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY')!, {
  apiVersion: '2023-08-16',
  httpClient: Stripe.createFetchHttpClient(),
});

const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);

serve(async (req) => {
  const { amount_cents, customer_id, booking_data, idempotency_key } = await req.json();
  
  // 1. Get or create Stripe customer
  const { data: customer } = await supabase
    .from('customers')
    .select('stripe_customer_id')
    .eq('id', customer_id)
    .single();
  
  let stripeCustomerId = customer?.stripe_customer_id;
  if (!stripeCustomerId) {
    const stripeCustomer = await stripe.customers.create({
      metadata: { supabase_user_id: customer_id }
    });
    stripeCustomerId = stripeCustomer.id;
    
    await supabase
      .from('customers')
      .update({ stripe_customer_id: stripeCustomerId })
      .eq('id', customer_id);
  }
  
  // 2. Create ephemeral key (for payment sheet)
  const ephemeralKey = await stripe.ephemeralKeys.create(
    { customer: stripeCustomerId },
    { apiVersion: '2023-08-16' }
  );
  
  // 3. Create payment intent
  const paymentIntent = await stripe.paymentIntents.create({
    amount: amount_cents,
    currency: 'eur',
    customer: stripeCustomerId,
    automatic_payment_methods: { enabled: true },
    metadata: {
      supabase_user_id: customer_id,
      idempotency_key,
      ...booking_data
    }
  }, { idempotencyKey: idempotency_key });
  
  return new Response(JSON.stringify({
    payment_intent: paymentIntent.client_secret,
    ephemeral_key: ephemeralKey.secret,
    customer: stripeCustomerId
  }), { 
    headers: { 'Content-Type': 'application/json' }
  });
});
```

### `dispatch-job`

```typescript
// supabase/functions/dispatch-job/index.ts
// Called after payment succeeds
// Finds nearest online partner, broadcasts job, sets timer

import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

const supabase = createClient(/* ... */);

serve(async (req) => {
  const { booking_id } = await req.json();
  
  // 1. Get booking details
  const { data: booking } = await supabase
    .from('bookings')
    .select(`
      *,
      customer_addresses(lat, lng),
      services(*)
    `)
    .eq('id', booking_id)
    .single();
  
  // 2. Find online partners within service area
  const { data: partners } = await supabase
    .from('partners')
    .select(`
      id, current_lat, current_lng, 
      rating_avg, total_jobs, last_location_update
    `)
    .eq('status', 'active')
    .eq('is_online', true);
  
  // 3. Filter by distance
  const nearbyPartners = partners.filter(p => {
    const dist = haversineDistance(
      p.current_lat, p.current_lng,
      booking.customer_addresses.lat, booking.customer_addresses.lng
    );
    return dist < 10; // 10km
  });
  
  if (nearbyPartners.length === 0) {
    // No partners available → notify customer, start search
    await supabase.from('bookings')
      .update({ state: 'broadcasting', broadcast_started_at: new Date() })
      .eq('id', booking_id);
    // Trigger delayed re-search
    return new Response(JSON.stringify({ dispatched: false, reason: 'no_partners' }));
  }
  
  // 4. Score partners (rating + distance + recency)
  const scoredPartners = nearbyPartners.map(p => ({
    ...p,
    score: (p.rating_avg * 20) - (haversineDistance(...) * 5) + (p.total_jobs * 0.1)
  })).sort((a, b) => b.score - a.score);
  
  // 5. Send job offer to top partner
  // (via FCM push + Realtime broadcast)
  const topPartner = scoredPartners[0];
  
  await supabase.from('bookings')
    .update({ 
      state: 'broadcasting',
      broadcast_started_at: new Date()
    })
    .eq('id', booking_id);
  
  // Notify partner via FCM (handled separately)
  await fetch(`${Deno.env.get('SUPABASE_URL')}/functions/v1/send-push`, {
    method: 'POST',
    body: JSON.stringify({
      user_id: topPartner.id,
      user_type: 'partner',
      type: 'job_offer',
      data: { booking_id }
    })
  });
  
  // 6. Set 60s timeout — if not accepted, try next partner
  setTimeout(async () => {
    const { data: current } = await supabase
      .from('bookings')
      .select('state')
      .eq('id', booking_id)
      .single();
    
    if (current?.state === 'broadcasting') {
      // Try next partner...
    }
  }, 60000);
  
  return new Response(JSON.stringify({ dispatched: true, partner_id: topPartner.id }));
});
```

### `send-push` (FCM Wrapper)

```typescript
// supabase/functions/send-push/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";
import admin from "https://esm.sh/firebase-admin@11";

admin.initializeApp({
  credential: admin.credential.cert({
    projectId: Deno.env.get('FCM_PROJECT_ID'),
    clientEmail: Deno.env.get('FCM_CLIENT_EMAIL'),
    privateKey: Deno.env.get('FCM_PRIVATE_KEY')!.replace(/\\n/g, '\n'),
  })
});

serve(async (req) => {
  const { user_id, user_type, type, data } = await req.json();
  
  const supabase = createClient(/* ... */);
  const { data: tokens } = await supabase
    .from('push_tokens')
    .select('fcm_token')
    .eq('user_id', user_id)
    .eq('user_type', user_type);
  
  if (!tokens?.length) return new Response('no tokens', { status: 404 });
  
  const messages = tokens.map(t => ({
    token: t.fcm_token,
    notification: getNotificationContent(type, data),  // {title, body}
    data: { ...data, type },
    android: { priority: 'high' as const },
    apns: { payload: { aps: { sound: 'default' } } }
  }));
  
  const response = await admin.messaging().sendEach(messages);
  return new Response(JSON.stringify(response), { 
    headers: { 'Content-Type': 'application/json' }
  });
});
```

---

## 10. RLS Policies (Security)

**Critical:** RLS ensures customers can ONLY access their own data.

```sql
-- Customers table
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile" ON customers
  FOR SELECT USING (id = auth.uid());

CREATE POLICY "Users can update own profile" ON customers
  FOR UPDATE USING (id = auth.uid());

-- Bookings table
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own bookings" ON bookings
  FOR SELECT USING (customer_id = auth.uid());

CREATE POLICY "Users can create own bookings" ON bookings
  FOR INSERT WITH CHECK (customer_id = auth.uid());

CREATE POLICY "Users can update own bookings" ON bookings
  FOR UPDATE USING (customer_id = auth.uid())
  WITH CHECK (
    customer_id = auth.uid() AND
    state IN ('cancelled')  -- only allow cancel via UPDATE
  );

-- Customer Addresses
ALTER TABLE customer_addresses ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users manage own addresses" ON customer_addresses
  FOR ALL USING (customer_id = auth.uid());

-- Services (public read, no write from client)
ALTER TABLE services ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Services are public" ON services
  FOR SELECT USING (is_active = true);

-- Service Addons (public read)
ALTER TABLE service_addons ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Addons are public" ON service_addons
  FOR SELECT USING (is_active = true);

-- Ratings
ALTER TABLE ratings ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own ratings" ON ratings
  FOR SELECT USING (customer_id = auth.uid());

CREATE POLICY "Users can create ratings" ON ratings
  FOR INSERT WITH CHECK (customer_id = auth.uid());

-- Push tokens
ALTER TABLE push_tokens ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users manage own push tokens" ON push_tokens
  FOR ALL USING (
    user_id = auth.uid() AND user_type = 'customer'
  );
```

---

## 11. Performance & Optimization

### Caching Strategy
```dart
// Cached services list (refresh daily)
@riverpod
class CachedServices extends _$CachedServices {
  @override
  Future<List<Service>> build() async {
    final box = await Hive.openBox<List>('services');
    final cached = box.get('all_services');
    
    if (cached != null) {
      // Return cached immediately, fetch fresh in background
      Future.microtask(() => _refresh());
      return cached.map(Service.fromJson).toList();
    }
    
    return _fetchFresh();
  }
  
  Future<List<Service>> _refresh() async {
    final fresh = await _fetchFresh();
    final box = await Hive.openBox<List>('services');
    await box.put('all_services', fresh.map((s) => s.toJson()).toList());
    state = AsyncData(fresh);
    return fresh;
  }
}
```

### Image Optimization
- Use Supabase Image Transformation API: `?width=300&height=200&quality=80`
- Cache images with `cached_network_image`
- Lazy load service images in lists

### Realtime Optimization
- Don't subscribe when screen is not visible (use `WidgetsBindingObserver`)
- Unsubscribe on `dispose()`
- Use selective column updates (only fetch what changed)

---

## 12. Push Notification Strategy

### Notification Types & Triggers

| Event | Title | Body | Priority |
|-------|-------|------|----------|
| Booking paid | ✅ Booking confirmed | "Your booking is confirmed for Wed 16:00" | Normal |
| Partner assigned | 👋 Partner assigned | "Raj Kumar will handle your service" | High |
| Partner en route | 🚗 Partner on the way | "Raj is heading to you. ETA 12 min" | High |
| Partner arrived | 📍 Partner arrived | "Raj has arrived. Share your PIN to start" | High |
| Service started | ✂️ Service in progress | "Your service has started" | Normal |
| Service complete | 🎉 Service complete | "Please rate your experience" | Normal |
| Booking cancelled | ❌ Booking cancelled | "Your booking was cancelled. €50 refunded" | Normal |
| Tip reminder | 💰 Add a tip? | "Show appreciation with a tip" | Low (marketing) |
| Re-book reminder | 🔄 Time for a cut? | "It's been 4 weeks. Book again!" | Low (marketing) |

### Quiet Hours
- Marketing notifications: 9 AM - 8 PM only
- Operational notifications (booking updates): 24/7
- Allow user to override in settings

---

## 13. Testing Strategy

### Unit Tests
- Repository mocks
- Provider state transitions
- Price calculations (addons, tips, fees)

### Widget Tests
- Each screen renders correctly
- Forms validate input
- Buttons trigger correct actions

### Integration Tests
- Full booking flow (sign up → browse → book → pay)
- Cancellation flow
- Rating flow
- Realtime updates (mock channel)

### Supabase Local Dev
```bash
# Install Supabase CLI
brew install supabase/tap/supabase

# Init local Supabase
supabase init

# Start local stack (Postgres, Auth, Realtime, Storage)
supabase start

# Run migrations
supabase db reset

# Generate TypeScript types
supabase gen types typescript --local > lib/types/database.types.dart
```

---

## 14. Build & Deploy

### Android (Play Store)

```bash
# 1. Generate signing key
keytool -genkey -v -keystore ~/salonathome-upload.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias salonathome

# 2. Configure signing in android/key.properties
storePassword=<password>
keyPassword=<password>
keyAlias=salonathome
storeFile=/Users/you/salonathome-upload.jks

# 3. Build release
flutter build appbundle --release

# 4. Upload to Play Console
```

### iOS (App Store)

```bash
# 1. Open Runner.xcworkspace
open ios/Runner.xcworkspace

# 2. Set up signing in Xcode (Apple Developer account)
# 3. Increment build number
# 4. Archive and upload
flutter build ipa --release
```

### CI/CD (GitHub Actions)

```yaml
# .github/workflows/build.yml
name: Build
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
      - run: flutter pub get
      - run: flutter test
      - run: flutter build apk --release
      - uses: actions/upload-artifact@v3
        with:
          name: app-release.apk
          path: build/app/outputs/flutter-apk/app-release.apk
```

---

## 15. Cost Estimate (Supabase)

| Resource | Free Tier | Pilot (100 bookings/day) | Scale (1000/day) |
|----------|-----------|---------------------------|-------------------|
| Database | 500 MB | 2 GB — €25/mo | 20 GB — €100/mo |
| Auth | 50K MAU | Within free | 100K+ MAU — €0.00325/MAU |
| Storage | 1 GB | 5 GB — €0.10/GB | 50 GB — €0.10/GB |
| Realtime | 200 concurrent | 500 concurrent — €10/mo | 5000+ — €100/mo |
| Edge Functions | 500K invocations | 2M invocations — €10/mo | 10M+ — €50/mo |
| **Total** | **€0** | **~€50-100/mo** | **~€300-500/mo** |

**Plus Stripe:** 1.4% + €0.25 per transaction (EU cards), 2.9% + €0.25 for international.

---

## 16. Build Roadmap

### Week 1-2: Foundation
- [ ] Supabase project setup
- [ ] Auth flow (phone + OTP)
- [ ] Customer profile creation
- [ ] Address CRUD

### Week 3-4: Browse & Book
- [ ] Home screen (services list)
- [ ] Service detail
- [ ] Schedule slot picker
- [ ] Edge function: get-available-slots

### Week 5-6: Payment
- [ ] Stripe integration
- [ ] Checkout flow
- [ ] Edge function: create-payment-intent
- [ ] Booking creation

### Week 7-8: Live Tracking
- [ ] Tracking screen
- [ ] Realtime subscriptions
- [ ] Map integration
- [ ] Push notifications

### Week 9-10: Post-Service
- [ ] Rating flow
- [ ] Booking history
- [ ] Support tickets
- [ ] PDF invoice generation

### Week 11-12: Polish
- [ ] Animations & transitions
- [ ] Empty states
- [ ] Error handling
- [ ] Beta testing with 10 users

---

## 17. Success Metrics

### North Star Metric
**Weekly Active Customers (WAC)** who completed a booking

### Supporting Metrics
1. **Activation rate:** Sign-up → first booking (target: 60% in 7 days)
2. **Time to first booking:** Median from sign-up (target: <10 min)
3. **Booking completion rate:** Started checkout → paid (target: 70%)
4. **Repeat rate:** % of customers with 2+ bookings (target: 40% in 30 days)
5. **Average rating:** Across all bookings (target: 4.7+)
6. **Cancellation rate:** Post-payment (target: <8%)
7. **App store rating:** (target: 4.5+)

### Analytics Events to Track
```dart
// Mixpanel/Amplitude events
analytics.track('app_opened');
analytics.track('onboarding_completed', { 'method': 'phone' });
analytics.track('service_viewed', { 'service_id': ..., 'category': 'men' });
analytics.track('addon_added', { 'addon_id': ..., 'service_id': ... });
analytics.track('slot_selected', { 'date': ..., 'time': ... });
analytics.track('checkout_started');
analytics.track('payment_completed', { 'amount_cents': ..., 'tip_cents': ... });
analytics.track('booking_cancelled', { 'state_at_cancel': ..., 'reason': ... });
analytics.track('rating_submitted', { 'stars': ..., 'comment_length': ... });
analytics.track('rebook_clicked', { 'from_booking_id': ... });
```

---

**End of Customer App Architecture Document**
