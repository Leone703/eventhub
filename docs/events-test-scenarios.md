---

## Create Events

### TC-010: Create a new event via Admin UI with all valid fields
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in and on `/admin/events`
**Steps**:
1. Fill `#event-title-input` with "Node.js Workshop"
2. Fill textarea (description) with a short description
3. Select Category "Workshop"
4. Select City "Bangalore"
5. Fill Venue with "Tech Park Hall A"
6. Fill Event Date & Time with a future date/time
7. Fill Price ($) with "199"
8. Fill Total Seats with "50"
9. Click `#add-event-btn`
**Expected Results**: A success toast "Event created!" appears; the new event card appears in the user-created events list below the form
**Business Rule**: Flow 5 — Admin Manage Events; POST /api/events
**Suggested Layer**: E2E

---

### TC-011: Newly created event appears on the public Events page
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User is logged in; a new event "Node.js Workshop" was just created via Admin UI
**Steps**:
1. Navigate to `/events`
2. Search for "Node.js Workshop"
**Expected Results**: The newly created event card is visible with correct title, city, venue, and price
**Business Rule**: GET /api/events returns user-created events; per-user sandbox isolation
**Suggested Layer**: E2E

---

### TC-012: Created event can be booked immediately after creation
**Category**: Happy Path
**Priority**: P0
**Preconditions**: User has created a new event with 50 seats; user is on `/events`
**Steps**:
1. Find the created event card and click "Book Now"
2. Fill the booking form (name, email, phone, quantity = 1)
3. Click "Confirm Booking"
**Expected Results**: Booking confirmation is shown with a valid booking reference; reference first character matches the event title's first character (uppercased)
**Business Rule**: Booking reference format; per-user seat computation for dynamic events
**Suggested Layer**: E2E

---

### TC-013: Admin form resets after successful event creation
**Category**: Happy Path
**Priority**: P1
**Preconditions**: User is on `/admin/events`; one event has just been created successfully
**Steps**:
1. Observe the Admin Event Form after the success toast disappears
**Expected Results**: All form fields are cleared/reset to their default/empty state
**Business Rule**: Flow 5 — "Event created!" toast then list updates
**Suggested Layer**: Component / E2E

---

### TC-100: FIFO pruning — 7th event creation deletes the oldest user event
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User already has exactly 6 user-created events (no more, no less)
**Steps**:
1. Note the title of the oldest (first-created) event, e.g. "Event 1"
2. Navigate to `/admin/events`
3. Create a 7th event with a distinct title, e.g. "Event 7"
4. Click `#add-event-btn`
**Expected Results**: "Event created!" toast appears; "Event 1" (oldest) is no longer visible in the list; the new "Event 7" appears; total remains 6
**Business Rule**: Max 6 user-created events; FIFO replacement on overflow
**Suggested Layer**: E2E

---

### TC-111: Static events are not counted toward the 6-event limit
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User has 6 user-created events; static seeded events exist on `/events`
**Steps**:
1. Confirm the 6-event count via `/admin/events`
2. Verify static events are still listed on `/events` alongside the user's 6
**Expected Results**: Static events remain unaffected and visible; user can still create a 7th event (triggering FIFO on user events only, not statics)
**Business Rule**: "Static events are not counted toward this limit"
**Suggested Layer**: E2E / API

---

### TC-112: Static events cannot be edited or deleted from Admin UI
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User is on `/admin/events`; at least one static event is rendered (or known by ID)
**Steps**:
1. Attempt to send PUT /api/events/:staticEventId with an auth token
2. Attempt to send DELETE /api/events/:staticEventId with an auth token
**Expected Results**: Both requests return 403 with message "Cannot modify static events"
**Business Rule**: Static events are immutable; 403 for edit/delete
**Suggested Layer**: API

---

### TC-113: Event date must be in the future
**Category**: Business Rule
**Priority**: P0
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Fill all required fields with valid values
2. Set Event Date & Time to a past date (e.g. yesterday)
3. Click `#add-event-btn`
**Expected Results**: Request returns 400 with message "Event date must be in the future"; event is NOT created; error is surfaced in the UI
**Business Rule**: eventDate must be future; API error table
**Suggested Layer**: E2E / API

---

### TC-114: Total seats must be at least 1
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Fill all required fields; set Total Seats to "0"
2. Click `#add-event-btn`
**Expected Results**: Validation error returned (400); event is NOT created
**Business Rule**: totalSeats >= 1 (schema constraint)
**Suggested Layer**: API / E2E

---

### TC-115: Price must be zero or positive
**Category**: Business Rule
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Fill all required fields; set Price to "-1"
2. Click `#add-event-btn`
**Expected Results**: Validation error returned (400); event is NOT created
**Business Rule**: price >= 0 (schema constraint)
**Suggested Layer**: API / E2E

---

### TC-200: Unauthenticated user cannot create an event via API
**Category**: Security
**Priority**: P0
**Preconditions**: No auth token (or expired token)
**Steps**:
1. Send POST /api/events with a valid body but without an Authorization header
**Expected Results**: 401 "Unauthorized" response; no event is created
**Business Rule**: All event endpoints require auth (JWT middleware)
**Suggested Layer**: API

---

### TC-201: User cannot edit another user's event via API
**Category**: Security
**Priority**: P0
**Preconditions**: User A has created event with id X; User B is logged in with a different token
**Steps**:
1. User B sends PUT /api/events/X with User B's token and a modified title
**Expected Results**: 403 Forbidden response; event remains unchanged
**Business Rule**: Cross-user access denied; sandbox isolation
**Suggested Layer**: API

---

### TC-202: User cannot delete another user's event via API
**Category**: Security
**Priority**: P0
**Preconditions**: User A has created event with id X; User B is logged in
**Steps**:
1. User B sends DELETE /api/events/X with User B's token
**Expected Results**: 403 Forbidden; event still exists for User A
**Business Rule**: Cross-user access denied
**Suggested Layer**: API

---

### TC-203: User B cannot see User A's dynamic events on the Events page
**Category**: Security
**Priority**: P0
**Preconditions**: User A has created at least one event; User B is logged in
**Steps**:
1. User B navigates to `/events`
2. Searches/browses for User A's event title
**Expected Results**: User A's event is not visible in User B's event list (sandbox isolation)
**Business Rule**: "Each user only sees their own dynamic events"
**Suggested Layer**: E2E

---

### TC-300: Create event with missing required title field
**Category**: Negative
**Priority**: P0
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Leave the Title field empty; fill all other fields correctly
2. Click `#add-event-btn`
**Expected Results**: 400 validation error; event not created; form may highlight the missing field
**Business Rule**: title is required
**Suggested Layer**: E2E / API

---

### TC-301: Create event with missing required venue field
**Category**: Negative
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Leave the Venue field empty; fill all other required fields
2. Click `#add-event-btn`
**Expected Results**: 400 validation error; event not created
**Business Rule**: venue is required
**Suggested Layer**: API

---

### TC-302: Create event with missing required city field
**Category**: Negative
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Leave the City dropdown at default/unselected; fill all other fields
2. Click `#add-event-btn`
**Expected Results**: 400 validation error; event not created
**Business Rule**: city is required
**Suggested Layer**: API

---

### TC-303: Create event with a non-numeric price value
**Category**: Negative
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Fill Price with "abc"
2. Fill all other fields correctly
3. Click `#add-event-btn`
**Expected Results**: Validation error; event not created
**Business Rule**: price is Decimal type; must be numeric
**Suggested Layer**: API / E2E

---

### TC-304: Create event with a non-numeric totalSeats value
**Category**: Negative
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Fill Total Seats with "ten"
2. Fill all other fields correctly
3. Click `#add-event-btn`
**Expected Results**: Validation error; event not created
**Business Rule**: totalSeats is Int; must be numeric
**Suggested Layer**: API / E2E

---

### TC-305: Create event with an invalid category not in the allowed list
**Category**: Negative
**Priority**: P2
**Preconditions**: Direct API call (bypass UI dropdown constraint)
**Steps**:
1. POST /api/events with category = "InvalidCategory" and all other fields valid
**Expected Results**: 400 validation error; event not created
**Business Rule**: category must be one of: Conference/Concert/Sports/Workshop/Festival
**Suggested Layer**: API

---

### TC-306: Create event with an invalid city not in the allowed list
**Category**: Negative
**Priority**: P2
**Preconditions**: Direct API call (bypass UI dropdown constraint)
**Steps**:
1. POST /api/events with city = "London" and all other fields valid
**Expected Results**: 400 validation error; event not created
**Business Rule**: city must be one of: Bangalore/Mumbai/Hyderabad/Delhi/Chennai
**Suggested Layer**: API

---

### TC-400: Create event at exactly the 6-event limit (the 6th event)
**Category**: Edge Case
**Priority**: P1
**Preconditions**: User has exactly 5 user-created events
**Steps**:
1. Navigate to `/admin/events`
2. Create the 6th event with valid data
**Expected Results**: Event is created successfully; all 6 events visible; no FIFO pruning occurs
**Business Rule**: Max 6 events; pruning only triggers on the 7th (overflow)
**Suggested Layer**: E2E / API

---

### TC-412: Event with price = 0 (free event) can be created and booked
**Category**: Edge Case
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Create an event with Price = "0" and all other fields valid
2. Navigate to `/events`, find the free event
3. Book 1 ticket
**Expected Results**: Event created; booking total price = $0.00; booking reference generated correctly
**Business Rule**: price >= 0; totalPrice = price × quantity
**Suggested Layer**: E2E

---

### TC-413: Event with exactly 1 total seat
**Category**: Edge Case
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Create an event with Total Seats = "1" and all other fields valid
2. Navigate to `/events`, find the event
3. Book the 1 available ticket
**Expected Results**: Event created; booking succeeds with quantity = 1; event now shows 0 available seats; attempting a second booking is rejected with "Insufficient seats available"
**Business Rule**: totalSeats >= 1; availableSeats computed for dynamic events
**Suggested Layer**: E2E

---

### TC-414: Event title with a lowercase first character — booking ref uses uppercase
**Category**: Edge Case
**Priority**: P2
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Create an event whose title starts with a lowercase letter, e.g. "awesome concert"
2. Book a ticket for this event
**Expected Results**: Booking reference starts with "A" (uppercase of "a"); format is `A-XXXXXX`
**Business Rule**: "Booking reference first character MUST match the event title's first character (uppercase)"
**Suggested Layer**: E2E / API

---

### TC-415: Event title starting with a digit — booking ref prefix is that digit
**Category**: Edge Case
**Priority**: P2
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Create an event with title "3D Art Exhibition"
2. Book 1 ticket
**Expected Results**: Booking reference starts with "3" (e.g. `3-XXXXXX`)
**Business Rule**: Booking ref first char = uppercase(title[0]); digit is already uppercase
**Suggested Layer**: E2E / API

---

### TC-500: Admin event form shows validation errors inline before submission
**Category**: UI State
**Priority**: P1
**Preconditions**: User is on `/admin/events`
**Steps**:
1. Click `#add-event-btn` without filling any fields
**Expected Results**: Form validation messages are shown inline next to the required fields (Title, Venue, City, Date, Seats); submit is blocked or returns an error
**Business Rule**: title and venue required; eventDate future; totalSeats >= 1
**Suggested Layer**: Component / E2E

---

### TC-514: Sandbox warning banner appears when user has 6 events
**Category**: UI State
**Priority**: P1
**Preconditions**: User has exactly 6 user-created events; user is on `/events`
**Steps**:
1. Navigate to `/events`
2. Observe the page
**Expected Results**: Sandbox warning banner is visible with text matching `/sandbox holds up to/i`
**Business Rule**: "Events page: Banner appears when user has close to or more than 6 events"
**Suggested Layer**: E2E

---

### TC-515: Sandbox warning banner is hidden when user has fewer than 5 events
**Category**: UI State
**Priority**: P2
**Preconditions**: User has 4 or fewer user-created events; user is on `/events`
**Steps**:
1. Navigate to `/events`
2. Observe for the sandbox banner
**Expected Results**: Sandbox warning banner is NOT visible
**Business Rule**: "Banners are hidden when counts are low (e.g., fewer than 5 events)"
**Suggested Layer**: E2E / Component

---

### TC-516: Admin events list shows loading state while fetching
**Category**: UI State
**Priority**: P2
**Preconditions**: User navigates to `/admin/events` on a slow connection (or simulated)
**Steps**:
1. Navigate to `/admin/events`
2. Observe the events list section before data loads
**Expected Results**: A loading skeleton or spinner is displayed while the events list is being fetched
**Business Rule**: Flow 5 — Admin Manage Events; React Query loading state
**Suggested Layer**: Component

---

### TC-517: Admin events list shows empty state when user has no events
**Category**: UI State
**Priority**: P1
**Preconditions**: User has 0 user-created events; user is on `/admin/events`
**Steps**:
1. Navigate to `/admin/events`
2. Observe the events list section
**Expected Results**: An empty-state message (e.g. "No events yet" or similar) is shown in place of event cards
**Business Rule**: Per-user sandbox isolation; user event list is empty on a fresh account
**Suggested Layer**: E2E / Component
