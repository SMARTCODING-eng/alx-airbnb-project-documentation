# Backend Feature Requirement Specifications


## 1. Feature: User Authentication

**Description**: This feature manages user registration, login, token generation, and password reset functionalities, ensuring secure access to the platform.

API Endpoints:

1.1. **User Registration**

Endpoint URL: /api/auth/register

HTTP Method: POST

Request Body (Input):

firstName: User's first name.

lastName: User's last name.

email: User's email address (for login and communication).

password: User's chosen password.

Response Body (Output - Success):

A success message confirming registration and instructions to verify email.

The newly generated unique user ID.

The registered email address.

Response Body (Output - Error):

An error message indicating the issue (e.g., "Email already registered," "Invalid input").

Specific details about which fields failed validation.

Validation Rules:

firstName: Must be present, a string, with a minimum length of 2 characters and a maximum of 50.

lastName: Must be present, a string, with a minimum length of 2 characters and a maximum of 50.

email: Must be present, a valid email format, and globally unique within the system (not already used by another user).

password: Must be present, a string, with a minimum length of 8 characters and a maximum of 100. It must include at least one uppercase letter, one lowercase letter, one number, and one special character (e.g., !@#$%^&*).



1.2. **User Login**
Endpoint URL: /api/auth/login

HTTP Method: POST

Request Body (Input):

email: User's registered email address.

password: User's password.

Response Body (Output - Success):

A success message indicating successful login.

An accessToken (JWT) for subsequent authenticated requests.

A refreshToken (JWT) for renewing the access token.

expiresIn: The duration (in seconds) until the accessToken expires.

Response Body (Output - Error):

An error message indicating invalid credentials.

Validation Rules:

email: Must be present and a valid email format.

password: Must be present.

1.3. **Email Verification**

Endpoint URL: /api/auth/verify-email

HTTP Method: GET

Query Parameters (Input):

token: The unique verification token sent to the user's email.

Response Body (Output - Success):

A message confirming that the email has been successfully verified, instructing the user that they can now log in.

Response Body (Output - Error):

An error message indicating an invalid or expired verification token.

Validation Rules:

token: Must be present and correspond to a valid, unexpired token stored in the database.

1.4. **Password Reset Request**

Endpoint URL: /api/auth/forgot-password

HTTP Method: POST

Request Body (Input):

email: The user's registered email address for which the password reset is requested.

Response Body (Output - Success):

A message indicating that if the email is registered, a password reset link has been sent. (This generic message is for security reasons, to avoid confirming account existence).

Validation Rules:

email: Must be present and a valid email format.



Performance Criteria:

Latency: All authentication endpoints (register, login, verify-email) must respond within 200 milliseconds for 95% of requests under normal load.

Throughput: The system should be capable of handling at least 50 authentication requests per second (RPS) for login and registration without any degradation in response time.

Error Rate: The cumulative error rate for all authentication endpoints must remain below 0.5%.

## Feature: Property Management
Description: This feature allows property owners and administrators to list, update, view, and delete property listings on the platform. It encompasses functionalities for managing property details, location, amenities, and associated media.

API Endpoints:

2.1. **List All Properties (Public/Filtered)**
Endpoint URL: /api/properties

HTTP Method: GET

Query Parameters (Optional Input for filtering/pagination):

page: Page number (integer, default 1).

limit: Number of items per page (integer, default 10, maximum 100).

search: Free text search term (string, e.g., property name, description, city).

type: Type of property (enum, e.g., 'apartment', 'house', 'villa', 'condo').

minPrice: Minimum price (number).

maxPrice: Maximum price (number).

minArea: Minimum area (number, in sqft/sqm).

maxArea: Maximum area (number, in sqft/sqm).

city: City name (string).

state: State name (string).

country: Country name (string).

sortBy: Sorting criteria (enum, e.g., 'price_asc' for ascending price, 'price_desc' for descending price, 'createdAt_desc' for newest first).

Response Body (Output - Success):

A list of property objects, each including: unique ID, title, description, type, address details (street, city, state, zip code, country, latitude, longitude), price, currency, number of bedrooms, bathrooms, area, area unit, list of amenities, list of media URLs, owner ID, availability status, creation timestamp, and last update timestamp.

Pagination metadata: total number of items, current page number, total number of pages.

Validation Rules:

All query parameters should be validated for correct data type and range. Invalid or malformed parameters should either be ignored or result in a 400 Bad Request error.

2.2. **Get Property by ID**

Endpoint URL: /api/properties/{id}

HTTP Method: GET

Path Parameters (Input):

id: The unique identifier (UUID) of the property.

Response Body (Output - Success):

A single property object with all its details, as described in the List All Properties response.

Response Body (Output - Error):

An error message indicating that the property was not found.

Validation Rules:

id: Must be present and a valid UUID format.

2.3.**Create New Property (Requires Authentication - Owner/Admin Role)**

Endpoint URL: /api/properties

HTTP Method: POST

Authentication: Requires a valid JWT Bearer Token. The authenticated user must possess an 'owner' or 'admin' role.

Request Body (Input):

title: The title of the property listing.

description: A detailed description of the property.

type: The type of property (e.g., apartment, house).

address: An object containing street, city, state, zip code, country, and optional latitude/longitude.

price: The rental or sale price of the property.

currency: The currency code for the price (e.g., "USD", "NGN").

area: The total area of the property.

areaUnit: The unit of area (e.g., 'sqft', 'sqm').

amenities: An optional list of amenities available (e.g., "WiFi", "Pool").

mediaUrls: An optional list of URLs to property images or videos.

Response Body (Output - Success):

The newly created property object, including its generated unique ID, the ID of the owner (from the authenticated user), and creation/update timestamps.

Validation Rules:

title: Must be present, a string, min 5 chars, max 100 chars.

description: Must be present, a string, min 20 chars, max 1000 chars.

type: Must be present and one of the allowed enum values.

address.street, city, state, country: Must be present, strings, min 2 chars, max 100 chars.

address.zipCode: Optional, string, valid format.

address.latitude, longitude: Optional, valid numeric coordinates.

price: Must be present and a positive number.

currency: Must be present, a 3-letter ISO currency code.

bedrooms, bathrooms: Must be present, non-negative integers.

area: Must be present and a positive number.

areaUnit: Must be present and one of 'sqft' or 'sqm'.

amenities: Optional, an array of strings, with a maximum of 20 amenities, each up to 50 characters.

mediaUrls: Optional, an array of valid URL strings, with a maximum of 10 URLs.

2.4. **Update Property (Requires Authentication - Owner/Admin Role, must own property)**

Endpoint URL: /api/properties/{id}

HTTP Method: PATCH (for partial updates)

Authentication: Requires a valid JWT Bearer Token. The authenticated user must own the property or be an administrator.

Path Parameters (Input):

id: The unique identifier (UUID) of the property to update.

Request Body (Input - Partial Update):

Any of the fields from the Create Property request body can be included, and all are optional for a PATCH request. Only provided fields will be updated.

Response Body (Output - Success):

The updated property object.

Response Body (Output - Error):

An error message indicating that the user is not authorized to update this property, or the property was not found.

Validation Rules:

id: Must be present and a valid UUID.

Any fields provided in the request body must adhere to the same validation rules as specified for Create New Property.

2.5. **Delete Property (Requires Authentication - Owner/Admin Role, must own property)**

Endpoint URL: /api/properties/{id}

HTTP Method: DELETE

Authentication: Requires a valid JWT Bearer Token. The authenticated user must own the property or be an administrator.

Path Parameters (Input):

id: The unique identifier (UUID) of the property to delete.

Response Body (Output - Success):

No content in the response body (HTTP 204 No Content).

Response Body (Output - Error):

An error message indicating that the user is not authorized to delete this property, or the property was not found.

Validation Rules:

id: Must be present and a valid UUID.

Performance Criteria:

Latency:

GET /api/properties: Should respond within 300 milliseconds for 95% of requests, even when applying complex filters.

GET /api/properties/{id}: Should respond within 100 milliseconds for 95% of requests.

POST/PATCH/DELETE /api/properties/{id}: Should respond within 250 milliseconds for 95% of requests.

Throughput:

GET /api/properties: The system should be able to handle at least 100 requests per second (RPS).

POST/PATCH/DELETE /api/properties/{id}: The system should be able to handle at least 30 requests per second (RPS).

Data Consistency: Property data should be immediately consistent across all read operations following any write operation (creation, update, deletion).

3. ## Feature: Booking System
Description: This feature enables users to book properties for specific date ranges, manage their bookings, and allows property owners to view bookings for their own properties. It incorporates availability checks and handles booking status management.

API Endpoints:

3.1. **Create Booking (Requires Authentication - User Role)**

Endpoint URL: /api/bookings

HTTP Method: POST

Authentication: Requires a valid JWT Bearer Token. The authenticated user must possess a 'user' role.

Request Body (Input):

propertyId: The unique identifier (UUID) of the property to be booked.

checkInDate: The desired check-in date for the booking (YYYY-MM-DD format).

checkOutDate: The desired check-out date for the booking (YYYY-MM-DD format).

numberOfGuests: The number of guests for this booking.

Response Body (Output - Success):

A message confirming successful booking creation.

The newly generated booking ID.

Details of the booking: property ID, user ID (from authenticated user), check-in/out dates, number of guests, calculated total price, initial booking status ('pending'), and creation timestamp.

Response Body (Output - Error):

An error message indicating issues such as "Property not found," "Property not available for the selected dates," or "Invalid date range."

Validation Rules:

propertyId: Must be present, a valid UUID, and correspond to an existing, active property.

checkInDate: Must be present, a valid date in YYYY-MM-DD format, must be in the future, and must be strictly before checkOutDate.

checkOutDate: Must be present, a valid date in YYYY-MM-DD format, and must be strictly after checkInDate.

numberOfGuests: Must be present, an integer, and at least 1. It must not exceed the property's maximum guest capacity (if defined).

Business Logic Validation: Crucially, the system must perform an availability check to ensure that the specified propertyId is not already booked or unavailable for any portion of the requested checkInDate to checkOutDate range. If there's an overlap, the booking must be rejected.

3.2. **Get My Bookings (Requires Authentication - User Role)**

Endpoint URL: /api/bookings/my

HTTP Method: GET

Authentication: Requires a valid JWT Bearer Token.

Query Parameters (Optional for filtering/pagination):

status: Filter by booking status (enum, e.g., 'pending', 'confirmed', 'cancelled', 'completed').

page: Page number (integer, default 1).

limit: Number of items per page (integer, default 10).

Response Body (Output - Success):

A list of booking objects specific to the authenticated user. Each booking includes: booking ID, simplified property details (ID, title, city, country, a sample media URL), check-in/out dates, number of guests, total price, current status, and creation timestamp.

Pagination metadata: total number of items, current page number, total number of pages.

Validation Rules:

status: Optional, but if provided, must be one of the allowed enum values.

3.3. **Get Bookings for My Properties (Requires Authentication - Owner Role)**

Endpoint URL: /api/owner/bookings

HTTP Method: GET

Authentication: Requires a valid JWT Bearer Token. The authenticated user must possess an 'owner' role.

Query Parameters (Optional for filtering/pagination):

propertyId: Filter bookings for a specific property owned by the user (UUID).

status: Filter by booking status (enum, e.g., 'pending', 'confirmed', 'cancelled', 'completed').

page: Page number (integer, default 1).

limit: Number of items per page (integer, default 10).

Response Body (Output - Success):

A list of booking objects for properties owned by the authenticated user. Similar to Get My Bookings, but also includes essential user details (ID, first name, last name, email) of the user who made the booking.

Pagination metadata: total number of items, current page number, total number of pages.

Validation Rules:

propertyId: Optional, but if provided, must be a valid UUID and belong to one of the properties owned by the authenticated user.

status: Optional, but if provided, must be one of the allowed enum values.

3.4. **Cancel Booking (Requires Authentication - User Role, for own bookings)**

Endpoint URL: /api/bookings/{id}/cancel

HTTP Method: POST

Authentication: Requires a valid JWT Bearer Token. The authenticated user must be the one who created the booking.

Path Parameters (Input):

id: The unique identifier (UUID) of the booking to be cancelled.

Response Body (Output - Success):

A message confirming that the booking has been successfully cancelled.

The booking ID.

The updated status ('cancelled').

The cancellation timestamp.

Response Body (Output - Error):

An error message indicating that the booking cannot be cancelled (e.g., "Booking already completed," "Cancellation deadline passed") or that the user is not authorized to cancel it.

Validation Rules:

id: Must be present and a valid UUID.

Business Logic Validation:

Only the user who initiated the booking can request its cancellation.

Bookings can only be cancelled if their current status is 'pending' or 'confirmed'.

A cancellation deadline policy must be enforced (e.g., no cancellations allowed within 24 hours of check-in).

Performance Criteria:

Latency:

POST /api/bookings: Must respond within 300 milliseconds due to the critical availability checks involved.

GET /api/bookings/my and GET /api/owner/bookings: Should respond within 250 milliseconds.

POST /api/bookings/{id}/cancel: Should respond within 200 milliseconds.

Throughput:

POST /api/bookings: The system should handle at least 20 requests per second (RPS).

GET /api/bookings/my and GET /api/owner/bookings: The system should handle at least 80 requests per second (RPS).

Concurrency: The booking system must robustly handle concurrent booking attempts for the same property and dates, absolutely preventing any overbooking scenarios. This implies the use of appropriate concurrency control mechanisms (e.g., database transactions, locking).

Accuracy: Booking availability checks and subsequent status updates must be 100% accurate to prevent double bookings and ensure data integrity.
