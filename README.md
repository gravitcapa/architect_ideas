# Programming Languages / Environments

## Main Idea: Serverless Platform

- **Front-end**:
  - Node.js, React, Bootstrap, and Templates
  - AWS S3 for static website hosting
- **Back-end**:
  - Python or .NET 8
  - AWS S3 Blob Storage for object hosting
  - AWS Lambda for serverless computation
  - AWS SNS/SQS for messaging queues
  - Optional: PostgreSQL RDS (only if necessary, as it is more expensive than S3 and requires scaling)
- **Tools**:
  - Visual Studio Code

---

## Sandbox/Prod Environments

Each environment must use a separate AWS account for:

1. **AWS Limits** and naming convention isolation
2. **Billing separation**

### Email Registration

- Use email aliases for each account:
  - Sandbox: `admin+sandbox@contoso.com`
  - Prod: `admin+prod@contoso.com`
- All registrations forward to: `admin@contoso.com`

---

## Static Websites

### Hosting

- AWS Route 53 for the domain name 
- **AWS S3 (General Purpose)** can host static websites, including single-page React apps.
- Supports:
  - Static content only
  - Easy deployment from Git

### Static Markdown Site Content

- Store site content in Markdown (`.md`) format in Git.
- Deploy on Git check-in.
- Render Markdown dynamically on the site using a viewer.
- Suggested folder structure:
  - Mimic Confluence-like hierarchy for documentation and APIs.

---

## Website Structure

### React Single Page Websites

- Link pages using React Router.
- Utilize landing URLs for navigation.

### Pages

#### Landing Page

- **Public-facing**:
  - Accessible to all users.
  - Includes login functionality.
  - Example: `contoso.com`

#### Admin App

- **Purpose**: Manage settings and API keys.
- **Features**:
  - Behind login
  - Owner transfer functionality
- **Example**: `admin.contoso.com`

#### Back-end App

- **Purpose**: Provide icons and URLs to list of available apps landing pages.
- **Access**: Behind login
- **Example**: `backend.contoso.com`

#### Micro Back-end Apps

- Represent single functionality, such as:
  - Product management
  - Order management
  - Inventory management
- **Examples**:
  - `app1.contoso.com`
  - `app2.contoso.com`

### Landing Page Navigation

- Use React Router for specific routes, e.g.,:
  - `app1.contoso.com/` (Root route)
  - `app1.contoso.com/Product` (Product page route)
- Optionally develop a URL mapper to prevent hardcoded routes, facilitating future upgrades.

---

## Authorization

### User Authorization

#### Workflow

- General landing page:
  - Supports login and account creation.
  - Login APIs return a token stored on the client side.
  - Token usage:
    - Valid across all single-page apps.
    - Expired tokens return `401` responses.
    - Implement token renewal via background APIs.
- If the user’s IP address changes, force re-login (similar to VPN client behavior).

#### Token Structure

- Encrypted using AES-256.
- Includes:
  - Account ID
  - Expiration datetime
  - Client IP
  - Scope

### API Authentication

#### API Keys

- **Admin App**: Generate and manage API keys.
- **Key Properties**:
  - Viewable once upon generation.
  - Lost or compromised keys must be deleted by users.
  - Should not be stored on the server.
- **Server Validation**: Validate signatures generated by user keys.

#### Protocols

1. **Preferred**: HMAC
   - User signs the request with their secret key.
   - Server validates the signature.
2. **Alternative**: OAuth

---

## Back-end

### APIs

- Use AWS API Gateway for managing endpoints.

### Computation

- AWS Lambda:
  - HTTP triggers
  - CRON triggers
  - Event triggers

### Queue

- AWS SNS or SQS for messaging queues.

### Storage

- **AWS S3**:
  - General Purpose Buckets: Website hosting
  - Directory Buckets: Object storage
  - Table Buckets: Search data

#### Object Storage Structure

- **Data Compression**: Use GZIP for storage cost reduction (up to 5x).
- Directory layout:
  ```
  Root
  ├── Settings
  ├── Clients
      ├── Client
          ├── Settings
          ├── Data
              ├── Object ID
                  ├── data.json
                  ├── search_index.json
                  ├── state_machine.json
  ```

#### Resource Locking

- To handle simultaneous writes:
  1. Create a temporary file (`temp.json`).
     - If two processes simultaneously attempt to create the same resource, only one process will succeed in creating the temp file. The second process will receive an error indicating "operation is in progress."
  2. Update the temp file with new data.
  3. Archive the old file:
     - Move `data.json` to `Archive/datetime_data.json`.
  4. Rename `temp.json` to `data.json`.
- Archive files should have retention policies (e.g., auto-delete after X days).

---

## IDs

### Format

- **Date-time based unique IDs**:
  - Example: `202412102352149999` (18 digits) → `2CBD4C3C8F4F` (12 digits in HEX)

#### Benefits

- Near-zero risk of duplication.
- Encodes creation date.
- Sequentially generated and readable.

---

## Client-Side Search

### Search Index Preparation

- On object save:
  1. Extract unique words from required fields.
  2. Hash each word (e.g., MD5 or SHA1).
- Example `search_index.json`:
  ```
  2aae6c35c94fcfb415dbe95f408b9ce91ee846ed: 7
  addf120b430021c36c232c99ef8d926aea2acd6b: 3
  ```

### Global Index Builder

- Background task updates the global search index based on changes in `search_index.json` files.
- Optimization:
  - Remove hashes for common terms (e.g., AND, OR).
  - Compress search data in GZIP batches (e.g., monthly).

### Search Index Batches

- Load in reverse chronological order:
  - `2024_search_index.json`
  - `2023_search_index.json`

### Client Search

- Update local search index in the background.
- Perform searches locally using the updated index.

### Server-Side Search

- If local search is unavailable, query server-side APIs.
- Server-side search can use SQL data tables supported by S3.

---

