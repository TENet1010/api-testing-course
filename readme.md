# API Testing Lab: Auntie Som’s Noodle Stall

> **Objective:** Prove that Nephew Lek’s "it works on my machine" attitude is a recipe for disaster.

---

## The Backstory

Auntie Som runs a **legendary noodle stall**. Her Tom Yum is world-class, her customers are loyal, and her business is booming.  

Recently, her nephew **Lek** (who just finished a 2-week coding bootcamp) decided to "digitize" the business. He built a backend API to handle online orders. Auntie Som is thrilled, but the code… well, Lek says:  

> *"It’s fine lah Auntie! I tested it once with my own phone. No need for professional testing!"*

**You are the Last Line of Defense.**  
Before Auntie Som launches this app to 1,000+ hungry customers, you must audit the system, find the logic flaws, and automate the validation using **Bruno**.

---

## Learning Objectives

By the end of this lab, you should be able to:
- Write **Bruno test scripts** for automated validation.
- Extract values from API responses using **Post-response scripts**.
- Manage and use **Environment Variables**.
- Verify **Business Rules**, not just HTTP status codes.
- Catch regressions automatically before they hit production.

---

## Setup Instructions

### 1. Prepare the Project
- **Fork this repository** to your own GitHub account.
- **Clone it** locally to your machine.

### 2. Start the API Server
The backend is a lightweight Flask application.
- **Requirements:** Python 3.10+ installed.
- **Navigate** to the `auntie-som-noodle-api` folder.
- **Install dependencies:**  
  ```bash
  pip install flask
  ```
- **Run the server:**  
  ```bash
  python app.py
  ```
- The server will run at: `http://localhost:5000`

### 3. Setup Bruno
- Install [Bruno](https://www.usebruno.com) and open the `auntie-som-noodle-api-test` folder as a collection.
- Select the `local` environment and run the entire collection in sequence.

---

## API Reference

### Authentication
`POST /auth/login` - Get an access token.
- **Body (JSON):**
  ```json
  {
    "username": "student",
    "password": "hungry"
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "accessToken": "uuid-token-string",
    "expiresIn": 3600
  }
  ```

---

### Menu
`GET /menu` - View available items and stock levels.
- **Response (200 OK):**
  ```json
  [
    { "id": 1, "name": "Tom Yum Noodles", "price": 50, "stock": 2 },
    { "id": 2, "name": "Fried Rice", "price": 45, "stock": 1 }
  ]
  ```

---

### Orders
`POST /orders` - Place a new order.
- **Headers:** `Authorization: Bearer <token>`
- **Body (JSON):**
  ```json
  {
    "itemId": 1,
    "quantity": 1
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

`GET /orders/<orderId>` - Retrieve order details.
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

---

## Your Mission: The Silent Auditor

Your task is to create a comprehensive Bruno collection that validates the entire workflow.  

**The Catch:** Nephew Lek left several **logic bugs** in the system. Some endpoints might return `200 OK` even when the business logic is completely broken.

**Your collection must include:**
1.  **Auth Flow**: Automatically store the `accessToken` from login and use it for subsequent requests.
2.  **Order Validation**: Ensure orders can only be placed with valid auth, valid items, and available stock.
3.  **Data Integrity**: Check that the `totalPrice` calculated by the API matches your expectations.
4.  **Edge Cases**: What happens if you order more than what's in stock? What happens if you request a non-existent order?

> [!TIP]
> **Clicking "Send" is not testing.** Your tests must contain scripts (Assertions or Javascript) that **FAIL** when the system behaves incorrectly.

---

## Rules of the Lab
- **Do NOT** modify the `app.py` backend code.
- **Do NOT** rely on manual checking.
- **Use environment variables** for the Base URL and Tokens.
- **Use Scripts/Assertions** for all validations.

---

## What to Submit

1.  **Push your Bruno collection folder** to your GitHub repository.
2.  **Update your README.md** (the one in your fork) with:
    - A summary of the bugs you discovered.
    - How your tests detect these bugs.
4.  **Send the GitHub link** to your instructor via Discord.

---
*Good luck. Auntie Som is counting on you!*

---

## HW3 audit results

### Run the collection

Start the Flask API using the setup instructions above. Then open a second terminal at the repository root and run the collection with Node.js and the Bruno CLI.

```bash
cd auntie-som-noodle-api-test
npx --yes --package @usebruno/cli@4.3.0 bru run --env local
```

Restart Flask before each full run so the in-memory menu starts with two portions of Tom Yum Noodles and one of Fried Rice. Use the collection order and let the runner continue after failed checks so that the stock checks still run.

The login post-response script saves the token in the selected Bruno environment. Authenticated order requests use it as a Bearer token. The valid order's ID is captured for the next request to look it up. The last request clears the token from the environment.

### Findings

The collection ran 15 requests against the supplied Flask app. Bruno reported 17 passing checks and 9 failing checks across the five findings below. The valid login, missing and invalid token rejection, rejection of unknown menu items, retrieval of a created order, and stock deduction for a valid purchase all passed.

| Finding | Expected | Observed and how Bruno detects it |
| --- | --- | --- |
| An unexplained 5-baht deduction from the order total | The listed unit price multiplied by quantity, assuming no discount. | `07 Create valid order` reads the menu price and multiplies it by the requested quantity. One bowl should total 50 baht under this assumption, but the order returns 45. The lab's response example also shows 45 and gives no discount rule. If a flat 5-baht discount is intended, this result matches that rule. |
| Orders can exceed available stock | HTTP 400 with `Out of stock`. Rice stock stays at 1. | `10 Order above stock` requests two portions when one is available and receives HTTP 200. `11 Stock after oversized order` sees stock drop from 1 to −1. |
| Negative quantities increase stock | HTTP 400. Noodle stock stays at 1. | `12 Order negative quantity` sends −1 and receives HTTP 200. `13 Stock after negative quantity` sees stock increase from 1 to 2. |
| The API accepts a token without the Bearer scheme | HTTP 401 with `Unauthorized`. | `14 Order with bare token` sends the valid token as the entire Authorization header and receives HTTP 200. |
| Looking up a missing order reports HTTP success | HTTP 404 with `Order not found`. | `15 Read nonexistent order` receives the expected error message with HTTP 200. The status check fails. |
