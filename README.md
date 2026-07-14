# Tailor Fullstack Challenge

## Goal

Build a small restaurant application for discovering restaurants and managing table reservations.

This is a two-day, time-boxed exercise. The goal is not to build a complete restaurant platform; it is to show clear domain modelling, sensible API contracts, server-side validation, pragmatic Java/Spring structure, and a usable React interface.

## Required Technology

- Java 21 or later.
- Spring Boot 3.
- Maven or Gradle.
- JSON REST API.
- React with TypeScript for the frontend. Vite is a lightweight default; Next.js is also acceptable.

Persistence is optional. For this exercise, an in-memory implementation loaded from the provided `restaurants.json` file is recommended. Data may reset when the application restarts.

Keep business logic separate from HTTP controllers. If using in-memory or JSON storage, isolate it behind a repository or service rather than accessing collections directly from controllers.

## Required Scope

### 1. Authentication

Implement a minimal login endpoint for one seeded demo user.

```txt
POST /auth/login
```

The authentication mechanism may use a documented simple token. JWT is not required. Reservation endpoints for the current user must require authentication.

Include the demo credentials in the README.

### 2. Restaurants

Use the supplied `restaurants.json` payload, an adapted version of it, or a small equivalent seed dataset.

Implement read-only restaurant endpoints:

```txt
GET /restaurants
GET /restaurants/{restaurantId}
```

Each restaurant response should expose enough information for a client to display a list and detail view, including its name, description, address, image, cuisine type, and reservation settings.

Restaurant administration is not part of this exercise. Do not implement restaurant CRUD.

### 3. Availability

Users must be able to request the availability for a restaurant, date, and party size.

```txt
GET /restaurants/{restaurantId}/availability?date=YYYY-MM-DD&partySize=4
```

Generate slots from each restaurant's `reservationSettings`:

- The default service windows are lunch from `13:00` to `15:00` and dinner from `20:00` to `23:00`.
- The default slot interval is 30 minutes.
- Window starts are inclusive and ends are exclusive. For example, `13:00` to `15:00` generates `13:00`, `13:30`, `14:00`, and `14:30`.
- Each slot has the restaurant's configured capacity.
- Future dates without bookings must still return generated slots.

The response should make it easy for a client to identify whether a slot can accommodate the requested party size:

```json
{
  "restaurantId": 1,
  "date": "2026-07-10",
  "slots": [
    {
      "time": "13:00",
      "capacity": 8,
      "reservedSeats": 8,
      "availableSeats": 0,
      "available": false
    },
    {
      "time": "13:30",
      "capacity": 8,
      "reservedSeats": 3,
      "availableSeats": 5,
      "available": true
    }
  ]
}
```

Availability is calculated from:

```txt
restaurant.reservationSettings.bookedSlots
+ confirmed reservations created by users
- cancelled reservations
```

New reservations may be stored separately from the original seed data.

### 4. Reservations

Authenticated users must be able to create, list, and cancel their own reservations.

```txt
POST  /reservations
GET   /me/reservations
PATCH /reservations/{reservationId}/cancel
```

Example create request:

```json
{
  "restaurantId": 1,
  "date": "2026-07-10",
  "time": "13:30",
  "partySize": 4
}
```

Example response:

```json
{
  "id": "res_123",
  "restaurantId": 1,
  "userId": 1,
  "date": "2026-07-10",
  "time": "13:30",
  "partySize": 4,
  "status": "confirmed"
}
```

Use these statuses:

```java
enum ReservationStatus {
  CONFIRMED,
  CANCELLED
}
```

### Business Rules

Implement and validate these rules on the server:

- A reservation cannot be created in the past.
- Party size must be greater than zero.
- The requested time must be a generated slot for that restaurant.
- Party size cannot exceed the available seats in that restaurant, date, and time slot.
- Creating a confirmed reservation reduces the available seats immediately.
- Cancelling a reservation releases its seats.
- A cancelled reservation cannot be cancelled again.
- A user can only list and cancel their own reservations.
- The availability response is not the source of truth; recalculate availability before accepting a reservation.

Return appropriate HTTP status codes and a clear error response for invalid input, missing resources, unauthenticated requests, and rule violations.

## Automated Tests

Write a small, focused test suite for the reservation domain. At minimum, cover:

- Generating availability for a future date with no existing bookings.
- Rejecting a reservation that exceeds the available capacity.
- Reducing availability when a reservation is confirmed.
- Releasing availability when a reservation is cancelled.

Integration tests and API documentation are welcome but not required.

## Frontend

Implement a frontend with React and TypeScript that consumes the Spring Boot API. Reservation availability and validation must remain server-side; do not reproduce the business rules in the browser.

The required screens and flows are:

- Demo login.
- Restaurant list and restaurant detail.
- Date and party-size selection followed by availability lookup.
- Slot selection and reservation creation.
- Current user's reservations with cancellation.
- Basic loading, empty, and error states.

Keep the frontend intentionally small. The interface is free-form: prioritise a simple, responsive, and usable experience over visual polish. Restaurant CRUD, comments, favourites, and advanced state-management libraries are not expected.

## Explicitly Out of Scope

Do not spend time implementing:

- Restaurant CRUD.
- Restaurant comments.
- Favourite restaurants.
- User registration, password recovery, roles, or a production-grade authentication system.
- A database, migrations, Docker, Swagger/OpenAPI, or end-to-end tests.

You may add these only after the required reservation flow is complete.

## Deliverables

- Source code in a public GitHub repository.
- A README with setup instructions, run commands, test commands, sample credentials, and API decisions.
- A deployed version of the application, with public frontend and API URLs documented in the README. A free-tier deployment is sufficient; for example, Vercel for the React frontend and Railway, Render, Fly.io, or a similar Java-compatible platform for the Spring Boot API. Any equivalent provider is acceptable.
- A Postman collection containing the login, restaurant, availability, and reservation requests. Include it in the repository and document how to configure its base URL and authentication token.
- A short explanation of the domain and technical trade-offs.
- If AI tools were used, a concise note describing which tools helped, what was reviewed, and any limitations identified.

## Evaluation

The mandatory assessment focuses on:

- Correct handling of availability and reservation capacity.
- Server-side validation and useful error handling.
- Clear Java/Spring structure and separation of responsibilities.
- Readable API contracts and code.
- Relevant automated tests.
- Frontend usability, API integration, and handling of loading, empty, and error states.
- README quality and ability to explain implementation decisions.
