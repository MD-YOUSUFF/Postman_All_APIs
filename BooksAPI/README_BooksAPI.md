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
- [Jenkins](https://www.jenkins.io/download/) (optional), to automate runs from CI

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

## Run with Jenkins (Freestyle Project)

Jenkins can run the Newman command automatically and keep the HTML report from every build.

### Prerequisites
- Java 17 or 21 (`java -version`)
- `jenkins.war` from https://www.jenkins.io/download/ (Generic Java package)
- Node.js, `newman` and `newman-reporter-htmlextra` installed globally (see the Newman section above)

### Start Jenkins
1. Open cmd and run:
   ```
   cd C:\Jenkins
   java -jar jenkins.war
   ```
2. Wait for `Jenkins is fully up and running`.
3. Open `http://localhost:8080` and log in. On the first run, use the initial admin password shown in the cmd log or in `C:\Users\<username>\.jenkins\secrets\initialAdminPassword`. Install the suggested plugins and create your admin user.

### Create a Freestyle Project
1. Click **New Item**, enter a name (for example `BooksAPI-Newman`), select **Freestyle project** and click **OK**.
2. Under **Source Code Management**, select **Git** and add this repository URL, with the branch `*/main`.
3. Under **Build Steps → Add build step → Execute Windows batch command**, add:
   ```
   call newman run BooksAPI_postman_collection.json --env-var "baseUrl_collect=<your-base-url>" -r cli,htmlextra
   ```
   `call` is needed on Windows. Without it, the batch script stops after Newman runs.
4. Under **Post-build Actions → Archive the artifacts**, enter `newman/*.html` so the report is saved with each build.
5. Click **Save**.

### Run and View Results
1. Click **Build Now**.
2. Open the build under **Build History**:
   - **Console Output** shows the Newman test results.
   - **Build Artifacts** has the HTML report.
3. A build is green when all tests pass and red when any test fails.

### Stop Jenkins
Press `Ctrl+C` in the cmd window.

## Notes

- Tokens expire, so re-run **Register API Client** if authorised calls start failing.
- Keep real tokens out of the repo. Clear `authorization_secret_token` before exporting and pushing.
- Possible improvements: add tests to *Delete an order* and *Update an order* (status code and body checks), and add negative test cases (invalid token, invalid book ID).

## Author

**MD Yousuff** – QA Tester / SDET
GitHub: [MD-YOUSUFF](https://github.com/MD-YOUSUFF) · LinkedIn: [Mohammed Yousuff](https://www.linkedin.com/in/mohammed-yousuff-068484217)
