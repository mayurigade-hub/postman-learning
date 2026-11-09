# Day 01 — HTTP Responses & Status Codes 🚀

## Overview

This document covers HTTP responses, status codes, response headers and response bodies using practical examples from my **Wanderlust project**, built with Node.js, Express.js and MongoDB.

The objective is to understand how a server communicates the result of an HTTP request to a client such as Postman.

---

# 1. Understanding HTTP Responses

An **HTTP Response** is a message sent by the server to the client after receiving and processing an HTTP request.

### How HTTP communication works

For example, when we send the following request from Postman:

```http
GET http://localhost:8080/listings
```

The communication takes place as follows:

1. Postman sends an HTTP request.
2. Express.js receives the request.
3. Express processes the request and interacts with MongoDB if necessary.
4. Express generates and sends an HTTP response.
5. Postman receives and displays the response.

### Client vs Server

| Component | Responsibility |
|---|---|
| Postman (Client) | Sends HTTP requests and receives responses |
| Express.js (Server) | Processes requests and sends HTTP responses |
| MongoDB (Database) | Stores and retrieves application data |

**Important:** MongoDB does not directly send the HTTP response to Postman. Express.js handles the HTTP communication.

## 1.1 Three Main Components of an HTTP Response

An HTTP response generally contains three important components:

1. **Status Code:** Indicates the result of the request.
2. **Response Headers:** Provide additional information about the response.
3. **Response Body:** Contains the actual content returned by the server.

---

# 2. HTTP Status Codes

An HTTP status code is a three-digit number returned by the server to indicate the result of processing a client's request.

HTTP status codes are divided into five categories.

| Category | Meaning |
|---|---|
| 1xx | Informational |
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client Error |
| 5xx | Server Error |

## 2.1 Informational Responses — 1xx

These responses indicate that the server has received the request and is continuing to process it.

| Status Code | Meaning | Explanation |
|---|---|---|
| 100 | Continue | Client can continue sending the request |
| 101 | Switching Protocols | Server agrees to switch protocols |
| 102 | Processing | Server has received the request and is processing it |
| 103 | Early Hints | Provides preliminary information before the final response |

### 100 Continue

Suppose a client wants to upload a large file.

1. The client sends the request headers.
2. The server responds with `100 Continue`.
3. The client proceeds with sending the request body.

**Remember:** 100 Continue means the client can continue sending the request.

## 2.2 Success Responses — 2xx

These responses indicate that the request was successfully received and processed.

| Status Code | Meaning | Typical Usage |
|---|---|---|
| 200 | OK | Successfully fetching data |
| 201 | Created | Successfully creating a resource |
| 204 | No Content | Successful operation without a response body |

### 200 OK

Indicates that the request was successfully processed.

Example from Wanderlust:

```js
router.get('/', async (req, res) => {
    const allListings = await Listing.find({});
    res.render('listings/index.ejs', { allListings });
});
```

When listings are fetched successfully, the server can return `200 OK`.

### 201 Created

Indicates that a new resource was successfully created.

Example:

```http
POST /listings
```

If the API successfully creates a listing, it can return:

```http
201 Created
```

### 204 No Content

Indicates that the request was successfully processed, but the server has no response body to return.

Example:

```js
res.status(204).end();
```

This is commonly used when an API successfully deletes a resource without returning additional data.

### Difference Between 200, 201 and 204

- **200:** Request successfully processed.
- **201:** New resource successfully created.
- **204:** Request succeeded, but no response body is returned.

**Wanderlust observation:** Our existing POST and DELETE routes use redirects and may produce different response statuses. The appropriate status depends on the backend implementation.

## 2.3 Redirection Responses — 3xx

These responses indicate that the client needs to take additional action, usually to access a different URL.

| Status Code | Meaning | Explanation |
|---|---|---|
| 301 | Moved Permanently | Resource has permanently moved |
| 302 | Found | Temporary redirection |
| 304 | Not Modified | Client can reuse its cached resource |

### 301 Moved Permanently

Indicates that a resource has permanently moved to a new URL.

Example:

```http
HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-page
```

The `Location` header specifies the new URL.

### 302 Found

Indicates that a resource is temporarily available at another URL.

Example:

```http
HTTP/1.1 302 Found
Location: /login
```

### Connection with Express.js

Express provides the `res.redirect()` method.

```js
res.redirect('/listings');
```

By default, Express sends a `302` redirect.

Our Wanderlust application uses redirects after operations such as login, logout, creating listings and deleting listings.

### 304 Not Modified

Used in HTTP caching.

Suppose a browser has already downloaded an image and stored it in its cache.

Later, it asks the server whether the image has changed.

If the resource has not changed, the server can return:

```http
304 Not Modified
```

The client can reuse its cached copy instead of downloading the resource again.

**Important:** A 304 response does not contain the resource body.

## 2.4 Client Error Responses — 4xx

These responses indicate that the server cannot fulfil the request because of an issue with the request or the client's permissions.

| Status Code | Meaning | Explanation |
|---|---|---|
| 400 | Bad Request | Invalid or malformed request |
| 401 | Unauthorized | Missing or invalid authentication credentials |
| 403 | Forbidden | Authenticated client lacks permission |
| 404 | Not Found | Requested resource does not exist |
| 405 | Method Not Allowed | HTTP method is not supported |
| 409 | Conflict | Request conflicts with current resource state |
| 422 | Unprocessable Content | Request is understood but fails semantic validation |

### 400 Bad Request

Indicates that the server cannot process the request because it contains invalid or incorrect information.

Example:

Our Wanderlust backend expects:

```json
{
  "listing": {
    "title": "Beach House",
    "price": 2500
  }
}
```

But suppose we send:

```json
{
  "title": "Beach House",
  "price": 2500
}
```

The expected `listing` wrapper is missing.

The backend uses `req.body.listing`, so the request does not have the expected structure.

A `400 Bad Request` response may be appropriate.

### 401 Unauthorized

Indicates that the request does not contain valid authentication credentials.

Common scenarios:

- User is not logged in.
- Authentication token is missing.
- Credentials are invalid or expired.

**Wanderlust connection:** Our `isLoggedIn` middleware checks whether the user is authenticated.

```js
if (!req.isAuthenticated()) {
    req.session.redirectUrl = req.originalUrl;
    req.flash("error", "You must be logged in!");
    return res.redirect("/login");
}
```

Important: Our current application redirects unauthenticated users to `/login`, usually with a `302` response, rather than returning `401`.

### 403 Forbidden

Indicates that the server understands the user's identity but refuses to authorize the requested action.

Example:

- User A owns a listing.
- User B is logged in.
- User B attempts to delete User A's listing.
- The ownership check fails.

This is a typical `403 Forbidden` scenario.

### Difference Between 401 and 403

| 401 Unauthorized | 403 Forbidden |
|---|---|
| Authentication is missing or invalid | Client is authenticated but lacks permission |
| Identity cannot be properly established | Identity is established, but access is denied |

### 404 Not Found

Indicates that the requested resource cannot be found.

Example:

```http
GET /listings/6ae955205cbbd674e7edac82
```

If the ObjectId is valid but no listing exists with that ID, the application should ideally return `404 Not Found`.

**Important distinction:**

- Invalid ObjectId format: often handled as `400 Bad Request`.
- Valid ObjectId but no matching listing: `404 Not Found`.

### 405 Method Not Allowed

Indicates that the HTTP method used is not supported for the requested resource.

Example:

Suppose an API supports:

```http
GET /listings
POST /listings
```

But the client sends:

```http
DELETE /listings
```

If DELETE is not supported for that resource, the server can return `405 Method Not Allowed`.

### 409 Conflict

Indicates that the request conflicts with the current state of a resource.

Example:

A user attempts to register with an email address that already exists.

The API may return:

```http
409 Conflict
```

### 422 Unprocessable Content

Indicates that the server understands the request and its syntax is valid, but the supplied data fails semantic validation.

Example from Wanderlust:

```json
{
  "listing": {
    "title": "Beach House",
    "price": -500
  }
}
```

The JSON is syntactically valid, but the price violates the validation rule:

```js
price: {
    type: Number,
    min: 0,
    required: true
}
```

An API can return:

```http
422 Unprocessable Content
```

### Difference Between 400 and 422

| 400 Bad Request | 422 Unprocessable Content |
|---|---|
| Request is invalid or malformed | Request is understood but fails semantic validation |
| Example: malformed JSON | Example: negative price |
| General request error | Often used for validation failures |

Both 400 and 422 can be used for validation failures, depending on the API's design.

## 2.5 Server Error Responses — 5xx

These responses indicate that the server encountered a problem while processing a request.

| Status Code | Meaning | Explanation |
|---|---|---|
| 500 | Internal Server Error | Unexpected server-side error |
| 502 | Bad Gateway | Gateway received an invalid upstream response |
| 503 | Service Unavailable | Server is temporarily unable to handle requests |

### 500 Internal Server Error

Indicates that the server encountered an unexpected condition and could not fulfil the request.

Common scenarios:

- Unexpected JavaScript exceptions.
- Database connection failures.
- Unhandled server-side errors.
- Internal application failures.

Example from Wanderlust:

```js
router.get('/', async (req, res) => {
    const allListings = await Listing.find({});
    res.render('listings/index.ejs', { allListings });
});
```

If an unexpected error occurs while executing the route and the error handler responds with status 500, Postman displays:

```http
500 Internal Server Error
```

### 502 Bad Gateway

Occurs when a server acting as a gateway or proxy receives an invalid response from an upstream server.

Example:

A reverse proxy forwards a request to another server but receives an invalid response.

It can return:

```http
502 Bad Gateway
```

### 503 Service Unavailable

Indicates that the server is temporarily unable to handle requests.

Common reasons:

- Server maintenance.
- Server overload.
- Temporary service outage.

A response may include:

```http
Retry-After: 120
```

This indicates when the client should retry, expressed here as 120 seconds.

### Difference Between 500, 502 and 503

| Status | Meaning |
|---|---|
| 500 | Unexpected internal server error |
| 502 | Gateway received an invalid upstream response |
| 503 | Server temporarily unavailable |

---

# 3. HTTP Response Headers

## 3.1 What are HTTP Headers?

HTTP headers are additional pieces of information sent along with HTTP requests and responses.

They provide metadata about the communication between the client and server.

Headers can describe:

- Content format.
- Response size.
- Authentication and cookies.
- Caching rules.
- Server information.

### Request Headers vs Response Headers

| Request Headers | Response Headers |
|---|---|
| Sent by the client | Sent by the server |
| Describe the request | Describe the response |
| Example: Accept | Example: Content-Type |
| Example: Authorization | Example: Set-Cookie |

## 3.2 Important Response Headers

### A. Content-Type

Specifies the media type of the response body.

Example:

```http
Content-Type: text/html; charset=utf-8
```

This tells the client that the response contains HTML.

Other examples:

```http
Content-Type: application/json
Content-Type: text/plain
Content-Type: image/png
```

**Wanderlust observation:**

Our `res.render()` route returns HTML, so the response contains:

```http
Content-Type: text/html; charset=utf-8
```

### B. Content-Length

Specifies the size of the response body in bytes.

Example:

```http
Content-Length: 3880
```

This means the response body is 3,880 bytes long.

Remember: Content-Length measures bytes, not the number of characters.

### C. Set-Cookie

Instructs the client to store a cookie sent by the server.

Example:

```http
Set-Cookie: connect.sid=abc123; HttpOnly; Path=/
```

Parts:

- `connect.sid`: Session cookie name.
- `abc123`: Example cookie value.
- `HttpOnly`: Prevents ordinary client-side JavaScript from reading the cookie.
- `Path=/`: Cookie applies to the root path and its subpaths.

**Wanderlust connection:**

Our Express session middleware uses a session cookie commonly named `connect.sid`.

Important distinction:

- `Set-Cookie`: Server instructs the client to store a cookie.
- `Cookie`: Client sends stored cookies back to the server.

### D. Location

Specifies a destination URL or resource location.

It is commonly used with redirection responses.

Example:

```http
HTTP/1.1 302 Found
Location: /login
```

**Wanderlust connection:**

```js
res.redirect('/listings');
```

Express sends a redirect response with a `Location` header.

### E. Cache-Control

Defines caching instructions for browsers and other HTTP clients.

Example:

```http
Cache-Control: no-cache
```

Important directives:

| Directive | Meaning |
|---|---|
| no-cache | Stored response must be revalidated before reuse |
| no-store | Response should not be stored |
| public | Response may be stored by shared caches |
| private | Response is intended for private caches |
| max-age=3600 | Response can be considered fresh for 3600 seconds |

**Important:** `no-cache` does not mean the response cannot be stored. `no-store` is used to prevent storage.

### F. Date

Indicates when the server generated the response.

Example:

```http
Date: Mon, 09 Nov 2026 06:00:00 GMT
```

### G. Server

Can identify the software used by the server.

Example:

```http
Server: nginx
```

Servers may omit or hide this header for security reasons.

### H. ETag

An ETag is an identifier associated with a particular representation of a resource.

It helps clients and servers perform cache validation.

Example:

```http
ETag: W/"..."
```

The `W/` prefix indicates a weak ETag.

### I. X-Powered-By

Can identify the technology or framework powering an application.

Example:

```http
X-Powered-By: Express
```

Our actual Wanderlust response showed this header, identifying Express as the server framework.

### J. Connection and Keep-Alive

`Connection` can specify whether a network connection should remain open.

Example:

```http
Connection: keep-alive
```

`Keep-Alive` can provide connection parameters.

Example:

```http
Keep-Alive: timeout=5
```

These headers are relevant to HTTP/1.1 connection management. HTTP/2 and HTTP/3 manage connections differently.

## 3.3 Practical: Inspecting Response Headers in Postman

Using the existing Wanderlust GET All Listings request:

1. Send the GET request.
2. Open the response Headers tab.
3. Inspect the returned headers.

### Actual headers observed in Wanderlust

| Header | Observed Value | Meaning |
|---|---|---|
| X-Powered-By | Express | Identifies Express |
| Content-Type | text/html; charset=utf-8 | HTML response |
| Content-Length | 3880 | Response size in bytes |
| ETag | W/"..." | Cache validation identifier |
| Set-Cookie | connect.sid=... | Session cookie |
| Date | Server response date | Response generation time |
| Connection | keep-alive | Connection can be reused |
| Keep-Alive | timeout=5 | Connection timeout parameter |

This practical exercise confirmed how HTTP response headers work in a real Express application.

---

# 4. HTTP Response Body

## 4.1 What is a Response Body?

The response body contains the actual content or data returned by the server to the client.

For example, when we send:

```http
GET http://localhost:8080/listings
```

The Express server fetches the listings and returns a rendered page.

That returned HTML is the response body.

## 4.2 Common Response Body Formats

### A. JSON Response

JSON stands for JavaScript Object Notation.

It is commonly used in REST APIs to exchange structured data between frontend and backend applications.

Example:

```json
{
  "success": true,
  "message": "Listing fetched successfully",
  "listing": {
    "title": "Beach House",
    "price": 2500,
    "location": "Goa"
  }
}
```

In Express:

```js
res.json({
    success: true,
    message: "Listing fetched successfully"
});
```

The response header will typically be:

```http
Content-Type: application/json; charset=utf-8
```

### B. HTML Response

HTML is used to represent web pages.

Our Wanderlust project uses EJS templating to generate HTML.

Example:

```js
router.get('/', async (req, res) => {
    const allListings = await Listing.find({});
    res.render('listings/index.ejs', { allListings });
});
```

Here, `res.render()` renders the EJS template into HTML and sends it to the client.

The response header typically contains:

```http
Content-Type: text/html; charset=utf-8
```

### C. Plain Text Response

A plain-text response contains ordinary text rather than structured JSON or HTML.

Example:

```js
app.get('/hello', (req, res) => {
    res.type('text/plain').send('Hello from Express!');
});
```

Response body:

```text
Hello from Express!
```

Response header:

```http
Content-Type: text/plain; charset=utf-8
```

## 4.3 JSON vs HTML vs Plain Text

| Format | Description | Express Method |
|---|---|---|
| JSON | Structured data | `res.json()` |
| HTML | Rendered web page | `res.render()` |
| Plain Text | Ordinary text | `res.type('text/plain').send()` |

## 4.4 Raw vs Preview in Postman

Postman provides different ways to inspect a response body.

### Raw

Displays the response body as received, without rendering it as a webpage.

For an HTML response, Raw displays the HTML markup.

### Preview

Attempts to render supported response content visually.

For an HTML response:

- Raw: Displays HTML tags and markup.
- Preview: Displays the rendered webpage.

**Important:** Preview is a display feature in Postman. It does not change the actual HTTP response sent by the server.

---

# 5. Practical Learning — Wanderlust API

During this learning session, we used Postman to practise the following operations:

| HTTP Method | Endpoint | Purpose |
|---|---|---|
| GET | /listings | Fetch all listings |
| GET | /listings/:id | Fetch a particular listing |
| POST | /listings | Create a new listing |
| PUT | /listings/:id | Update an existing listing |
| DELETE | /listings/:id | Delete a listing |

We also practised:

- Sending JSON request bodies.
- Understanding `express.json()` middleware.
- Logging in using `x-www-form-urlencoded`.
- Understanding session-based authentication.
- Inspecting `connect.sid`.
- Inspecting HTML response bodies.
- Understanding status codes and redirects.
- Inspecting response headers in Postman.

### Important observations

1. Our GET requests returned HTML because the application uses EJS rendering.
2. Our POST and PUT routes used redirects, so Postman could follow the redirect and display the final response.
3. Our login request used `x-www-form-urlencoded` because the Express login route uses form-based credentials.
4. Our Express session middleware generated a `connect.sid` cookie.
5. MongoDB stores the application data, while Express handles the HTTP request and response.

---

# 6. Important Revision Table

## Status Codes

| Status | Meaning |
|---|---|
| 100 | Continue |
| 101 | Switching Protocols |
| 102 | Processing |
| 103 | Early Hints |
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 301 | Moved Permanently |
| 302 | Found |
| 304 | Not Modified |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 409 | Conflict |
| 422 | Unprocessable Content |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

## Important Response Headers

| Header | Purpose |
|---|---|
| Content-Type | Response media type |
| Content-Length | Response body size in bytes |
| Set-Cookie | Instructs client to store a cookie |
| Location | Redirect destination or resource location |
| Cache-Control | Caching instructions |
| Date | Response generation date |
| Server | Server software information |
| ETag | Resource representation identifier |
| X-Powered-By | Technology identification |
| Connection | Connection management |

---

# 7. Important Interview Questions

1. What is an HTTP response?
2. What are the three main components of an HTTP response?
3. What are the five categories of HTTP status codes?
4. What is the difference between 200, 201 and 204?
5. What is the difference between 301 and 302?
6. What is the purpose of 304 Not Modified?
7. What is the difference between 401 Unauthorized and 403 Forbidden?
8. What is the difference between 400 Bad Request and 422 Unprocessable Content?
9. What is the difference between 500, 502 and 503?
10. What are HTTP response headers?
11. What is the difference between Content-Type and Content-Length?
12. What is the purpose of Set-Cookie?
13. What is the difference between Cookie and Set-Cookie?
14. What is the purpose of the Location header?
15. What is the difference between Cache-Control: no-cache and no-store?
16. What is an ETag?
17. What is the difference between JSON, HTML and plain-text responses?
18. What is the difference between Raw and Preview in Postman?

---

# 8. Key Takeaways

- HTTP responses are generated by the server after processing client requests.
- Status codes indicate the outcome of a request.
- HTTP status codes are divided into five categories: 1xx, 2xx, 3xx, 4xx and 5xx.
- Response headers provide metadata about the response.
- The response body contains the actual returned content.
- Express provides methods such as `res.status()`, `res.json()`, `res.send()`, `res.render()` and `res.redirect()` to generate responses.
- Postman helps inspect status codes, headers and response bodies.
- Our Wanderlust application uses session-based authentication and EJS-rendered HTML responses.

---

**Day 01 Completed — HTTP Responses & Status Codes** ✅
