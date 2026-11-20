# Books API – Postman Collection

Postman collection for testing a **Books REST API** (books catalogue and order management). It covers API status, client registration with token generation, reading books, and the full order lifecycle (create, read, update, delete), with automated test scripts on the requests.

## Requests Covered

| # | Method | Request | Endpoint | Auth |
|---|--------|---------|----------|------|
| 1 | GET | API Status | `/status` | No |
| 2 | POST | Register API Client | `/api-clients/` | No |
| 3 | GET | List of Books | `/books` | No |
| 4 | GET | Get a single book | `/books/1` | No |
| 5 | POST | Order Book | `/orders` | Bearer token |
| 6 | GET | Get all books orders | `/orders` | Bearer token |
| 7 | GET | Get an Order | `/orders/{{orderId_collect}}` | Bearer token |
| 8 | PATCH | Update an order | `/orders/{{orderId_collect}}` | Bearer token |
| 9 | DELETE | Delete an order | `/orders/{{orderId_collect}}` | Bearer token |

> Run the requests **in this order**. Request 2 generates the token, and request 5 generates the order ID used by requests 7 to 9.

## Collection Variables

| Variable | Purpose | Set by |
|----------|---------|--------|
| `baseUrl_collect` | Base URL of the Books API | **You** (set once before running) |
| `authorization_secret_token` | Bearer token used for authorised requests | Auto-saved by *Register API Client* |
| `orderId_collect` | ID of the order created | Auto-saved by *Order Book* |

Authorization is set at collection level as **Bearer Token** with `{{authorization_secret_token}}`, so every request inherits it.

## Prerequisites

- [Postman](https://www.postman.com/downloads/)
- Internet access (or the API running locally)
- [Node.js](https://nodejs.org/) (includes npm), only needed for command-line runs with Newman

## Setup

1. Open Postman and go to **Import → File**, then select `BooksAPI_postman_collection.json`.
2. Open the collection → **Variables** tab.
3. Set `baseUrl_collect` (Current value) to the Books API base URL, with no trailing slash.
4. Leave `authorization_secret_token` and `orderId_collect` empty. They are filled automatically.
5. Click **Save**.

## How to Run

1. **API Status** – confirms the API is up.
2. **Register API Client** – registers a client and saves the access token.
3. **List of Books** and **Get a single book** – read the catalogue.
4. **Order Book** – creates an order and saves its ID.
5. **Get all books orders**, **Get an Order**, **Update an order** – verify and modify the order.
6. **Delete an order** – removes the order.

You can also run the whole collection with **Collection Runner**.

## Tests Included

| Request | Checks |
|---------|--------|
| API Status | Status code 200; `status` field equals `OK` |
| Register API Client | Status code 201; access token saved to a collection variable |
| List of Books | Status code 200; response matches the expected JSON schema |
| Get a single book | Status code 200; `author` and `price` fields present; response time below 1000 ms |
| Order Book | Status code 201; `created` is `true`; `orderId` present; order ID saved |
| Get all books orders | Status code 200; response is a non-empty array; each order has the required fields |
| Get an Order | Status code 200 |
| Update an order | `Content-Type` header present and includes `application/json` |
| Delete an order | No test script yet |

## Run from Command Line (Newman)

```bash
node -v
npm -v
npm install -g newman
npm install -g newman-reporter-htmlextra
```

Export the collection (and an environment, if you use one), then run:

```bash
newman run BooksAPI_postman_collection.json --env-var "baseUrl_collect=<your-base-url>"
```

Generate an HTML report:

```bash
newman run BooksAPI_postman_collection.json --env-var "baseUrl_collect=<your-base-url>" -r cli,htmlextra
```

The report is saved in the `newman` folder.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Requests fail with a connection or 404 error | Check `baseUrl_collect` is set correctly and has no trailing slash |
| `401 Unauthorized` on order requests | Run **Register API Client** first to generate a fresh token |
| Register request fails because the client already exists | Change `clientEmail` in the request body to a new unique email |
| Order requests fail with a missing ID | Run **Order Book** first so `orderId_collect` is set |
| Token or order ID is empty after running | Check the **Tests** tab of the request and the Postman Console |

## Notes

- Tokens expire, so re-run **Register API Client** if authorised calls start failing.
- Keep real tokens out of the repo. Clear `authorization_secret_token` before exporting and pushing.
- Possible improvements: add tests to *Delete an order* and *Update an order* (status code and body checks), and add negative test cases (invalid token, invalid book ID).

## Author

**MD Yousuff** – QA Tester / SDET
GitHub: [MD-YOUSUFF](https://github.com/MD-YOUSUFF) · LinkedIn: [Mohammed Yousuff](https://www.linkedin.com/in/mohammed-yousuff-068484217)
