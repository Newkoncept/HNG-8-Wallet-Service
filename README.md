# HNG Wallet Service API

A FastAPI-based wallet service built during the HNG Backend Internship Stage 8.

This project demonstrates backend API development for wallet operations, authentication, API key access control, Paystack payment initialization, webhook-based wallet crediting, and transaction tracking.

It was developed under a fast-paced internship timeline and is kept as a backend portfolio project to show practical work with authentication, payments, webhooks, database-backed APIs, and service-to-service access patterns.

---

## Overview

The Wallet Service API allows authenticated users to:

* Sign in with Google OAuth
* Receive JWT-based sessions
* Create and manage API keys
* Assign API key permissions
* Create wallet deposits through Paystack
* Validate Paystack webhook events
* Credit wallets after successful payment confirmation
* Transfer funds between wallets
* View wallet balance
* Retrieve wallet transaction history

---

## Core Features

### Authentication

* Google OAuth login flow
* Google callback handling
* User creation after successful OAuth authentication
* JWT access token generation
* Authenticated route access using bearer tokens

### API Key Access

* API key generation
* API key format using `sk_live_<public_id>_<secret>`
* Hashed API key secret storage
* API key verification through the `x-api-key` header
* API key permission control
* API key revocation
* API key rollover for expired keys

Supported API key permissions:

* `read`
* `deposit`
* `transfer`

### Wallet Operations

* Automatic wallet creation for authenticated users
* Wallet balance retrieval
* Wallet deposit initialization
* Wallet-to-wallet transfer
* Wallet transaction history

### Paystack Integration

* Paystack transaction initialization
* Unique transaction reference generation
* Paystack webhook endpoint
* HMAC-SHA512 webhook signature validation
* Successful payment wallet crediting
* Failed or abandoned transaction handling
* Idempotent webhook handling to prevent duplicate wallet crediting

### Transaction Tracking

The project tracks wallet transactions for:

* Deposits
* Incoming transfers
* Outgoing transfers
* Pending transactions
* Successful transactions
* Failed transactions

---

## Tech Stack

| Area                  | Technology                       |
| --------------------- | -------------------------------- |
| Backend Framework     | FastAPI                          |
| Language              | Python 3.11                      |
| Database              | PostgreSQL                       |
| ORM                   | SQLAlchemy                       |
| Migrations            | Alembic                          |
| Authentication        | Google OAuth, JWT                |
| API Access            | API Keys                         |
| Payments              | Paystack                         |
| Webhook Security      | HMAC-SHA512 signature validation |
| HTTP Client           | HTTPX                            |
| Server                | Uvicorn                          |
| Containerization      | Docker                           |
| Dependency Management | uv, pyproject.toml               |

---

## Project Structure

```txt
HNG-8-Wallet-Service/
├── alembic/                  # Database migrations
├── app/
│   ├── database/             # Database configuration and session handling
│   └── features/
│       ├── auth/             # Google OAuth, JWT, user authentication
│       ├── api_keys/         # API key generation, verification, revocation
│       ├── wallet/           # Wallet deposits, transfers, balance, transactions
│       └── transaction/      # Transaction models and related structures
├── Dockerfile
├── main.py                   # FastAPI application entry point
├── pyproject.toml            # Project dependencies
├── uv.lock
└── README.md
```

---

## API Routes

### Default

| Method | Endpoint | Description      |
| ------ | -------- | ---------------- |
| `GET`  | `/`      | Welcome endpoint |

### Authentication

| Method | Endpoint                | Description                                       |
| ------ | ----------------------- | ------------------------------------------------- |
| `GET`  | `/auth/google/`         | Generate Google sign-in URL                       |
| `GET`  | `/auth/google/callback` | Handle Google OAuth callback and return JWT token |

### API Keys

| Method | Endpoint                 | Description                                 |
| ------ | ------------------------ | ------------------------------------------- |
| `POST` | `/keys/create`           | Create a new API key                        |
| `GET`  | `/keys/`                 | List user API keys                          |
| `GET`  | `/keys/active`           | List active API keys                        |
| `POST` | `/keys/{api_key}/revoke` | Revoke an API key                           |
| `POST` | `/keys/rollover`         | Create a replacement key for an expired key |

### Wallet

| Method | Endpoint                             | Description                          |
| ------ | ------------------------------------ | ------------------------------------ |
| `POST` | `/wallet/deposit`                    | Initialize a Paystack wallet deposit |
| `POST` | `/wallet/paystack/webhook`           | Handle Paystack webhook events       |
| `GET`  | `/wallet/deposit/{reference}/status` | Check deposit transaction status     |
| `GET`  | `/wallet/balance`                    | Retrieve wallet balance              |
| `POST` | `/wallet/transfer`                   | Transfer funds to another wallet     |
| `GET`  | `/wallet/transactions`               | Retrieve wallet transaction history  |

---

## Authentication Flow

1. User requests the Google login URL.
2. User authenticates through Google.
3. Google redirects back with an authorization code.
4. The backend exchanges the code for Google user information.
5. A user record is created or retrieved.
6. A wallet is created for the user if one does not already exist.
7. A JWT access token is returned for authenticated API access.

---

## API Key Flow

1. Authenticated user creates an API key.
2. The API key is returned once in the format:

```txt
sk_live_<public_id>_<secret>
```

3. The secret is stored as a hash.
4. Future requests can authenticate with:

```http
x-api-key: sk_live_<public_id>_<secret>
```

5. API key permissions are checked before allowing protected wallet actions.

---

## Wallet Deposit Flow

1. Authenticated user or authorized API key sends a deposit request.
2. The backend creates a pending transaction record.
3. The backend initializes payment through Paystack.
4. Paystack returns an authorization URL.
5. The user completes payment through Paystack.
6. Paystack sends a webhook event to the backend.
7. The backend validates the webhook signature.
8. If payment is successful, the wallet balance is credited.
9. The transaction status is updated.
10. Duplicate successful webhook events are ignored to prevent double-crediting.

---

## Wallet Transfer Flow

1. Authenticated user or authorized API key sends a transfer request.
2. The backend checks the sender wallet.
3. The backend verifies the recipient wallet exists.
4. The backend checks sufficient balance.
5. Sender balance is debited.
6. Recipient balance is credited.
7. Transfer-out and transfer-in transaction records are created.

---

## Environment Variables

Create a `.env` file in the project root.

```env
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=http://localhost:8000/auth/google/callback

JWT_SECRET_KEY=your_jwt_secret_key
JWT_ALGORITHM=HS256
JWT_EXPIRES_MINUTES=60

DATABASE_URL=postgresql://username:password@localhost:5432/wallet_service

PAYSTACK_SECRET_KEY=your_paystack_secret_key
PAYSTACK_CALLBACK_URL=http://localhost:8000/wallet/paystack/webhook
```

Do not commit real secrets, API keys, database URLs, or OAuth credentials.

---

## Running Locally

### 1. Clone the repository

```bash
git clone https://github.com/Newkoncept/HNG-8-Wallet-Service.git
cd HNG-8-Wallet-Service
```

### 2. Create and activate a virtual environment

```bash
python -m venv venv
```

On Windows:

```bash
venv\Scripts\activate
```

On macOS/Linux:

```bash
source venv/bin/activate
```

### 3. Install dependencies with uv

```bash
pip install uv
uv sync
```

Alternative:

```bash
uv pip install .
```

### 4. Configure environment variables

Create a `.env` file using the environment variable guide above.

### 5. Run database migrations

```bash
alembic upgrade head
```

### 6. Start the development server

```bash
uvicorn main:app --reload
```

The API should be available at:

```txt
http://127.0.0.1:8000
```

Swagger documentation should be available at:

```txt
http://127.0.0.1:8000/docs
```

---

## Paystack Webhook Testing Locally

To test Paystack webhooks locally, expose the local FastAPI server using a tunneling tool such as `ngrok` or `cloudflared`.

Example using ngrok:

```bash
ngrok http 8000
```

Set the webhook URL in the Paystack dashboard:

```txt
https://<your-ngrok-subdomain>/wallet/paystack/webhook
```

---

## Example Deposit Workflow

1. Authenticate with Google.
2. Use the returned JWT token.
3. Call:

```http
POST /wallet/deposit
```

4. Visit the returned Paystack `authorization_url`.
5. Complete payment.
6. Paystack sends a webhook event to:

```http
POST /wallet/paystack/webhook
```

7. Check wallet balance:

```http
GET /wallet/balance
```

8. Check transaction status:

```http
GET /wallet/deposit/{reference}/status
```

---

## Security Notes

This project includes several backend security patterns:

* JWT-based authenticated access
* API key authentication through request headers
* API key secrets stored as hashes
* Permission-based API key access
* Paystack webhook signature validation using HMAC-SHA512
* Idempotent webhook handling to prevent duplicate wallet crediting
* Server-side transaction state updates

---

## Current Status

This project was built as part of the HNG Backend Internship Stage 8 wallet API task.

It demonstrates practical backend patterns around authentication, wallets, payments, API keys, webhooks, and transaction tracking.

Planned improvements include:

* Automated tests
* Cleaner route/module refactoring
* Expanded API documentation
* Postman collection
* Docker Compose setup
* Improved error response consistency
* More detailed transaction filtering
* Deployment guide

---

## Author

Built by [Oluwagbemiga Taiwo](https://github.com/Newkoncept)

LinkedIn: https://www.linkedin.com/in/hermmanuel
