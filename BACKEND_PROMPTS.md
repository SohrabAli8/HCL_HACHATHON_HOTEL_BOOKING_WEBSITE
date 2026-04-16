# Hotel Booking System — Backend Generation Prompts

> **How to use this document:**  
> Copy each prompt one by one into your AI tool (ChatGPT / Gemini / Copilot).  
> Follow them in exact order — each prompt builds on the previous one.  
> Estimated time: **~1 hour** to generate complete backend.

> **Tech Stack:** ASP.NET 8 Web API · MySQL · EF Core (Database First) · JWT Auth  
> **Package:** `MySql.EntityFrameworkCore` (Oracle's official — NOT Pomelo)

---

## PROMPT 1 — Project Setup + NuGet Packages

> Copy and paste this prompt:

```
I am building a Hotel Booking System backend using ASP.NET 8 Web API with MySQL database.

Give me the exact terminal commands to:

1. Create a new ASP.NET 8 Web API project called "HotelBookingAPI" (no HTTPS for local dev)
2. Navigate into the project folder
3. Install these NuGet packages:
   - MySql.EntityFrameworkCore (Oracle's official MySQL provider for EF Core — DO NOT use Pomelo)
   - Microsoft.EntityFrameworkCore.Design
   - Microsoft.EntityFrameworkCore.Tools
   - Microsoft.AspNetCore.Authentication.JwtBearer
   - BCrypt.Net-Next
   - Swashbuckle.AspNetCore (if not already included)

4. After showing commands, also create this folder structure inside the project:
   - Controllers/
   - Services/Interfaces/
   - Services/Implementations/
   - DTOs/Auth/
   - DTOs/Hotel/
   - DTOs/Room/
   - DTOs/Booking/
   - DTOs/Coupon/
   - DTOs/Search/
   - Middleware/

Give me only the terminal commands. I will run them myself.
Keep it simple — fresher level.
```

---

## PROMPT 2 — MySQL Database + All Tables + Seed Data

> Copy and paste this prompt:

```
I am building a Hotel Booking System. I need to create a MySQL database with 7 tables.

Give me a SINGLE SQL script that I can run in MySQL to create everything at once.

The script should:

1. CREATE DATABASE HotelBookingDB and USE it

2. Create these 7 tables IN THIS ORDER (foreign key dependencies):

TABLE 1 — Users:
- UserId (INT, PK, AUTO_INCREMENT)
- Name (VARCHAR 100, NOT NULL)
- Email (VARCHAR 150, NOT NULL, UNIQUE)
- PasswordHash (VARCHAR 255, NOT NULL)
- Role (VARCHAR 20, NOT NULL, DEFAULT 'User') — values: 'User' or 'Admin'
- CreatedAt (DATETIME, DEFAULT CURRENT_TIMESTAMP)

TABLE 2 — Hotels:
- HotelId (INT, PK, AUTO_INCREMENT)
- Name (VARCHAR 150, NOT NULL)
- Location (VARCHAR 150, NOT NULL)
- Description (TEXT, NULL)

TABLE 3 — Amenities:
- AmenityId (INT, PK, AUTO_INCREMENT)
- Name (VARCHAR 100, NOT NULL)

TABLE 4 — Rooms:
- RoomId (INT, PK, AUTO_INCREMENT)
- HotelId (INT, FK → Hotels, NOT NULL, ON DELETE CASCADE)
- RoomType (VARCHAR 100, NOT NULL)
- PricePerNight (DECIMAL 10,2, NOT NULL)
- Capacity (INT, NOT NULL)
- IsActive (BOOLEAN, DEFAULT TRUE)

TABLE 5 — RoomAmenities (junction table):
- RoomId (INT, FK → Rooms, ON DELETE CASCADE)
- AmenityId (INT, FK → Amenities, ON DELETE CASCADE)
- PRIMARY KEY (RoomId, AmenityId)

TABLE 6 — Coupons:
- CouponId (INT, PK, AUTO_INCREMENT)
- Code (VARCHAR 50, NOT NULL, UNIQUE)
- DiscountPercentage (DECIMAL 5,2, NOT NULL)
- ExpiryDate (DATE, NOT NULL)
- IsActive (BOOLEAN, DEFAULT TRUE)

TABLE 7 — Bookings:
- BookingId (INT, PK, AUTO_INCREMENT)
- UserId (INT, FK → Users, NOT NULL)
- RoomId (INT, FK → Rooms, NOT NULL)
- CheckInDate (DATE, NOT NULL)
- CheckOutDate (DATE, NOT NULL)
- TotalAmount (DECIMAL 10,2, NOT NULL)
- CouponId (INT, FK → Coupons, NULLABLE)
- DiscountAmount (DECIMAL 10,2, DEFAULT 0)
- ReservationNumber (VARCHAR 100, NOT NULL, UNIQUE)
- CreatedAt (DATETIME, DEFAULT CURRENT_TIMESTAMP)

3. After CREATE TABLEs, add seed data:

SEED Hotels (4 hotels):
- Taj Palace, Delhi, "A luxury 5-star hotel with world-class amenities and spa."
- Marriott Mumbai, Mumbai, "Premium business hotel in the heart of Mumbai."
- Grand Hyatt, Pune, "Modern hotel with conference facilities and rooftop pool."
- Oberoi Palace, Jaipur, "Heritage hotel with royal architecture and gardens."

SEED Amenities (8):
- WiFi, AC, TV, Pool, Gym, Parking, Restaurant, Mini Bar

SEED Rooms (9 rooms across 4 hotels with varied types and prices from ₹1200 to ₹8000)

SEED RoomAmenities (map amenities to rooms — give each room 2-8 amenities)

SEED Coupons (3):
- SAVE20 (20% off, expires 2026-12-31)
- SUMMER15 (15% off, expires 2026-06-30)
- WELCOME10 (10% off, expires 2026-09-30)

Give me the complete SQL script in one block that I can copy-paste and run.
```

---

## PROMPT 3 — Scaffold Models from Database

> Copy and paste this prompt:

```
I have a MySQL database called "HotelBookingDB" with 7 tables (Users, Hotels, Amenities, Rooms, RoomAmenities, Coupons, Bookings).

I am using ASP.NET 8 with MySql.EntityFrameworkCore package (Oracle's official provider — NOT Pomelo).

Give me:

1. The exact scaffold command to reverse-engineer my MySQL database into C# model classes.
   - Output models to a "Models" folder
   - Output DbContext to a "Data" folder
   - Name the context class "AppDbContext"
   - Use MySql.EntityFrameworkCore as the provider
   - My connection string: Server=localhost;Database=HotelBookingDB;User=root;Password=root;

2. After scaffolding, show me what the generated files will look like. Show me the approximate generated code for:
   - User.cs
   - Hotel.cs
   - Room.cs (with navigation properties)
   - Amenity.cs
   - RoomAmenity.cs
   - Booking.cs (with navigation properties)
   - Coupon.cs
   - AppDbContext.cs

Note: I am using MySql.EntityFrameworkCore provider, NOT Pomelo. The provider name for scaffold command should be "MySql.EntityFrameworkCore".

Keep code simple and beginner-friendly.
```

---

## PROMPT 4 — All DTOs (Request + Response Models)

> Copy and paste this prompt:

```
I am building a Hotel Booking System API in ASP.NET 8.

I have these scaffolded models from my database: User, Hotel, Room, Amenity, RoomAmenity, Booking, Coupon.

Now I need DTOs (Data Transfer Objects) to control what data goes in and out of my API.
I do NOT want to expose my database models directly to the frontend.

Create ALL the following DTO classes with proper data annotations ([Required], [EmailAddress], [Range], etc.):

1. DTOs/Auth/RegisterDto.cs
   - Name (string, required, min 2 chars)
   - Email (string, required, valid email)
   - Password (string, required, min 6 chars)

2. DTOs/Auth/LoginDto.cs
   - Email (string, required)
   - Password (string, required)

3. DTOs/Auth/AuthResponseDto.cs
   - Token (string)
   - Name (string)
   - Role (string)

4. DTOs/Hotel/CreateHotelDto.cs
   - Name (string, required)
   - Location (string, required)
   - Description (string, optional)

5. DTOs/Hotel/HotelDto.cs
   - HotelId (int)
   - Name (string)
   - Location (string)
   - Description (string)

6. DTOs/Room/CreateRoomDto.cs
   - HotelId (int, required)
   - RoomType (string, required)
   - PricePerNight (decimal, required, > 0)
   - Capacity (int, required, >= 1)
   - AmenityIds (List<int>, optional)

7. DTOs/Room/RoomDto.cs
   - RoomId, HotelId, RoomType, PricePerNight, Capacity, IsActive
   - Amenities (List<string>) — just the amenity names like ["WiFi", "AC"]

8. DTOs/Booking/CreateBookingDto.cs
   - RoomId (int, required)
   - CheckInDate (DateTime, required)
   - CheckOutDate (DateTime, required)
   - CouponCode (string, optional)

9. DTOs/Booking/BookingResponseDto.cs
   - BookingId, ReservationNumber, HotelName, RoomType
   - CheckInDate, CheckOutDate, TotalAmount, DiscountAmount, CouponCode

10. DTOs/Booking/BookingHistoryDto.cs
    - BookingId, ReservationNumber, HotelName, HotelLocation, RoomType, RoomId
    - CheckInDate, CheckOutDate, TotalAmount, DiscountAmount, CouponCode, CreatedAt

11. DTOs/Coupon/CreateCouponDto.cs
    - Code (string, required)
    - DiscountPercentage (decimal, required, 1-100)
    - ExpiryDate (DateTime, required)

12. DTOs/Coupon/ValidateCouponDto.cs
    - Code (string, required)

13. DTOs/Coupon/CouponResponseDto.cs
    - IsValid (bool)
    - Code (string)
    - DiscountPercentage (decimal)

14. DTOs/Search/SearchResultDto.cs
    - HotelId, HotelName, Location
    - RoomId, RoomType, PricePerNight, Capacity
    - Amenities (List<string>)

Generate all 14 DTO files. Keep code simple and clean — fresher level.
Use data annotation attributes for validation.
```

---

## PROMPT 5 — Program.cs (Complete Configuration)

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8 Web API with MySQL.

I have:
- Models in /Models folder (scaffolded from DB)
- AppDbContext in /Data folder
- DTOs in /DTOs folder
- I will create Services and Controllers next

Now generate my complete Program.cs file that configures:

1. DATABASE: Register AppDbContext with MySql.EntityFrameworkCore
   Connection string from appsettings.json: "Server=localhost;Database=HotelBookingDB;User=root;Password=root;"
   Use: builder.Services.AddDbContext with UseMySQL() method (MySql.EntityFrameworkCore uses UseMySQL, NOT UseMySql)

2. DEPENDENCY INJECTION: Register these services as Scoped:
   - IAuthService → AuthService
   - IHotelService → HotelService
   - IRoomService → RoomService
   - ISearchService → SearchService
   - IBookingService → BookingService
   - ICouponService → CouponService
   - IEmailService → EmailService

3. JWT AUTHENTICATION:
   - Read Key, Issuer, Audience from appsettings.json "Jwt" section
   - Configure JwtBearer authentication
   - Validate: Issuer, Audience, Lifetime, SigningKey

4. CORS:
   - Allow origin: http://localhost:4200 (Angular)
   - Allow methods: GET, POST, PUT, DELETE, OPTIONS
   - Allow headers: Content-Type, Authorization
   - Allow credentials: true

5. SWAGGER:
   - Add Swagger with JWT support (so I can test authenticated APIs from Swagger UI)
   - Add a "Bearer" security definition

6. RATE LIMITING:
   - Fixed window rate limiter: 100 requests per 1 minute per IP

7. MIDDLEWARE PIPELINE (in this order):
   - CORS
   - Rate Limiter
   - Authentication
   - Authorization
   - Swagger (in development only)
   - Map Controllers

8. Also generate the appsettings.json with:
   - ConnectionStrings.DefaultConnection
   - Jwt.Key (at least 32 characters)
   - Jwt.Issuer = "HotelBookingAPI"
   - Jwt.Audience = "HotelBookingClient"
   - Jwt.ExpiryHours = 24

Keep code simple, clean, and beginner-friendly.
Add comments explaining each section.
This is for a fresher-level hackathon project.
```

---

## PROMPT 6 — AuthService (JWT + BCrypt)

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

I have:
- User model (scaffolded): UserId, Name, Email, PasswordHash, Role, CreatedAt
- AppDbContext with Users DbSet
- DTOs: RegisterDto (Name, Email, Password), LoginDto (Email, Password), AuthResponseDto (Token, Name, Role)
- JWT settings in appsettings.json: Key, Issuer, Audience, ExpiryHours

Now create:

1. Services/Interfaces/IAuthService.cs
   - Task<string> RegisterUser(RegisterDto dto)
   - Task<AuthResponseDto> LoginUser(LoginDto dto)

2. Services/Implementations/AuthService.cs

AuthService needs:
- Constructor injection: AppDbContext, IConfiguration
- RegisterUser method:
  a. Check if email already exists → if yes, throw exception "Email already registered"
  b. Hash password using BCrypt.Net.BCrypt.HashPassword()
  c. Create User object with Role = "User"
  d. Add to DB and save
  e. Return success message

- LoginUser method:
  a. Find user by email → if not found, throw exception "Invalid credentials"
  b. Verify password using BCrypt.Net.BCrypt.Verify() → if wrong, throw exception
  c. Generate JWT token with claims: UserId, Email, Role
  d. JWT token settings: read Key, Issuer, Audience, ExpiryHours from IConfiguration
  e. Return AuthResponseDto with token, name, role

Keep code simple and beginner-friendly.
Use try-catch where needed.
Add comments explaining each step.
This is for a fresher-level HCL hackathon.
```

---

## PROMPT 7 — HotelService + RoomService

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

I have these scaffolded models:
- Hotel: HotelId, Name, Location, Description
- Room: RoomId, HotelId, RoomType, PricePerNight, Capacity, IsActive (has navigation: Hotel, RoomAmenities)
- Amenity: AmenityId, Name
- RoomAmenity: RoomId, AmenityId (has navigation: Room, Amenity)

I have AppDbContext with DbSets: Hotels, Rooms, Amenities, RoomAmenities

I have these DTOs:
- CreateHotelDto: Name, Location, Description
- HotelDto: HotelId, Name, Location, Description
- CreateRoomDto: HotelId, RoomType, PricePerNight, Capacity, AmenityIds (List<int>)
- RoomDto: RoomId, HotelId, RoomType, PricePerNight, Capacity, IsActive, Amenities (List<string>)

Now create:

1. Services/Interfaces/IHotelService.cs
   - Task<List<HotelDto>> GetAllHotels()
   - Task<HotelDto> AddHotel(CreateHotelDto dto)

2. Services/Implementations/HotelService.cs
   - Constructor: inject AppDbContext
   - GetAllHotels: fetch all hotels, map to HotelDto list
   - AddHotel: create Hotel from dto, save to DB, return HotelDto

3. Services/Interfaces/IRoomService.cs
   - Task<List<RoomDto>> GetRoomsByHotel(int hotelId)
   - Task<RoomDto> AddRoom(CreateRoomDto dto)

4. Services/Implementations/RoomService.cs
   - Constructor: inject AppDbContext
   - GetRoomsByHotel:
     a. Check hotel exists → if not, throw exception
     b. Fetch rooms WHERE HotelId matches AND IsActive = true
     c. Include RoomAmenities → ThenInclude Amenity
     d. Map to RoomDto — convert amenities to List<string> of names
   - AddRoom:
     a. Check hotel exists → if not, throw exception
     b. Create Room entity
     c. For each amenityId in dto.AmenityIds, create RoomAmenity entry
     d. Save to DB
     e. Return RoomDto

Keep code simple and beginner-friendly.
Use LINQ for queries. Add comments.
This is for a fresher-level HCL hackathon.
```

---

## PROMPT 8 — SearchService (Dynamic Filtering)

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

I need a SearchService that implements DYNAMIC FILTERING — the user can apply any combination of filters (one, multiple, or none). Only apply a filter if the user provides that parameter.

I have these models (scaffolded):
- Hotel: HotelId, Name, Location, Description
- Room: RoomId, HotelId, RoomType, PricePerNight, Capacity, IsActive (navigation: Hotel, RoomAmenities, Bookings)
- Amenity: AmenityId, Name
- RoomAmenity: RoomId, AmenityId (navigation: Room, Amenity)
- Booking: BookingId, RoomId, CheckInDate, CheckOutDate

I have: SearchResultDto: HotelId, HotelName, Location, RoomId, RoomType, PricePerNight, Capacity, Amenities (List<string>)

Now create:

1. Services/Interfaces/ISearchService.cs
   - Task<List<SearchResultDto>> SearchRooms(string? location, decimal? minPrice, decimal? maxPrice, string? amenities, DateTime? checkIn, DateTime? checkOut)

2. Services/Implementations/SearchService.cs
   - Constructor: inject AppDbContext
   - SearchRooms method:

   Step-by-step logic:
   a. Start with base query: all active rooms, Include Hotel, Include RoomAmenities.ThenInclude(Amenity)
   b. Use .AsQueryable() to build dynamic query

   c. IF location is not null/empty:
      → filter: WHERE room.Hotel.Location == location (case-insensitive)

   d. IF minPrice has value:
      → filter: WHERE room.PricePerNight >= minPrice

   e. IF maxPrice has value:
      → filter: WHERE room.PricePerNight <= maxPrice

   f. IF amenities is not null/empty:
      → Split amenities by comma into a list
      → filter: WHERE room has at least one matching amenity

   g. IF checkIn AND checkOut both have values:
      → filter: EXCLUDE rooms that have any overlapping booking
      → Overlap logic: existing.CheckInDate < requestedCheckOut AND existing.CheckOutDate > requestedCheckIn

   h. Execute query with .ToListAsync()
   i. Map results to List<SearchResultDto>
   j. Return results

Keep code simple. Use IQueryable for dynamic building.
Add comments explaining each filter step.
This is for a fresher-level hackathon — clean, readable code.
```

---

## PROMPT 9 — BookingService (Core Booking Logic)

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

The BookingService is the CORE of my system. It handles: creating bookings, checking availability, fetching booking history, and rebooking.

I have these models (scaffolded):
- User: UserId, Name, Email, PasswordHash, Role
- Room: RoomId, HotelId, RoomType, PricePerNight, Capacity, IsActive (navigation: Hotel)
- Booking: BookingId, UserId, RoomId, CheckInDate, CheckOutDate, TotalAmount, CouponId (nullable), DiscountAmount, ReservationNumber, CreatedAt (navigation: User, Room, Coupon)
- Coupon: CouponId, Code, DiscountPercentage, ExpiryDate, IsActive

I have these DTOs:
- CreateBookingDto: RoomId, CheckInDate, CheckOutDate, CouponCode (optional)
- BookingResponseDto: BookingId, ReservationNumber, HotelName, RoomType, CheckInDate, CheckOutDate, TotalAmount, DiscountAmount, CouponCode
- BookingHistoryDto: BookingId, ReservationNumber, HotelName, HotelLocation, RoomType, RoomId, CheckInDate, CheckOutDate, TotalAmount, DiscountAmount, CouponCode, CreatedAt

Now create:

1. Services/Interfaces/IBookingService.cs with these methods:
   - Task<BookingResponseDto> CreateBooking(CreateBookingDto dto, int userId)
   - Task<bool> CheckAvailability(int roomId, DateTime checkIn, DateTime checkOut)
   - Task<List<BookingHistoryDto>> GetUserBookings(int userId)
   - Task<BookingHistoryDto> Rebook(int bookingId, int userId)

2. Services/Implementations/BookingService.cs

   Constructor: inject AppDbContext

   METHOD: CreateBooking(dto, userId) — THIS IS THE MOST IMPORTANT METHOD
   Step by step:
   a. Validate: CheckInDate must be >= today
   b. Validate: CheckOutDate must be > CheckInDate
   c. Find room by RoomId → include Hotel → if not found or not active → throw "Room not found"
   d. Check availability: query Bookings for overlapping dates on this room
      → If overlap exists → throw "Room is not available for selected dates"
   e. Handle coupon (if CouponCode is provided):
      - Find coupon by code
      - Check it exists, is active, not expired
      - If valid → calculate discount
      - If invalid → ignore (don't apply discount, don't throw error)
   f. Calculate price:
      - nights = (CheckOutDate - CheckInDate).Days
      - subtotal = room.PricePerNight * nights
      - discount = subtotal * (coupon.DiscountPercentage / 100) — if coupon valid
      - totalAmount = subtotal - discount
   g. Generate reservation number: "RES-" + Guid.NewGuid().ToString().Substring(0, 8).ToUpper()
   h. Create Booking entity with all fields
   i. Save to DB
   j. Map to BookingResponseDto and return

   METHOD: CheckAvailability(roomId, checkIn, checkOut)
   - Query Bookings WHERE RoomId matches AND dates overlap
   - Overlap: existingCheckIn < checkOut AND existingCheckOut > checkIn
   - Return true if NO overlap found, false if overlap exists

   METHOD: GetUserBookings(userId)
   - Query Bookings WHERE UserId matches
   - Include Room → ThenInclude Hotel
   - Include Coupon
   - Order by CreatedAt descending (newest first)
   - Map to List<BookingHistoryDto>

   METHOD: Rebook(bookingId, userId)
   - Find booking WHERE BookingId matches AND UserId matches
   - If not found → throw "Booking not found"
   - Include Room → Hotel
   - Map to BookingHistoryDto and return (frontend will use this to start a new booking)

Keep code simple, clean, beginner-friendly.
Add comments explaining each step.
Handle errors with clear exception messages.
This is for a fresher-level HCL hackathon.
```

---

## PROMPT 10 — CouponService + EmailService

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

I need two more services: CouponService and EmailService.

MODELS:
- Coupon: CouponId, Code, DiscountPercentage, ExpiryDate, IsActive

DTOs:
- CreateCouponDto: Code (required), DiscountPercentage (required, 1-100), ExpiryDate (required)
- ValidateCouponDto: Code (required)
- CouponResponseDto: IsValid (bool), Code (string), DiscountPercentage (decimal)

Create:

1. Services/Interfaces/ICouponService.cs
   - Task<CouponResponseDto> ValidateCoupon(string code)
   - Task<CouponResponseDto> CreateCoupon(CreateCouponDto dto)
   - Task<string> ToggleCoupon(int couponId)

2. Services/Implementations/CouponService.cs
   - Constructor: inject AppDbContext
   
   ValidateCoupon(code):
   a. Find coupon by code (case-insensitive)
   b. If not found → return { IsValid: false }
   c. If found but not active → return { IsValid: false }
   d. If found but expired (ExpiryDate < today) → return { IsValid: false }
   e. If valid → return { IsValid: true, Code, DiscountPercentage }

   CreateCoupon(dto):
   a. Check if code already exists → throw "Coupon code already exists"
   b. Create Coupon entity, IsActive = true
   c. Save to DB and return

   ToggleCoupon(couponId):
   a. Find coupon by id → if not found, throw
   b. Flip IsActive (true → false, false → true)
   c. Save and return "Coupon activated" or "Coupon deactivated"

3. Services/Interfaces/IEmailService.cs
   - Task SendBookingConfirmation(string email, string reservationNumber, string hotelName, string roomType, DateTime checkIn, DateTime checkOut, decimal totalAmount)

4. Services/Implementations/EmailService.cs
   - This is a MOCK email service for hackathon demo
   - Just log to console: Console.WriteLine($"📧 EMAIL SENT to {email}")
   - Print booking details to console
   - Return Task.CompletedTask
   - Add a comment: "Replace with real SMTP implementation for production"

Keep code simple and clean. Fresher-level.
Add comments.
```

---

## PROMPT 11 — All 6 Controllers

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

I have these services (already created):
- IAuthService: RegisterUser(RegisterDto), LoginUser(LoginDto)
- IHotelService: GetAllHotels(), AddHotel(CreateHotelDto)
- IRoomService: GetRoomsByHotel(hotelId), AddRoom(CreateRoomDto)
- ISearchService: SearchRooms(location?, minPrice?, maxPrice?, amenities?, checkIn?, checkOut?)
- IBookingService: CreateBooking(CreateBookingDto, userId), CheckAvailability(roomId, checkIn, checkOut), GetUserBookings(userId), Rebook(bookingId, userId)
- ICouponService: ValidateCoupon(code), CreateCoupon(CreateCouponDto), ToggleCoupon(couponId)

Now create ALL 6 controllers. Controllers should be THIN — only receive request, call service, return response. NO business logic in controllers.

1. Controllers/AuthController.cs
   Route: api/auth
   - POST /register → calls AuthService.RegisterUser(dto) → returns 201
   - POST /login → calls AuthService.LoginUser(dto) → returns 200 with token

2. Controllers/HotelsController.cs
   Route: api/hotels
   - GET / → returns all hotels → 200
   - POST / → [Authorize(Roles = "Admin")] → calls AddHotel → returns 201

3. Controllers/RoomsController.cs
   Route: api
   - GET /hotels/{hotelId}/rooms → calls GetRoomsByHotel → returns 200
   - POST /rooms → [Authorize(Roles = "Admin")] → calls AddRoom → returns 201

4. Controllers/SearchController.cs
   Route: api/search
   - GET / → accepts query parameters (location, minPrice, maxPrice, amenities, checkIn, checkOut) → all OPTIONAL → calls SearchRooms → returns 200

5. Controllers/BookingsController.cs
   Route: api/bookings
   - POST / → [Authorize] → extract userId from JWT claims → calls CreateBooking → returns 201
   - GET /availability → accepts query params (roomId, checkIn, checkOut) → calls CheckAvailability → returns 200
   - GET /my → [Authorize] → extract userId from JWT → calls GetUserBookings → returns 200
   - POST /rebook/{bookingId} → [Authorize] → extract userId → calls Rebook → returns 200

6. Controllers/CouponsController.cs
   Route: api/coupons
   - POST /validate → [Authorize] → calls ValidateCoupon → returns 200
   - POST / → [Authorize(Roles = "Admin")] → calls CreateCoupon → returns 201
   - PUT /{id}/toggle → [Authorize(Roles = "Admin")] → calls ToggleCoupon → returns 200

IMPORTANT for extracting userId from JWT in controllers:
Use: int userId = int.Parse(User.FindFirst("UserId")?.Value ?? "0");

For ALL controllers:
- Use [ApiController] attribute
- Use [Route("api/[controller]")] or explicit routes
- Use try-catch in each action: catch exceptions → return appropriate status code (400, 404, 500)
- Return Ok(), Created(), BadRequest(), NotFound() as appropriate

Keep code simple, clean. Add comments.
Fresher-level for HCL hackathon.
```

---

## PROMPT 12 — Global Exception Middleware + Final Wiring

> Copy and paste this prompt:

```
I am building a Hotel Booking System in ASP.NET 8.

I need two final things:

1. Middleware/GlobalExceptionMiddleware.cs
   - Catches ALL unhandled exceptions globally
   - Returns standardized JSON error responses:
     - KeyNotFoundException → 404 { "error": message }
     - UnauthorizedAccessException → 401 { "error": message }
     - InvalidOperationException → 400 { "error": message }
     - ArgumentException → 400 { "error": message }
     - Any other exception → 500 { "error": "Something went wrong" }
   - Logs the exception to Console.WriteLine
   - Uses a try-catch pattern with RequestDelegate

2. Show me how to register this middleware in Program.cs:
   - app.UseMiddleware<GlobalExceptionMiddleware>() — place it FIRST in the pipeline (before CORS, Auth)

3. Also create a simple test checklist that I can use to verify my API:

   TEST CHECKLIST (using Swagger):
   a. POST /api/auth/register — register a user
   b. POST /api/auth/login — login and get JWT token
   c. In Swagger, click Authorize → paste "Bearer <token>"
   d. GET /api/hotels — should return seed hotels
   e. GET /api/hotels/1/rooms — should return rooms with amenities
   f. GET /api/search?location=Delhi — should return Delhi hotels/rooms
   g. GET /api/search?minPrice=1000&maxPrice=3000 — should filter by price
   h. GET /api/bookings/availability?roomId=1&checkIn=2026-04-10&checkOut=2026-04-12 — should return isAvailable
   i. POST /api/bookings — book a room
   j. POST /api/coupons/validate — validate SAVE20 coupon
   k. GET /api/bookings/my — see booking history
   l. POST /api/auth/login with admin credentials → get admin token
   m. POST /api/hotels — add hotel (admin only)
   n. POST /api/rooms — add room (admin only)
   o. POST /api/coupons — create coupon (admin only)

Keep code simple, clean. Fresher-level.
```

---

## ⚡ Quick Reference — Prompt Order

| # | Prompt | What It Generates | Estimated Time |
|---|--------|-------------------|:--------------:|
| 1 | Project Setup | Terminal commands, folder structure | 5 min |
| 2 | MySQL Database | Complete SQL script (tables + seed data) | 5 min |
| 3 | Scaffold Models | EF Core scaffold command + generated models | 5 min |
| 4 | All DTOs | 14 DTO files with validations | 5 min |
| 5 | Program.cs | Complete config (DI, JWT, CORS, Swagger, Rate Limiting) | 10 min |
| 6 | AuthService | Registration + Login + JWT generation | 5 min |
| 7 | Hotel + Room Services | CRUD for hotels and rooms with amenities | 5 min |
| 8 | SearchService | Dynamic multi-criteria filtering | 5 min |
| 9 | BookingService | Core booking flow + availability + history + rebook | 10 min |
| 10 | Coupon + Email Services | Coupon validation + mock email | 5 min |
| 11 | All Controllers | 6 controllers, 15 endpoints | 5 min |
| 12 | Middleware + Testing | Global error handler + test checklist | 5 min |
| | | **TOTAL** | **~70 min** |

---

## 🎯 Tips for Using These Prompts

1. **Copy each prompt EXACTLY as written** — they contain specific context the AI needs
2. **Follow the order** — each prompt assumes the previous ones are done
3. **After each prompt**, review the generated code before moving to the next
4. **If the AI gives complex code**, say: *"Simplify this. Make it fresher-friendly. Remove unnecessary complexity."*
5. **If the AI uses Pomelo instead of MySql.EntityFrameworkCore**, say: *"I am using MySql.EntityFrameworkCore (Oracle's official provider), NOT Pomelo. Use UseMySQL() method, not UseMySql()."*
6. **After all prompts are done**, run the test checklist from Prompt 12 to verify everything works

---

## 📝 What You'll Have After All 12 Prompts

```
HotelBookingAPI/
├── Controllers/          ← 6 controllers, 15 API endpoints
├── Services/
│   ├── Interfaces/       ← 7 interfaces
│   └── Implementations/  ← 7 service classes
├── Models/               ← 7 auto-scaffolded entity models
├── DTOs/                 ← 14 DTO classes
├── Data/                 ← AppDbContext (auto-generated)
├── Middleware/            ← GlobalExceptionMiddleware
├── Program.cs            ← All config (DI, JWT, CORS, Swagger)
└── appsettings.json      ← Connection string + JWT settings
```

**Database:** 7 tables + seed data (4 hotels, 9 rooms, 8 amenities, 3 coupons)  
**APIs:** 15 REST endpoints covering all MVP use cases  
**Security:** JWT auth + role-based authorization + rate limiting  
**Ready for:** Angular frontend integration
