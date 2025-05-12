# Backend Requirement Specifications: Core Features

## Feature 1: User Authentication & Authorization

### Description

Facilitates secure user identity management including sign-up, login, token refresh, and access control using JWT.

### API Endpoints

| Method | Endpoint              | Description                        |
| ------ | --------------------- | ---------------------------------- |
| POST   | /api/v1/auth/register | Register a new user                |
| POST   | /api/v1/auth/login    | Authenticate user and issue tokens |
| POST   | /api/v1/auth/logout   | Revoke refresh token               |
| POST   | /api/v1/auth/refresh  | Get new access token               |

### Input/Output Specifications

**Register**

```json
Request:
{
  "email": "user@example.com",
  "password": "StrongPass#23",
  "full_name": "Jane Doe"
}

Response:
{
  "message": "Registration successful. Please verify your email."
}
```

**Login**

```json
Request:
{
  "email": "user@example.com",
  "password": "StrongPass#23"
}

Response:
{
  "access_token": "<jwt_token>",
  "refresh_token": "<jwt_refresh>"
}
```

### Validation Rules

* Email must be RFC5322 compliant and unique.
* Password must be at least 8 characters, contain a digit, symbol, and letter.
* Rate limit: 5 failed attempts / 15 mins (to prevent brute-force).

### Performance Criteria

* Token generation within 200ms.
* Auth system uptime ≥ 99.95%.
* Refresh token validity: 7 days; Access token: 15 minutes.

---

## Feature 2:  Property Listing Management

### Description

Allows verified users (hosts) to CRUD property listings, manage availability, and attach media.

### API Endpoints

| Method | Endpoint                       | Description             |
| ------ | ------------------------------ | ----------------------- |
| POST   | /api/v1/properties             | Create new listing      |
| GET    | /api/v1/properties             | Get all listings        |
| GET    | /api/v1/properties/\:id        | View specific property  |
| PUT    | /api/v1/properties/\:id        | Update property details |
| DELETE | /api/v1/properties/\:id        | Remove a listing        |
| POST   | /api/v1/properties/\:id/images | Upload listing images   |

### Input/Output Specifications

**Create Listing**

```json
Request:
{
  "title": "Modern Bungalow in Nakuru",
  "price_per_night": 95.00,
  "location": "Nakuru",
  "amenities": ["WiFi", "Air Conditioning", "Kitchen"],
  "description": "Serene getaway spot with private garden.",
  "availability": {
    "from": "2025-06-01",
    "to": "2025-08-31"
  }
}

Response:
{
  "id": "prop-789",
  "message": "Property listed successfully"
}
```

### Validation Rules

* Title is required, max 120 characters.
* Price must be a positive float.
* Availability date range must be valid (start < end).
* Max 10 images per property (JPEG/PNG only, max 3MB each).

### Performance Criteria

* Average image upload response time: < 1.5s.
* API must handle 500 concurrent listings retrievals.
* Paginate results (default: 10 per page, max: 50).

---

## Feature 3: 🗖️ Booking & Availability System

### Description

Allows users to book available properties, check availability, cancel reservations, and manage their booking history.

### API Endpoints

| Method | Endpoint                   | Description                 |
| ------ | -------------------------- | --------------------------- |
| POST   | /api/v1/bookings           | Make a booking              |
| GET    | /api/v1/bookings           | Get user's bookings         |
| GET    | /api/v1/bookings/\:id      | View specific booking       |
| DELETE | /api/v1/bookings/\:id      | Cancel a booking            |
| POST   | /api/v1/availability/check | Check property availability |

### Input/Output Specifications

**Make Booking**

```json
Request:
{
  "property_id": "prop-789",
  "check_in": "2025-07-01",
  "check_out": "2025-07-04",
  "guests": 2
}

Response:
{
  "booking_id": "book-123",
  "status": "confirmed",
  "total": 285.00
}
```

**Check Availability**

```json
Request:
{
  "property_id": "prop-789",
  "start": "2025-07-01",
  "end": "2025-07-04"
}

Response:
{
  "available": true
}
```

### Validation Rules

* No overlapping bookings for same property.
* Booking duration: 1–30 days.
* Guests must not exceed property capacity.
* Cancellation: Only allowed before check-in date.

### Performance Criteria

* Availability check within 300ms.
* Bookings persisted in ACID-compliant transactions.
* Idempotent booking creation to prevent duplicates.

---

## Feature 4:  Messaging System

### Description

Enables real-time messaging between guests and hosts, with conversation threading and optional unread message counters.

### API Endpoints

| Method | Endpoint                            | Description             |
| ------ | ----------------------------------- | ----------------------- |
| POST   | /api/v1/messages                    | Send a message          |
| GET    | /api/v1/messages/conversations      | List user conversations |
| GET    | /api/v1/messages/conversations/\:id | Get specific thread     |

### Validation Rules

* Message content cannot be empty.
* Only booking participants can message each other.
* Rate-limit messages (e.g., max 10/min).

### Performance Criteria

* Messages stored in indexed threads.
* Conversation loading < 200ms.
* Support socket-based live updates (optional).

---

## Feature 5:  Ratings & Reviews

### Description

Users can rate and leave reviews after completed bookings. Hosts can respond to feedback.

### API Endpoints

| Method | Endpoint                   | Description              |
| ------ | -------------------------- | ------------------------ |
| POST   | /api/v1/reviews            | Leave a review           |
| GET    | /api/v1/reviews/\:id       | View reviews for listing |
| PUT    | /api/v1/reviews/\:id/reply | Host replies to review   |

### Validation Rules

* Rating: 1–5 stars.
* Reviews only allowed after booking checkout.
* Limit 1 review per booking.

### Performance Criteria

* Display average ratings in listing fetch.
* Review operations complete within 400ms.

---

## Feature 6:  Payment Integration

### Description

Enables secure payment handling via third-party gateways like Stripe. Supports bookings, refunds, and digital receipts.

### API Endpoints

| Method | Endpoint                | Description          |
| ------ | ----------------------- | -------------------- |
| POST   | /api/v1/payments        | Process a payment    |
| POST   | /api/v1/payments/refund | Refund a transaction |

### Input/Output Specifications

**Process Payment**

```json
Request:
{
  "booking_id": "book-123",
  "payment_token": "tok_visa_456"
}

Response:
{
  "payment_id": "pay-001",
  "status": "success"
}
```

### Validation Rules

* Payment token must be valid and from authorized gateway.
* Only unpaid, valid bookings can be paid.
* Refunds limited to policy and booking conditions.

### Performance Criteria

* Gateway response < 2 seconds.
* PCI-DSS compliance enforced.
* Logs stored for audit with 5-year retention.
