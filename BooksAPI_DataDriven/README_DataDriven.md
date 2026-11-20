# Books API – Data-Driven Testing (CSV and JSON)

Postman collection **BooksAPI-DataDriven** that runs the same order requests many times with different test data. The test data is kept in external **CSV** and **JSON** files, so new test cases can be added by editing the data file, with no change to the requests.

## Requests Covered

| # | Method | Request | Endpoint |
|---|--------|---------|----------|
| 1 | POST | Order Book | `/orders` |
| 2 | GET | Get an Order | `/orders/{{orderId_collect}}` |
| 3 | DELETE | Delete an order | `/orders/{{orderId_collect}}` |

Each data row runs all three requests once (one iteration per row).

## Environment Variables

The order ID is stored in an **environment variable** during each iteration and removed at the end, so every data row starts clean.

| Variable | Created in | Removed in |
|----------|-----------|------------|
| `orderId_collect` | **Order Book** (Tests/Scripts tab), after the order is created | **Delete an order** (Tests/Scripts tab), at the end of the iteration |

Example scripts:

```javascript
// Order Book: save the order ID
pm.environment.set("orderId_collect", pm.response.json().orderId);

// Delete an order: clean up
pm.environment.unset("orderId_collect");
```

Because the variable is removed after **Delete an order**, a stale order ID from one row is never reused in the next row.

## Test Data

The same data is kept in two formats. Use either one.

### CSV file (`books_data.csv`)

```csv
BookID,CustomerName
1,John
1,Kim
3,Scott
4,David
6,Mary
```

### JSON file (`books_data.json`)

```json
[
  { "BookID": 1, "CustomerName": "John" },
  { "BookID": 1, "CustomerName": "Kim" },
  { "BookID": 3, "CustomerName": "Scott" },
  { "BookID": 4, "CustomerName": "David" },
  { "BookID": 6, "CustomerName": "Mary" }
]
```

| Field | Used for |
|-------|----------|
| `BookID` | ID of the book to order |
| `CustomerName` | Name of the customer placing the order |

5 rows = **5 iterations**.

## How the Data Is Used

The **Order Book** request body uses the data fields as variables:

```json
{
  "bookId": {{BookID}},
  "customerName": "{{CustomerName}}"
}
```

- The CSV column headers (or JSON keys) must match the variable names exactly, including upper and lower case. `BookID` in the file means `{{BookID}}` in the request.
- In every iteration, Postman replaces `{{BookID}}` and `{{CustomerName}}` with the values from the current row.

## Prerequisites

- [Postman](https://www.postman.com/downloads/)
- [Node.js](https://nodejs.org/) and Newman, only for command-line runs:
  ```bash
  npm install -g newman
  npm install -g newman-reporter-htmlextra
  ```
- An environment selected in Postman (needed for the environment variables to work)
- `baseUrl_collect` set to the Books API base URL
- A valid Bearer token in `authorization_secret_token` (run **Register API Client** first)

## How to Run in Postman (Collection Runner)

1. Select your environment in the top-right dropdown.
2. Right-click the **BooksAPI-DataDriven** collection and click **Run collection**.
3. Under **Data**, click **Select File** and choose `books_data.csv` or `books_data.json`.
4. Check the preview. It should show 5 rows and the iteration count should be 5.
5. Click **Run BooksAPI-DataDriven**.
6. Open each iteration in the results to see the request, response and test results for that row.

## How to Run from Command Line (Newman)

With the CSV file:

```bash
newman run BooksAPI_DataDriven_collection.json -d books_data.csv --env-var "baseUrl_collect=<your-base-url>"
```

With the JSON file:

```bash
newman run BooksAPI_DataDriven_collection.json -d books_data.json --env-var "baseUrl_collect=<your-base-url>"
```

With an HTML report:

```bash
newman run BooksAPI_DataDriven_collection.json -d books_data.csv --env-var "baseUrl_collect=<your-base-url>" -r cli,htmlextra
```

The report is saved in the `newman` folder.

## CSV vs JSON

| | CSV | JSON |
|---|-----|------|
| Easy to edit in Excel | Yes | No |
| Supports nested data and numbers/strings explicitly | No | Yes |
| Best for | Simple flat test data | Structured data |

## Adding a New Test Case

1. Add a new row to the CSV (or a new object to the JSON file).
2. Keep the same field names: `BookID`, `CustomerName`.
3. Run the collection again. The new row runs as an extra iteration.

## Notes

- Keep real tokens out of the repo. Clear `authorization_secret_token` before exporting and pushing.
- Possible improvements: add an expected status code column (for example `ExpectedStatus`) to the data file, and add rows for negative cases (invalid book ID, empty customer name).

## Author

**MD Yousuff** – QA Tester / SDET
GitHub: [MD-YOUSUFF](https://github.com/MD-YOUSUFF) · LinkedIn: [Mohammed Yousuff](https://www.linkedin.com/in/mohammed-yousuff-068484217)
