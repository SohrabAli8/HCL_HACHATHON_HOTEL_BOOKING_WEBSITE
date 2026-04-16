# Hotel Booking System — Frontend Generation Prompts

> **How to use this document:**  
> Copy each prompt one by one into your AI tool (ChatGPT / Gemini / Copilot).  
> Follow them in exact order — each prompt builds on the previous one.  
> Estimated time: **~1 hour** to generate complete frontend.

> **Tech Stack:** Angular 19 · TypeScript · Vanilla CSS (dark theme)  
> **Auth State:** Angular Signals (in AuthService)  
> **Other State:** Normal variables / subscriptions  
> **Backend:** ASP.NET 8 API running on `http://localhost:5000`

---

## PROMPT 1 — Angular Project Setup

> Copy and paste this prompt:

```
I am building the frontend for a Hotel Booking System using Angular.



1. Create a new Angular project called "hotel-booking-frontend" using Angular CLI
   - Use standalone components (no NgModules)
   - Enable routing
   - Use CSS for styling (not SCSS)
   - Use --skip-tests flag
   - Run in non-interactive mode

2. Navigate into the project

3. After project creation, create this folder structure inside src/app/:
   - components/login/
   - components/register/
   - components/navbar/
   - components/hotel-list/
   - components/room-list/
   - components/booking/
   - components/booking-success/
   - components/my-bookings/
   - components/admin-dashboard/
   - services/
   - guards/
   - interceptors/
   - models/

4. Also show me the command to generate each component using Angular CLI:
   - LoginComponent
   - RegisterComponent
   - NavbarComponent
   - HotelListComponent
   - RoomListComponent
   - BookingComponent
   - BookingSuccessComponent
   - MyBookingsComponent
   - AdminDashboardComponent

Use: ng generate component components/component-name --standalone --skip-tests

Give me only the terminal commands. I will run them myself.
```

---

## PROMPT 2 — TypeScript Models / Interfaces

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular.

My backend API returns these JSON shapes. I need TypeScript interfaces to match them.

Create ALL the following interface files:

1. models/user.model.ts
   export interface LoginRequest {
     email: string;
     password: string;
   }
   export interface RegisterRequest {
     name: string;
     email: string;
     password: string;
   }
   export interface AuthResponse {
     token: string;
     name: string;
     role: string;
   }

2. models/hotel.model.ts
   export interface Hotel {
     hotelId: number;
     name: string;
     location: string;
     description: string;
   }
   export interface CreateHotel {
     name: string;
     location: string;
     description: string;
   }

3. models/room.model.ts
   export interface Room {
     roomId: number;
     hotelId: number;
     roomType: string;
     pricePerNight: number;
     capacity: number;
     isActive: boolean;
     amenities: string[];
   }
   export interface CreateRoom {
     hotelId: number;
     roomType: string;
     pricePerNight: number;
     capacity: number;
     amenityIds: number[];
   }

4. models/booking.model.ts
   export interface CreateBooking {
     roomId: number;
     checkInDate: string;
     checkOutDate: string;
     couponCode?: string;
   }
   export interface BookingResponse {
     bookingId: number;
     reservationNumber: string;
     hotelName: string;
     roomType: string;
     checkInDate: string;
     checkOutDate: string;
     totalAmount: number;
     discountAmount: number;
     couponCode: string;
   }
   export interface BookingHistory {
     bookingId: number;
     reservationNumber: string;
     hotelName: string;
     hotelLocation: string;
     roomType: string;
     roomId: number;
     checkInDate: string;
     checkOutDate: string;
     totalAmount: number;
     discountAmount: number;
     couponCode: string;
     createdAt: string;
   }
   export interface AvailabilityResponse {
     isAvailable: boolean;
   }

5. models/coupon.model.ts
   export interface CouponResponse {
     isValid: boolean;
     code: string;
     discountPercentage: number;
   }
   export interface CreateCoupon {
     code: string;
     discountPercentage: number;
     expiryDate: string;
   }

6. models/search.model.ts
   export interface SearchResult {
     hotelId: number;
     hotelName: string;
     location: string;
     roomId: number;
     roomType: string;
     pricePerNight: number;
     capacity: number;
     amenities: string[];
   }
   export interface SearchFilters {
     location?: string;
     minPrice?: number;
     maxPrice?: number;
     amenities?: string;
     checkIn?: string;
     checkOut?: string;
   }

Generate all 6 files. Keep it clean and simple.
```

---

## PROMPT 3 — AuthService with Angular Signals

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular.

I need an AuthService that uses ANGULAR SIGNALS for reactive state management.

My backend API:
- POST http://localhost:5000/api/auth/register → body: { name, email, password } → returns: { message }
- POST http://localhost:5000/api/auth/login → body: { email, password } → returns: { token, name, role }

Create: services/auth.service.ts

Requirements:
- Injectable in root
- Use Angular's signal() for reactive state:
  - currentUser = signal<AuthResponse | null>(null)
  - isLoggedIn = computed(() => this.currentUser() !== null)
  - isAdmin = computed(() => this.currentUser()?.role === 'Admin')
  - userName = computed(() => this.currentUser()?.name || '')

- Constructor: inject HttpClient. On init, check localStorage for saved token and decode it to restore state.

- Methods:

  register(data: RegisterRequest): Observable<any>
    → POST to /api/auth/register

  login(data: LoginRequest): Observable<AuthResponse>
    → POST to /api/auth/login
    → On success: save token to localStorage, decode token to get name + role, update currentUser signal

  logout(): void
    → Remove token from localStorage
    → Set currentUser signal to null

  getToken(): string | null
    → Return token from localStorage

  private decodeToken(token: string): { userId: string, email: string, role: string, name: string }
    → Decode JWT manually (split by '.', base64 decode the payload part, parse JSON)
    → Extract claims: UserId, email, role
    → Note: the name might be stored in token or come from login response

- On service initialization (constructor):
  → Check if token exists in localStorage
  → If yes, decode it and set currentUser signal
  → This restores auth state on page refresh

IMPORTANT:
- Use signal() and computed() from @angular/core
- Store the FULL AuthResponse (token, name, role) in the signal
- The token in localStorage key should be "jwt_token"
- The login response name and role should be stored alongside token
- Store name and role in localStorage too: "user_name", "user_role"

Keep code simple. Fresher-level. Add comments explaining signals.
```

---

## PROMPT 4 — AuthGuard + AdminGuard

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular.

I have an AuthService with these signals:
- isLoggedIn: computed signal (boolean)
- isAdmin: computed signal (boolean)
- getToken(): returns token string or null

Create TWO route guards:

1. guards/auth.guard.ts
   - Use canActivate functional guard (Angular 16+ style, NOT class-based)
   - Inject AuthService using inject()
   - If user is logged in (isLoggedIn() is true) → return true
   - If user is NOT logged in → redirect to /login → return false
   - Use Router to redirect

2. guards/admin.guard.ts
   - Use canActivate functional guard
   - Inject AuthService using inject()
   - If user is logged in AND is admin (isAdmin() is true) → return true
   - If user is logged in but NOT admin → redirect to /hotels → return false
   - If user is NOT logged in → redirect to /login → return false

Use the modern Angular functional guard pattern:
export const authGuard: CanActivateFn = (route, state) => { ... }

Keep code simple. Fresher-level. Add comments.
```

---

## PROMPT 5 — AuthInterceptor (JWT Token Injection)

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular.

I need an HTTP interceptor that automatically attaches the JWT token to every outgoing API request.

I have an AuthService with: getToken(): string | null → returns JWT from localStorage

Create: interceptors/auth.interceptor.ts

Requirements:
- Use the modern Angular functional interceptor pattern (HttpInterceptorFn, NOT class-based)
- Inject AuthService using inject()
- For every outgoing HTTP request:
  a. Get token from AuthService.getToken()
  b. If token exists → clone the request and add header: Authorization: Bearer <token>
  c. If no token → pass the request as-is
- Return next(clonedRequest or originalRequest)

Use this pattern:
export const authInterceptor: HttpInterceptorFn = (req, next) => { ... }

ALSO show me how to register this interceptor in app.config.ts or the providers array:
- Use provideHttpClient(withInterceptors([authInterceptor]))

Keep code simple. Fresher-level. Add comments.
```

---

## PROMPT 6 — All Remaining Angular Services

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular.

My backend API runs at: http://localhost:5000

I already have AuthService. Now I need 5 more services. All these use NORMAL state management (no signals — signals are only in AuthService).

Create ALL 5 services:

1. services/hotel.service.ts
   - Injectable in root
   - Inject HttpClient
   - Base URL: http://localhost:5000/api
   - Methods:
     getAllHotels(): Observable<Hotel[]> → GET /api/hotels
     addHotel(data: CreateHotel): Observable<Hotel> → POST /api/hotels

2. services/room.service.ts
   - Methods:
     getRoomsByHotel(hotelId: number): Observable<Room[]> → GET /api/hotels/{hotelId}/rooms
     addRoom(data: CreateRoom): Observable<Room> → POST /api/rooms

3. services/search.service.ts
   - Methods:
     searchRooms(filters: SearchFilters): Observable<SearchResult[]> → GET /api/search
     → Build HttpParams dynamically: only add params that have values
     → If filters.location has value → add params "location"
     → If filters.minPrice has value → add params "minPrice"
     → If filters.maxPrice has value → add params "maxPrice"
     → If filters.amenities has value → add params "amenities"
     → If filters.checkIn has value → add params "checkIn"
     → If filters.checkOut has value → add params "checkOut"

4. services/booking.service.ts
   - Methods:
     createBooking(data: CreateBooking): Observable<BookingResponse> → POST /api/bookings
     checkAvailability(roomId: number, checkIn: string, checkOut: string): Observable<{isAvailable: boolean}> → GET /api/bookings/availability?roomId=...&checkIn=...&checkOut=...
     getMyBookings(): Observable<BookingHistory[]> → GET /api/bookings/my
     rebook(bookingId: number): Observable<BookingHistory> → POST /api/bookings/rebook/{bookingId}

5. services/coupon.service.ts
   - Methods:
     validateCoupon(code: string): Observable<CouponResponse> → POST /api/coupons/validate → body: { code }
     createCoupon(data: CreateCoupon): Observable<any> → POST /api/coupons
     toggleCoupon(couponId: number): Observable<any> → PUT /api/coupons/{couponId}/toggle

Import the model interfaces from ../models/ folder.
Keep code simple. No signals in these services — just normal HttpClient calls.
Fresher-level. Add comments.
```

---

## PROMPT 7 — App Routing + Configuration

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular.

I have these components:
- LoginComponent, RegisterComponent
- NavbarComponent
- HotelListComponent, RoomListComponent
- BookingComponent, BookingSuccessComponent
- MyBookingsComponent
- AdminDashboardComponent

I have these guards:
- authGuard (only logged-in users)
- adminGuard (only admin users)

I have AuthInterceptor (JWT token injection)

Create TWO files:

1. app.routes.ts — Define ALL routes:

   /login → LoginComponent (no guard)
   /register → RegisterComponent (no guard)
   /hotels → HotelListComponent (no guard) — this is also the DEFAULT route
   /hotels/:hotelId/rooms → RoomListComponent (no guard)
   /booking/:roomId → BookingComponent (authGuard)
   /booking-success → BookingSuccessComponent (authGuard)
   /my-bookings → MyBookingsComponent (authGuard)
   /admin → AdminDashboardComponent (adminGuard)
   ** (wildcard) → redirect to /hotels

2. app.config.ts — Configure providers:
   - provideRouter(routes)
   - provideHttpClient(withInterceptors([authInterceptor]))
   - provideAnimations() — if needed

3. Also update app.component.ts:
   - Import NavbarComponent
   - Template should be:
     <app-navbar></app-navbar>
     <router-outlet></router-outlet>
   - NavbarComponent appears on EVERY page

Keep code simple. Use standalone component imports. Fresher-level.
```

---

## PROMPT 8 — Global Styles (Dark Theme Design System)

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a PREMIUM DARK THEME.

Create the complete styles.css (global styles) file with:

1. CSS VARIABLES (design tokens):
   --bg-primary: #0f172a (deep navy — page backgrounds)
   --bg-secondary: #1e293b (dark slate — cards, panels)
   --bg-tertiary: #334155 (slate — inputs, hover states)
   --accent-primary: #3b82f6 (royal blue — buttons, links)
   --accent-success: #10b981 (emerald — available, success)
   --accent-warning: #f59e0b (amber — coupons, warnings)
   --accent-danger: #ef4444 (rose — errors, unavailable)
   --text-primary: #f8fafc (white — headings)
   --text-secondary: #94a3b8 (light gray — descriptions)
   --text-muted: #64748b (gray — placeholders)
   --border: #334155 (card borders)
   --gradient-primary: linear-gradient(135deg, #3b82f6, #8b5cf6) (blue to purple — CTA buttons)

2. GLOBAL RESETS:
   - Import Google Font: Inter (weights: 400, 500, 600, 700)
   - *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0 }
   - body: bg-primary background, text-primary color, Inter font, 14px
   - a: no underline, accent-primary color

3. REUSABLE CLASSES:

   .container: max-width 1280px, margin auto, padding 24px

   .card: bg-secondary, border-radius 16px, border 1px solid var(--border), padding 24px, transition transform 0.2s
   .card:hover: translateY(-4px), enhanced shadow

   .btn-primary: gradient-primary background, white text, border-radius 10px, padding 12px 24px, font-weight 600, cursor pointer, border none, transition transform 0.15s
   .btn-primary:hover: scale(1.02), brightness increase
   .btn-primary:disabled: opacity 0.5, cursor not-allowed

   .btn-secondary: bg-tertiary background, text-primary, similar padding/radius
   .btn-danger: accent-danger background

   .btn-outline: transparent background, 2px solid accent-primary border, accent-primary text

   .form-group: margin-bottom 16px
   .form-label: display block, text-secondary, font-size 12px, font-weight 500, margin-bottom 6px
   .form-input: width 100%, padding 12px, bg-tertiary background, border 1px solid var(--border), border-radius 8px, text-primary color, font-size 14px
   .form-input:focus: outline none, border-color accent-primary, box-shadow 0 0 0 3px rgba(59,130,246,0.2)
   .form-input::placeholder: text-muted

   .badge: display inline-block, padding 4px 12px, border-radius 20px, font-size 12px, font-weight 500
   .badge-success: bg accent-success with 20% opacity, accent-success text
   .badge-danger: bg accent-danger with 20% opacity, accent-danger text
   .badge-amenity: bg-tertiary background, text-secondary

   .error-text: accent-danger color, font-size 12px, margin-top 4px
   .success-text: accent-success color, font-size 14px

   .page-title: font-size 32px, font-weight 700, margin-bottom 24px
   .section-title: font-size 24px, font-weight 600, margin-bottom 16px

   .grid-3: display grid, grid-template-columns repeat(3, 1fr), gap 24px
   .grid-2: display grid, grid-template-columns repeat(2, 1fr), gap 24px

   .text-center: text-align center
   .text-muted: color text-muted
   .text-price: color accent-primary, font-size 20px, font-weight 700
   .text-strikethrough: text-decoration line-through, color text-muted

   .spinner: 20px × 20px, border 3px solid bg-tertiary, border-top accent-primary, border-radius 50%, animation spin 0.8s linear infinite
   @keyframes spin { to { transform: rotate(360deg) } }

   .empty-state: text-center, padding 48px, text-secondary

   .toast: position fixed, top 20px, right 20px, padding 16px, border-radius 12px, bg-secondary, z-index 1000, animation slideIn 0.3s
   .toast-success: border-left 4px solid accent-success
   .toast-error: border-left 4px solid accent-danger
   .toast-warning: border-left 4px solid accent-warning

4. RESPONSIVE:
   @media (max-width: 1023px): .grid-3 becomes 2 columns
   @media (max-width: 767px): .grid-3 and .grid-2 become 1 column

   @keyframes fadeInUp: from { opacity 0, translateY(20px) } to { opacity 1, translateY(0) }
   @keyframes slideIn: from { translateX(100%), opacity 0 } to { translateX(0), opacity 1 }

Make it look premium and modern. This is for a hackathon demo — first impression matters.
```

---

## PROMPT 9 — NavbarComponent

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a dark theme.

Create the NavbarComponent (standalone component).

I have AuthService with these SIGNALS:
- authService.isLoggedIn() → boolean signal
- authService.isAdmin() → boolean signal
- authService.userName() → string signal
- authService.logout() → method

The Navbar should have:

LAYOUT (horizontal bar, fixed to top):
LEFT: Logo "🏨 StayEase" (clickable → /hotels)
CENTER: Search input with placeholder "Search hotels by location..." 
RIGHT: Navigation links + auth buttons

NAVIGATION LINKS:
- "Hotels" → always visible → routerLink="/hotels"
- "My Bookings" → ONLY visible when logged in (not admin) → routerLink="/my-bookings"
- "Admin Panel" → ONLY visible when logged in AND admin → routerLink="/admin"

AUTH BUTTONS (right side):
- When NOT logged in: "Login" button → routerLink="/login"
- When logged in: Show "👤 {userName}" text + "Logout" button

SEARCH BEHAVIOR:
- When user types location and presses Enter → navigate to /hotels with queryParam: ?location=value
- Use Router.navigate(['/hotels'], { queryParams: { location: searchValue } })

CSS (in the component's CSS file):
- Navbar: fixed top, full width, bg-secondary background, height ~64px, z-index 100
- Flex layout: space-between, align-center, padding 0 24px
- Logo: bold, white, font-size 20px, cursor pointer, no text-decoration
- Search input: width 300px, bg-tertiary, border-radius 8px, padding 10px 16px, text-primary
- Nav links: text-secondary color, hover text-primary, gap 24px
- Active link: accent-primary color (use routerLinkActive)
- Login button: btn-outline style
- Logout button: btn-danger style, small
- Add a body padding-top of 80px to account for fixed navbar

Use the AuthService signals directly in the template with @if() blocks.
The component should import: RouterLink, RouterLinkActive, FormsModule, Router
Keep code simple. Fresher-level. Add comments.
```

---

## PROMPT 10 — LoginComponent + RegisterComponent

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a dark theme.

Create TWO components:

=== 1. LoginComponent ===

I have AuthService with: login(data: LoginRequest): Observable<AuthResponse>

TEMPLATE:
- Centered card (max-width 420px) on dark background
- Title: "🏨 Welcome Back"
- Subtitle: "Sign in to your account"
- Email input field (with icon prefix)
- Password input field (with toggle visibility button)
- "SIGN IN" button (full width, btn-primary class)
- Error message area (shows when login fails — red text)
- Link: "Don't have an account? Register here" → /register

BEHAVIOR:
- Use template-driven forms or reactive forms (FormsModule)
- On submit: validate fields not empty → call authService.login() → subscribe
- On success: navigate to /hotels using Router
- On error: show error message "Invalid email or password"
- While loading: show spinner in button, disable button
- Use a boolean: isLoading = false; errorMessage = '';

CSS:
- Center the card vertically and horizontally (min-height calc(100vh - 80px), display flex, align-items center, justify-content center)
- Card: bg-secondary, border-radius 16px, padding 40px, width 100%, max-width 420px, box-shadow
- All inputs use .form-input class
- Button uses .btn-primary class
- Link: accent-primary color

=== 2. RegisterComponent ===

I have AuthService with: register(data: RegisterRequest): Observable<any>

TEMPLATE:
- Same centered card layout as Login
- Title: "🏨 Create Account"
- Subtitle: "Join us to start booking"
- Full Name input
- Email input
- Password input (with toggle)
- Confirm Password input (with toggle)
- "CREATE ACCOUNT" button
- Error messages (field-level validation)
- Link: "Already have an account? Sign in here" → /login

BEHAVIOR:
- Validate: name not empty, email valid format, password min 6 chars, confirm password matches
- On submit: call authService.register() → subscribe
- On success: show success message, navigate to /login
- On error: show error message from backend

Use FormsModule with [(ngModel)] for two-way binding.
Keep code simple, clean. Fresher-level. Use global CSS classes from styles.css.
Add comments.
```

---

## PROMPT 11 — HotelListComponent (Main Page + Filters)

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a dark theme.

Create the HotelListComponent — this is the MAIN LANDING PAGE.

I have these services:
- hotelService.getAllHotels(): Observable<Hotel[]>
- searchService.searchRooms(filters: SearchFilters): Observable<SearchResult[]>

I have these models:
- Hotel: hotelId, name, location, description
- SearchResult: hotelId, hotelName, location, roomId, roomType, pricePerNight, capacity, amenities
- SearchFilters: location?, minPrice?, maxPrice?, amenities?, checkIn?, checkOut?

LAYOUT — Two columns:
LEFT SIDEBAR (width ~280px): Filter panel
RIGHT AREA (remaining width): Hotel cards grid (3 columns)

=== FILTER SIDEBAR ===
- Title: "🔍 Filters"
- Location: text input, placeholder "Enter city..."
- Min Price: number input, placeholder "Min ₹"
- Max Price: number input, placeholder "Max ₹"
- Check-in Date: date input (min = today)
- Check-out Date: date input (min = check-in + 1 day)
- Amenities: checkboxes — WiFi, AC, TV, Pool, Gym, Parking, Restaurant, Mini Bar
  (user can select MULTIPLE checkboxes)
- "APPLY FILTERS" button (btn-primary, full width)
- "Clear All" link (resets all filter fields and reloads all hotels)

=== HOTEL CARDS GRID ===
When NO filters applied → show all hotels from getAllHotels()
When filters applied → show results from searchRooms()

Each hotel card:
- Gradient placeholder top area (height 180px, use CSS gradient background as placeholder image)
- Hotel name (bold, white, 18px)
- Location with 📍 icon (text-secondary)
- Description (2 lines max, overflow hidden, text-muted)
- "Browse Rooms →" button (btn-primary, full width) → navigates to /hotels/:hotelId/rooms

BEHAVIOR:
- On page load: call getAllHotels() → display as cards → show skeleton loading while fetching
- When user clicks "APPLY FILTERS":
  → Build SearchFilters object from form fields
  → For amenities: join selected checkboxes as comma-separated string ("WiFi,AC,Pool")
  → Call searchService.searchRooms(filters)
  → Display SearchResult[] in the grid instead of Hotel[]
  → Show result count: "Showing {count} results"
- When filters are active, show SearchResult data (which includes room info)
  → Each card shows: hotelName, location, roomType, pricePerNight, amenities
  → "Browse Rooms" navigates using hotelId from the result
- "Clear All": reset all form fields, call getAllHotels() again
- Check ActivatedRoute queryParams on init — if location param exists (from navbar search), pre-fill location filter and auto-trigger search
- Empty state: "No hotels match your criteria. Try adjusting your filters."

Use:
- hotels: Hotel[] = []
- searchResults: SearchResult[] = []
- isFiltered: boolean = false
- isLoading: boolean = true

CSS:
- Two-column layout using flexbox (sidebar + grid area)
- Sidebar: bg-secondary, border-radius 16px, padding 24px, position sticky top 100px
- Grid: use .grid-3 class
- Cards: use .card class with hover effect
- Gradient placeholder: use linear-gradient(135deg, #1e3a5f, #2d1b69) as background
- On mobile (<768px): sidebar goes above grid as a collapsible section

Keep code simple. Fresher-level. Add comments.
Use ngModel for filter inputs. Use *ngIf or @if for conditional rendering.
```

---

## PROMPT 12 — RoomListComponent (Availability Check Flow)

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a dark theme.

Create the RoomListComponent — this shows rooms for a selected hotel with AVAILABILITY CHECK per room.

I have these services:
- roomService.getRoomsByHotel(hotelId: number): Observable<Room[]>
- bookingService.checkAvailability(roomId, checkIn, checkOut): Observable<{isAvailable: boolean}>
- authService.isLoggedIn(): signal<boolean>

I have: Room model: roomId, hotelId, roomType, pricePerNight, capacity, isActive, amenities (string[])

Route: /hotels/:hotelId/rooms → extract hotelId from ActivatedRoute params

LAYOUT:
- "← Back to Hotels" link at top (routerLink="/hotels")
- Hotel header section (hotel name + location — from route state or a quick hotel fetch)
- Section title: "AVAILABLE ROOMS ({count})"
- List of ROOM CARDS (full width, stacked vertically)

EACH ROOM CARD contains:
- Room type (bold, white, 18px) — "Deluxe Room"
- Price (accent-primary, large) — "₹2,500/night"
- Capacity — "👤 2 Guests" (text-secondary)
- Amenity badges — row of small pill badges like [WiFi] [AC] [TV]
- Check-in date input (type="date", min = today)
- Check-out date input (type="date", min = check-in date + 1 day)
- "CHECK AVAILABILITY" button
- Availability result area
- "BOOK NOW" button (only when available)

=== CRITICAL BEHAVIOR PER ROOM CARD (VERY IMPORTANT) ===

Each room card manages its OWN state independently. Use an object/map to track state per room.

Use a roomStates map to track each room's state:
roomStates: { [roomId: number]: { checkIn: string, checkOut: string, status: 'idle' | 'checking' | 'available' | 'unavailable', isChecked: boolean } } = {}

STEP 1 — Initial (no dates):
- Check-in and Check-out fields are EMPTY
- "CHECK AVAILABILITY" button is DISABLED (grayed out)
- No availability result shown
- No "BOOK NOW" button

STEP 2 — User enters BOTH dates:
- "CHECK AVAILABILITY" button becomes ENABLED (blue, clickable)

STEP 3 — User clicks "CHECK AVAILABILITY":
- Button shows spinner text "Checking..."
- Call bookingService.checkAvailability(roomId, checkIn, checkOut)

STEP 4a — Room IS available:
- Show ✅ green text: "Room is available!"
- "CHECK AVAILABILITY" button becomes DISABLED (already checked)
- 🔵 "BOOK NOW" button APPEARS below

STEP 4b — Room is NOT available:
- Show ❌ red text: "Room is not available for selected dates"
- "BOOK NOW" button does NOT appear

STEP 5 — User CHANGES dates after a check:
- Reset: hide availability result, hide BOOK NOW, re-enable CHECK AVAILABILITY
- User must check again

STEP 6 — User clicks "BOOK NOW":
- If logged in → navigate to /booking/:roomId?checkIn=...&checkOut=...
- If NOT logged in → navigate to /login

CSS:
- Room cards: bg-secondary, border-radius 16px, padding 24px, margin-bottom 16px
- Amenity badges: display inline-flex, gap 8px, bg-tertiary, border-radius 20px, padding 4px 12px, font-size 12px
- Date inputs: use .form-input, display inline with gap
- Buttons: CHECK AVAILABILITY (btn-outline when enabled, disabled when not), BOOK NOW (btn-primary)
- Available text: accent-success color
- Unavailable text: accent-danger color

Keep code simple. Fresher-level. Add comments explaining the state logic.
Use FormsModule with [(ngModel)] for date inputs.
```

---

## PROMPT 13 — BookingComponent + BookingSuccessComponent

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a dark theme.

Create TWO components:

=== 1. BookingComponent ===

Route: /booking/:roomId?checkIn=...&checkOut=...
Protected by AuthGuard (only logged-in users)

I have:
- roomService.getRoomsByHotel(hotelId) — but I need room details, so use a different approach
- bookingService.createBooking(data: CreateBooking): Observable<BookingResponse>
- bookingService.checkAvailability(roomId, checkIn, checkOut): Observable<{isAvailable: boolean}>
- couponService.validateCoupon(code: string): Observable<CouponResponse>

Extract from route: roomId from params, checkIn + checkOut from queryParams

LAYOUT — Two columns:
LEFT (60%): Booking form
RIGHT (40%): Price summary card (sticky on scroll)

LEFT — BOOKING FORM:
- Title: "📋 Booking Details"
- Room info display (read-only): Hotel name, Room type, Capacity
  (fetch room details on init)
- Check-in Date input (pre-filled from query params, editable)
- Check-out Date input (pre-filled from query params, editable)
- When user CHANGES dates → call checkAvailability again
  → If not available → show error "Room is not available for these dates"
  → Disable CONFIRM button
- Coupon Code section:
  → Text input + "APPLY" button (inline, side by side)
  → On click APPLY: call couponService.validateCoupon(code)
  → If valid: show ✅ "Coupon applied! {percentage}% off" (green)
  → If invalid: show ❌ "Invalid or expired coupon" (red)
- "CONFIRM BOOKING" button (full width, btn-primary, large)

RIGHT — PRICE SUMMARY CARD:
- Title: "💰 Price Breakdown"
- Room: {roomType}
- Price/Night: ₹{pricePerNight}
- Nights: {calculated from dates}
- Subtotal: ₹{price × nights}
- (If coupon applied):
  → Coupon: {code} ✅
  → Discount: -₹{amount} ({percentage}%)
- Original price with STRIKETHROUGH (text-strikethrough class)
- TOTAL: ₹{finalAmount} (large, green, bold)

CONFIRM BOOKING BEHAVIOR:
- Calculate: nights, subtotal, discount, total
- Create CreateBooking object: { roomId, checkInDate, checkOutDate, couponCode }
- Call bookingService.createBooking()
- On success: navigate to /booking-success with booking data (use Router state or query params)
- On error: show toast "Booking failed"
- While submitting: show spinner, disable button

STATE VARIABLES:
- room: Room | null
- checkIn, checkOut: string
- couponCode: string
- couponResponse: CouponResponse | null
- couponError: string
- isAvailable: boolean = true
- isSubmitting: boolean = false
- nights, subtotal, discount, total: number

CSS:
- Two column: flex layout, gap 32px
- Left: flex 3
- Right: flex 2, position sticky, top 100px
- Price summary card: bg-secondary, border-radius 16px, padding 24px
- Total amount: font-size 28px, accent-success color, font-weight 700
- Strikethrough price: text-decoration line-through, text-muted color
- On mobile: single column, price summary below form (not sticky)

=== 2. BookingSuccessComponent ===

Route: /booking-success
Protected by AuthGuard

Receives booking data from Router state (or localStorage fallback)

LAYOUT — Centered card:
- Large animated green checkmark (use CSS animation)
- Title: "BOOKING CONFIRMED! 🎉" (large, bold)
- Details card: Reservation #, Hotel, Room, Check-in, Check-out, Total
- Reservation number: large, accent-primary, bold
- Email notice: "📧 Confirmation email sent to your registered email"
- Two buttons:
  → "View My Bookings" (btn-outline) → /my-bookings
  → "Browse More Hotels" (btn-primary) → /hotels

CSS:
- Center everything vertically and horizontally
- Checkmark animation: grow from 0 to full size with bounce
- Use @keyframes for the animation

Keep code simple. Fresher-level. Use ngModel for inputs.
Add comments explaining price calculation logic.
```

---

## PROMPT 14 — MyBookingsComponent + AdminDashboardComponent

> Copy and paste this prompt:

```
I am building a Hotel Booking System frontend in Angular with a dark theme.

Create TWO components:

=== 1. MyBookingsComponent ===

Route: /my-bookings (protected by AuthGuard)

I have: bookingService.getMyBookings(): Observable<BookingHistory[]>

BookingHistory: bookingId, reservationNumber, hotelName, hotelLocation, roomType, roomId, checkInDate, checkOutDate, totalAmount, discountAmount, couponCode, createdAt

LAYOUT:
- Page title: "📜 My Bookings"
- List of booking cards (full width, stacked)

Each booking card:
- TOP ROW: Reservation # (left, bold, accent-primary) + Date range (right, text-secondary)
- MIDDLE ROW: Hotel name + location | Room type | Total amount (bold) | Coupon badge (if applied)
- BOTTOM ROW: Two buttons —
  → "🔄 Quick Rebook" (btn-outline)
  → "📋 Details" (btn-secondary) — toggles expanded view

Quick Rebook behavior:
- Navigate to /booking/:roomId (using the roomId from this booking)
- The BookingComponent will handle new date selection + availability check

Empty state (no bookings): "You haven't made any bookings yet." + "Browse Hotels →" link to /hotels

STATE: bookings: BookingHistory[] = []; isLoading: boolean = true;

CSS:
- Booking cards: bg-secondary, border-radius 16px, padding 24px, margin-bottom 16px
- Reservation #: font-size 16px, font-weight 600, accent-primary
- Amount: font-size 18px, font-weight 700
- Coupon badge: badge-amenity class with 🎟️ icon

=== 2. AdminDashboardComponent ===

Route: /admin (protected by AdminGuard)

I have:
- hotelService.getAllHotels(), hotelService.addHotel(data)
- roomService.getRoomsByHotel(hotelId), roomService.addRoom(data)
- couponService.createCoupon(data), couponService.toggleCoupon(id)

LAYOUT — Tab-based design:
Three tabs at top: 🏨 Hotels | 🛏️ Rooms | 🎟️ Coupons

Use a simple variable: activeTab: string = 'hotels'
Switch content based on activeTab using @if or *ngIf

=== HOTELS TAB ===
TOP: Add Hotel form (inline — all fields in one row + button)
- Hotel Name input (required)
- Location input (required)
- Description textarea (optional)
- "ADD HOTEL" button (btn-primary)
- On submit: call hotelService.addHotel() → refresh hotel list

BOTTOM: Hotels table
| ID | Name | Location | Description |

Load hotels using hotelService.getAllHotels()

=== ROOMS TAB ===
TOP: Add Room form
- Hotel dropdown (<select> populated from getAllHotels())
- Room Type input (text)
- Price Per Night input (number)
- Capacity input (number)
- Amenities checkboxes: WiFi, AC, TV, Pool, Gym, Parking, Restaurant, Mini Bar
  → Map selected to amenityIds (WiFi=1, AC=2, TV=3, Pool=4, Gym=5, Parking=6, Restaurant=7, Mini Bar=8)
- "ADD ROOM" button
- On submit: call roomService.addRoom() → show success

BOTTOM: Show added rooms (or just a success message for simplicity since we don't have a get-all-rooms API)

=== COUPONS TAB ===
TOP: Create Coupon form
- Code input (text)
- Discount % input (number, 1-100)
- Expiry Date input (date, must be future)
- "CREATE COUPON" button

BOTTOM: We don't have a GET all coupons API, so for hackathon simplicity, just show a success message after creation.
If you want, you can add a simple local array to track created coupons in the session.

TAB STYLING:
- Tab buttons: inline-flex, bg-tertiary for inactive, accent-primary for active
- Active tab has bottom border or different background
- Content area changes based on active tab

FORM VALIDATION: Show error messages on empty required fields.

STATE per tab:
- hotels: Hotel[] = []
- hotelForm: { name, location, description }
- roomForm: { hotelId, roomType, pricePerNight, capacity, amenityIds[] }
- couponForm: { code, discountPercentage, expiryDate }
- successMessage: string = ''

CSS:
- Tabs: display flex, gap 0, bg-tertiary rounded buttons, active tab bg accent-primary, padding 12px 24px
- Table: width 100%, border-collapse separate, bg-secondary cells
- Table header: text-secondary, font-weight 600, padding 12px
- Table row: padding 12px, border-bottom 1px solid var(--border)
- Form: bg-secondary card, border-radius 16px, padding 24px, margin-bottom 24px
- Inputs: use .form-input class

Keep code simple. Fresher-level. Use ngModel for form binding.
Add comments.
```

---

## ⚡ Quick Reference — Prompt Order

| # | Prompt | What It Generates | Time |
|---|--------|-------------------|:----:|
| 1 | Project Setup | Angular project + folder structure + component generation | 5 min |
| 2 | Models | 6 interface files (all TypeScript types) | 3 min |
| 3 | AuthService | Signals-based auth with JWT decode + localStorage | 5 min |
| 4 | Guards | AuthGuard + AdminGuard (functional style) | 3 min |
| 5 | Interceptor | JWT token auto-injection on all HTTP requests | 3 min |
| 6 | All Services | 5 services (Hotel, Room, Search, Booking, Coupon) | 5 min |
| 7 | Routing + Config | app.routes.ts + app.config.ts + app.component.ts | 3 min |
| 8 | Global Styles | Complete dark theme CSS design system | 5 min |
| 9 | Navbar | Fixed navbar with signals, search, conditional links | 5 min |
| 10 | Login + Register | Auth forms with validation + error handling | 7 min |
| 11 | Hotel List | Main page with filter sidebar + hotel card grid | 10 min |
| 12 | Room List | Room cards with availability check per card | 10 min |
| 13 | Booking + Success | Booking form with coupon + price calc + confirmation | 10 min |
| 14 | My Bookings + Admin | Booking history with rebook + Admin tabs (CRUD) | 10 min |
| | | **TOTAL** | **~84 min** |

---

## 🎯 Tips for Using These Prompts

1. **Copy each prompt EXACTLY** — they have all the context the AI needs
2. **Follow the order strictly** — later prompts reference earlier files
3. **After each prompt**, paste the generated code into the correct file before moving to the next
4. **If the AI gives overly complex code**, say: *"Simplify this. Use basic ngModel binding. No reactive forms. Fresher-friendly."*
5. **If CSS doesn't look right**, say: *"Use the CSS variables from my global styles.css. Follow the dark theme."*
6. **Angular Signals are ONLY in AuthService** — all other services use normal Observable + subscribe pattern
7. **Test after completing all prompts**: `ng serve` → open `http://localhost:4200`

---

## 📝 What You'll Have After All 14 Prompts

```
hotel-booking-frontend/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── login/          ← Login form
│   │   │   ├── register/       ← Register form
│   │   │   ├── navbar/         ← Global nav with search
│   │   │   ├── hotel-list/     ← Main page + filters + card grid
│   │   │   ├── room-list/      ← Room cards + availability check
│   │   │   ├── booking/        ← Booking form + coupon + price
│   │   │   ├── booking-success/← Confirmation page
│   │   │   ├── my-bookings/    ← History + rebook
│   │   │   └── admin-dashboard/← Admin CRUD tabs
│   │   ├── services/
│   │   │   ├── auth.service.ts      ← Signals-based auth
│   │   │   ├── hotel.service.ts
│   │   │   ├── room.service.ts
│   │   │   ├── search.service.ts
│   │   │   ├── booking.service.ts
│   │   │   └── coupon.service.ts
│   │   ├── guards/
│   │   │   ├── auth.guard.ts        ← Logged-in users only
│   │   │   └── admin.guard.ts       ← Admin only
│   │   ├── interceptors/
│   │   │   └── auth.interceptor.ts  ← JWT auto-injection
│   │   ├── models/                  ← 6 TypeScript interface files
│   │   ├── app.routes.ts            ← 8 routes with guards
│   │   ├── app.config.ts            ← Providers config
│   │   └── app.component.ts         ← Navbar + router-outlet
│   └── styles.css                   ← Complete dark theme design system
```

**Components:** 9 · **Services:** 6 · **Guards:** 2 · **Interceptor:** 1  
**Routes:** 8 · **Models:** 6 files  
**Auth state:** Angular Signals · **Other state:** Normal variables  
**Theme:** Premium dark mode with blue/purple accents
