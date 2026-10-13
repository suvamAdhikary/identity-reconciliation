# Identity Reconciliation Service

[![CI](https://github.com/suvamAdhikary/identity-reconciliation/actions/workflows/ci.yml/badge.svg)](https://github.com/suvamAdhikary/identity-reconciliation/actions/workflows/ci.yml)

A NestJS-based backend service for reconciling and linking customer identities across multiple contact points (email, phone).

## Overview

This service implements an identity reconciliation system that consolidates customer information from various sources. When a customer interacts through different channels (email, phone, web forms), the system intelligently links these interactions to maintain a unified customer profile.

## Data Model

The system uses a contact entity structure with the following relationships:

### Contact Entity
- **Primary Contact**: The root contact representing a unique customer identity
- **Secondary Contact**: Linked contacts that share the same underlying customer identity

### Key Fields
- `email`: Customer's email address (nullable, indexed)
- `phoneNumber`: Customer's phone number (nullable, indexed)
- `linkedId`: Foreign key pointing to the primary contact (null for primary contacts)
- `linkPrecedence`: Enum value - "primary" or "secondary"
- `createdAt`, `updatedAt`, `deletedAt`: Standard audit timestamps

### Reconciliation Logic

The service handles several edge cases when identifying and linking contacts:

1. **Exact Match**: When both email and phone match an existing contact, return the existing identity
2. **Partial Match**: When either email or phone matches, link to the existing primary contact
3. **Multiple Primaries**: When email matches one primary and phone matches another, merge the identities by:
   - Converting the newer primary to secondary
   - Linking it to the older primary
   - Cascading all secondary contacts of the newer primary to the older primary
4. **No Match**: Create a new primary contact when neither email nor phone exist in the system
5. **Null Handling**: Gracefully handle null values in email or phone fields without creating false links

### Data Integrity Constraints
- At least one of email or phoneNumber must be provided
- Composite unique index on (email, phoneNumber) pairs
- Soft delete support via deletedAt timestamp

## API Endpoints

### POST `/identify`

Identify and reconcile customer contacts.

**Request Body:**
```json
{
  "email": "customer@example.com",
  "phoneNumber": "1234567890"
}
```

**Response:**
```json
{
  "contact": {
    "primaryContactId": 1,
    "emails": ["customer@example.com", "alternate@example.com"],
    "phoneNumbers": ["1234567890", "0987654321"],
    "secondaryContactIds": [2, 3]
  }
}
```

**Validation Rules:**
- At least one of `email` or `phoneNumber` must be provided
- Email must be valid email format if provided
- Phone number must be numeric string if provided

## Local Development

### Prerequisites
- Node.js 20.x or higher
- PostgreSQL 14.x or higher
- npm 9.x or higher

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/suvamAdhikary/identity-reconciliation.git
   cd identity-reconciliation
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure environment variables**
   
   Create a `.env` file in the project root:
   ```env
   DATABASE_HOST=localhost
   DATABASE_PORT=5432
   DATABASE_USER=postgres
   DATABASE_PASSWORD=yourpassword
   DATABASE_NAME=identity_reconciliation
   NODE_ENV=development
   PORT=3000
   ```

4. **Setup PostgreSQL database**
   ```bash
   # Using psql
   psql -U postgres
   CREATE DATABASE identity_reconciliation;
   \q
   ```

5. **Run database migrations**
   ```bash
   npm run migration:run
   ```

### Running the Application

**Development mode with hot reload:**
```bash
npm run start:dev
```

**Production mode:**
```bash
npm run build
npm run start:prod
```

The API will be available at `http://localhost:3000`

### Testing

**Run all unit tests:**
```bash
npm test
```

**Run tests in watch mode:**
```bash
npm run test:watch
```

**Run end-to-end tests:**
```bash
npm run test:e2e
```

**Generate test coverage report:**
```bash
npm run test:cov
```

### Code Quality

**Run linter:**
```bash
npm run lint
```

**Format code:**
```bash
npm run format
```

## Project Structure

```
identity-reconciliation/
├── src/
│   ├── app.controller.ts       # Main application controller
│   ├── app.controller.spec.ts  # Controller unit tests
│   ├── app.module.ts           # Root application module
│   ├── app.service.ts          # Core business logic
│   └── main.ts                 # Application entry point
├── test/
│   ├── app.e2e-spec.ts         # End-to-end tests
│   └── jest-e2e.json           # E2E test configuration
├── .github/
│   └── workflows/
│       └── ci.yml              # GitHub Actions CI pipeline
├── ROADMAP.md                  # Project roadmap and decisions
└── package.json                # Project dependencies and scripts
```

## Technology Stack

- **Framework**: NestJS 10.x
- **Language**: TypeScript 5.x
- **Database**: PostgreSQL with TypeORM
- **Testing**: Jest + Supertest
- **Linting**: ESLint + Prettier
- **CI/CD**: GitHub Actions

## License

This project is licensed under the UNLICENSED license - see the LICENSE file for details.

## Author

[Suvam Adhikary](https://github.com/suvamAdhikary)
