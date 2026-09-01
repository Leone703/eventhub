# EventHub — Events Management Test Scenarios

Generated: 2026-09-01
Scope: Event Discovery + Event Detail + Admin Event Management (create, edit, delete, filters, limits, validation, access control)

---

## Happy Path

### TC-E001: Browse events list with default pagination
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in
**Steps**:
1. Navigate to `/events`
2. Wait for the events grid to load
**Expected Results**: Events render as cards with title, category badge, venue/city, date, price, and seat status; page uses 12 cards per page by default
**Business Rule**: Events listing with `limit=12` from frontend
**Suggested Layer**: E2E

---

### TC-E002: Open event details from an event card
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; at least one event exists
**Steps**:
1. Navigate to `/events`
2. Click an event title or "Book Now"
**Expected Results**: User lands on `/events/:id` and sees event details, seat summary, and booking form panel
**Business Rule**: Flow 2 → Flow 3 navigation
**Suggested Layer**: E2E

---

### TC-E003: Filter events by category
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User is logged in; events exist across multiple categories
**Steps**:
1. Navigate to `/events`
2. Select a category (for example, Conference)
**Expected Results**: URL query updates with `category`; cards update to only matching category results
**Business Rule**: `GET /api/events?category=...`
**Suggested Layer**: E2E / API

---

### TC-E004: Filter events by city
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User is logged in; events exist across multiple cities
**Steps**:
1. Navigate to `/events`
2. Select a city (for example, Bangalore)
**Expected Results**: URL query updates with `city`; only events from that city are shown
**Business Rule**: `GET /api/events?city=...`
**Suggested Layer**: E2E / API

---

### TC-E005: Search events by title/description/venue
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User is logged in; searchable events exist
**Steps**:
1. Navigate to `/events`
2. Type a search term in the search input
3. Wait for debounce to apply
**Expected Results**: URL query updates with `search`; result set matches term across title, description, or venue
**Business Rule**: Search uses `contains` on title/description/venue
**Suggested Layer**: E2E / API

---

### TC-E006: Clear all applied event filters
**Category**: Happy Path
**Priority**: P2
**Preconditions**: User is on `/events` with search/category/city filter applied
**Steps**:
1. Click "Clear filters"
**Expected Results**: Query params are removed, page resets to unfiltered event list, and filter controls reset
**Business Rule**: Filter clear resets search params and page
**Suggested Layer**: E2E

---

### TC-E007: Create a custom event from admin page
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in
**Steps**:
1. Navigate to `/admin/events`
2. Fill all required event fields with valid data
3. Submit the form
**Expected Results**: Success toast appears (`Event created!`), event appears in the admin table and events list
**Business Rule**: `POST /api/events` creates a dynamic event with `isStatic=false`
**Suggested Layer**: E2E / API

---

### TC-E008: Edit an existing custom event
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User has at least one custom event
**Steps**:
1. Navigate to `/admin/events`
2. Click "Edit" on a custom event row
3. Update one or more fields and submit
**Expected Results**: Success toast appears (`Event updated!`) and updated values are visible in the admin table/event detail
**Business Rule**: `PUT /api/events/:id` allowed for owner-owned dynamic events only
**Suggested Layer**: E2E / API

---

### TC-E009: Delete a custom event from admin table
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User has at least one custom event
**Steps**:
1. Navigate to `/admin/events`
2. Click "Delete" on a custom event row
3. Confirm delete in dialog
**Expected Results**: Event is removed from table; success toast appears; event no longer resolves in detail view
**Business Rule**: `DELETE /api/events/:id` removes event (and associated bookings by cascade)
**Suggested Layer**: E2E / API

---

## Business Rules

### TC-E100: Max 6 custom events per user with FIFO replacement
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User already has 6 custom events
**Steps**:
1. Note oldest custom event ID
2. Create a 7th custom event
3. Fetch admin events list
**Expected Results**: Total custom events remains 6; oldest custom event is removed; new event exists
**Business Rule**: FIFO pruning on `MAX_USER_DYNAMIC_EVENTS = 6`
**Suggested Layer**: API

---

### TC-E101: Static events appear for all users and are not counted as custom events
**Category**: Business Rule
**Priority**: P1
**Preconditions**: Static seeded events exist
**Steps**:
1. Login as User A and fetch events
2. Login as User B and fetch events
3. Compare static event presence
**Expected Results**: Both users can see static events; static events are not affected by custom-event FIFO limit
**Business Rule**: `isStatic=true` shared events + custom-event limit applies to dynamic events only
**Suggested Layer**: API / E2E

---

### TC-E102: Per-user computed seat availability for dynamic events
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User has a dynamic event and creates bookings for it
**Steps**:
1. Capture event `totalSeats`
2. Create booking(s) as same user for that event
3. Fetch event list/detail again
**Expected Results**: `availableSeats` shown to that user is `stored availableSeats - user's booked quantity`, never below 0
**Business Rule**: `withPersonalSeats` adjustment for dynamic events
**Suggested Layer**: API

---

### TC-E103: Low-seat and sold-out visual states on cards and detail page
**Category**: Business Rule
**Priority**: P1
**Preconditions**: Events exist with seat states (>10, <=10, 0)
**Steps**:
1. Open events list and one event detail page for each seat state
**Expected Results**: >10 shows available seat count; <=10 shows low-seat warning; 0 shows sold-out state and disabled booking CTA
**Business Rule**: Seat indicators and sold-out guard in EventCard and detail page
**Suggested Layer**: E2E / Component

---

### TC-E104: Sandbox warning banner threshold on events page
**Category**: Business Rule
**Priority**: P2
**Preconditions**: User has enough visible events to cross threshold
**Steps**:
1. Navigate to `/events`
2. Observe banner presence when event count > 5
**Expected Results**: Banner appears with limit guidance; banner is hidden when event count is low
**Business Rule**: Conditional sandbox warning in events page
**Suggested Layer**: E2E / Component

---

## Security & Access Control

### TC-E200: Unauthenticated user cannot access events APIs
**Category**: Security
**Priority**: P0
**Preconditions**: No JWT token
**Steps**:
1. Call `GET /api/events`
2. Call `GET /api/events/:id`
3. Call `POST /api/events`
**Expected Results**: Each request returns 401 Unauthorized
**Business Rule**: `authMiddleware` required for all event routes
**Suggested Layer**: API

---

### TC-E201: User cannot update another user's dynamic event
**Category**: Security
**Priority**: P0
**Preconditions**: User A owns dynamic event; User B has valid JWT
**Steps**:
1. As User B, call `PUT /api/events/:userAEventId`
**Expected Results**: Request is rejected (404 not found in scoped lookup or 403 ownership rule)
**Business Rule**: Scoped event lookup + ownership enforcement
**Suggested Layer**: API

---

### TC-E202: User cannot delete another user's dynamic event
**Category**: Security
**Priority**: P0
**Preconditions**: User A owns dynamic event; User B has valid JWT
**Steps**:
1. As User B, call `DELETE /api/events/:userAEventId`
**Expected Results**: Request is rejected (404 not found in scoped lookup or 403 ownership rule)
**Business Rule**: Scoped event lookup + ownership enforcement
**Suggested Layer**: API

---

### TC-E203: Static events are read-only in admin actions
**Category**: Security
**Priority**: P0
**Preconditions**: Static seeded event exists
**Steps**:
1. Attempt `PUT /api/events/:staticEventId`
2. Attempt `DELETE /api/events/:staticEventId`
**Expected Results**: Both operations return 403 with static-event protection message
**Business Rule**: Static events cannot be modified/deleted
**Suggested Layer**: API / E2E

---

## Validation & Error Handling

### TC-E300: Creating event with missing required fields fails validation
**Category**: Validation
**Priority**: P0
**Preconditions**: User is authenticated
**Steps**:
1. Submit `POST /api/events` missing one or more required fields
**Expected Results**: 400 response with `Validation failed` and field-level details
**Business Rule**: `validateCreateEvent` required fields
**Suggested Layer**: API

---

### TC-E301: Event date must be future date
**Category**: Validation
**Priority**: P0
**Preconditions**: User is authenticated
**Steps**:
1. Submit event payload with current/past `eventDate`
**Expected Results**: 400 validation error (`Event date must be in the future`)
**Business Rule**: Future date custom validator
**Suggested Layer**: API / Component

---

### TC-E302: Event date must be valid ISO 8601 format
**Category**: Validation
**Priority**: P1
**Preconditions**: User is authenticated
**Steps**:
1. Submit invalid `eventDate` format
**Expected Results**: 400 validation error (`Event date must be a valid ISO 8601 date`)
**Business Rule**: `isISO8601` validation
**Suggested Layer**: API

---

### TC-E303: Price cannot be negative
**Category**: Validation
**Priority**: P0
**Preconditions**: User is authenticated
**Steps**:
1. Submit `price < 0`
**Expected Results**: 400 validation error (`Price must be a non-negative number`)
**Business Rule**: price `isFloat({ min: 0 })`
**Suggested Layer**: API / Component

---

### TC-E304: Total seats must be positive integer
**Category**: Validation
**Priority**: P0
**Preconditions**: User is authenticated
**Steps**:
1. Submit `totalSeats = 0` or negative
2. Submit non-integer `totalSeats`
**Expected Results**: 400 validation error (`Total seats must be a positive integer`)
**Business Rule**: totalSeats `isInt({ min: 1 })`
**Suggested Layer**: API / Component

---

### TC-E305: Invalid image URL fails validation
**Category**: Validation
**Priority**: P2
**Preconditions**: User is authenticated
**Steps**:
1. Submit non-URL value for `imageUrl`
**Expected Results**: 400 validation error (`Image URL must be a valid URL`)
**Business Rule**: optional `imageUrl` must pass `isURL`
**Suggested Layer**: API

---

### TC-E306: Event detail for inaccessible/nonexistent id returns not found state
**Category**: Error Handling
**Priority**: P1
**Preconditions**: User is logged in
**Steps**:
1. Navigate to `/events/:id` with invalid/nonexistent/inaccessible event ID
**Expected Results**: UI shows "Event not found" empty state with "Browse Events" action
**Business Rule**: `getEventById` not-found behavior + frontend fallback state
**Suggested Layer**: E2E

---

### TC-E307: Events list API/server error shows retry state
**Category**: Error Handling
**Priority**: P2
**Preconditions**: Simulated API failure
**Steps**:
1. Load `/events` while events API fails
2. Click "Retry"
**Expected Results**: Error empty state appears first; retry attempts refetch
**Business Rule**: events page error handling with retry action
**Suggested Layer**: Component / E2E

---

## Pagination & Query Behavior

### TC-E400: Events list returns stable pagination metadata
**Category**: Pagination
**Priority**: P1
**Preconditions**: More events exist than one page limit
**Steps**:
1. Call `GET /api/events?page=1&limit=10`
2. Call `GET /api/events?page=2&limit=10`
**Expected Results**: Response includes `{ total, page, limit, totalPages }`; page slices are consistent and non-overlapping
**Business Rule**: Pagination contract from event service/repository
**Suggested Layer**: API

---

### TC-E401: Changing filters resets page to first page
**Category**: Pagination
**Priority**: P2
**Preconditions**: User is on `/events?page=2`
**Steps**:
1. Change category, city, or search filter
**Expected Results**: URL drops prior `page` and results load from page 1 of new filter set
**Business Rule**: filter push deletes `page` query param
**Suggested Layer**: E2E / Component

---

### TC-E402: Events list ordering prioritizes static events then newest
**Category**: Query Behavior
**Priority**: P2
**Preconditions**: Mix of static and dynamic events available
**Steps**:
1. Fetch `/api/events`
2. Inspect response order
**Expected Results**: Static events appear first; within groups ordering is newest-first by `createdAt`
**Business Rule**: repository `orderBy: [{ isStatic: 'desc' }, { createdAt: 'desc' }]`
**Suggested Layer**: API

---

## Quick Coverage Summary

- Total scenarios: 28
- Happy Path: 9
- Business Rules: 5
- Security & Access Control: 4
- Validation & Error Handling: 8
- Pagination & Query Behavior: 3
