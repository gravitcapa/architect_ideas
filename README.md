# Programming Languages / Environments

Main idea is serverless platform

- Node.js/React/Bootstrap/Templates for the front-end
- AWS S3 static website hosting
- Python/.Net 8 for the back-end 
- VS Code
- AWS S3 Blob Storage
- AWS Lambda
- AWS SNS/SQS messaging Queue
- worst case is a Postgress RDS, but this is more expensive then S3 and will require scaling up

# Sandbox/prod

each environment must have separate account in AWS
this is due to limits and naming convention in AWS
and better to have separate billing per environment
register each account using email Alias

- Sandbox: admin+sandbox@contoso.com
- Prod: admin+prod@contoso.com

registrations goes to admin@contoso.com


# Static Websites

AWS S3 (General Purpose) can host static websites as files, 
possible to host single page react apps on it
- can contain only static content
- easy to deploy from Git

# Static Markdown site content
Possible to make MD format reader and store static pages in the Git,
deploy on Check In, Render MD format on the site using viewer
this is good for documentation and APIs
possible to make folder structure similar to Confluence 

# Website Structure

Use React single page websites linked together by landing URLs

## Landing page

Generic landing page, visible to Public
Login

- contoso.com

## Admin App

Manages Settings and API keys, behind login
Should be similar to Shopify
Owner transfer

- admin.contoso.com

## Back-end App

Has and icons URLs to Functional Apps landing pages, behind login


- backend.contoso.com

## Micro Back-end Apps

Represents single functionality, 
for example product management, order management, inventory management
with a cross link to each other
Easy to fix, limited functionality

- app1.contoso.com
- app2.contoso.com

## Landing Pages

Using React Router land user on the specific page, 
other Apps they can use redirect URLs
May need to develop URL mapper, to prevent hardcoded URLs 
In this case will be easy to replace micro-apps with V2

- app1.contoso.com Root route
- app1.contoso.com/Product Product page route etc.

# Authorization

## User Authorization

for website users, general landing page should have login/create account
After the login, Login APIs produces the token
Token should be stored on the client side
Token can be used on other single page apps
if token expires, APIs return 401
Should be renew strategy implemented if client uses website, 
it can be done via Renew APIs running in the background periodically
If user IP address changes, login is required, all VPN clients work like this

### Token structure

- Encrypted by server using minimum AES 256 
- account ID
- Expiration date time
- client IP
- scope

## API Authentication

Admin APP should be able to produce API keys

### API keys

- Can be generated and Viewed one time only for the user
- if compromised or lost, user should only be able to delete the key
- Ideally Should not be stored on the Server side
- Server Side Should be able to validate the signature produced by user key

### Protocol

- HMAC preferred, user generates the request and signs it using secret, server side validates the signature
- Option 2: OAuth

# Backend

## APIs

AWS API Geteway

## Computation

AWS Lambda, HTTP Trigger, CRON Trigger, Event Triggers

## Queue

AWS SNS

## Storage

- AWS S3 General Purpose Buckets: Website hosting
- AWS S3 Directory Buckets: Object Storage
- AWS S3 Table Buckets: Search

### Object Storage Structure 

All data must be compressed using GZIP, it reduces storage cost 5 times

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

Directory buckets offers single digits ms access performance

### Object Absolute URLs

70% of all transactions is Read transactions and 30% Updates,
Application must be designed the way to avoid consume resources on Read operations

All objects must have AWS S3 CDN Path to the fast access to the Data by object ID

- Root -> Client -> Data -> Object ID -> data.json

### Resource Locking

In case of the competition for the same resource, writing service must create temp file

- Root -> Client -> Data -> Object ID -> object_temp.json 

if 2 processes simultaneously tried to create same resource 
only 1 will be able to create temp file, 
second will get the error and should return operation in the progress error

- create empty temp file temp.json (concurrent process cannot do it and will fail before processing)
- update temp file temp.json with new object data
- move/rename old file data.json to Archive folder  - Root -> Client -> Data -> Object ID -> Archive -> datetime_data.json
- rename temp.json to the current data.json
- Archive files should be removed after X days
- this approach makes minimum downtime, only for renaming/moving files
- version history in Archive folder for the audit

# IDs

all IDs should be unique and be able to be generated in the code
ideal is the date time stamp integer converted to HEX

```
24-12-10 23:52:14 9999ms -> 
202412102352149999 (18 digits) -> 
2CBD4C3C8F4F (12 digits)
```

- reduced (close to zero) risk of duplicate ID (if happens must be resolve via retry)
- does not disclose the previous ID directly
- clearly identifies object creation date
- makes object IDs sequentially generated
- readable

# Client Side Search

Explore ability for the client side search, based on local DB on the client Side

## Search Index Prep

On Object Save, Search index should be updated

get the list of unique words from required fields and HASH every word with short hashing algorithm MD5 SHA1
IDs and other required data also should be HASHed

### search_index.json
contains hash and qty matched

'''
2aae6c35c94fcfb415dbe95f408b9ce91ee846ed 7
addf120b430021c36c232c99ef8d926aea2acd6b 3
'''

## Search Index Object Data

Object folder should contain search index file, generated based on requirements

- Root -> Client -> Data -> Object ID -> search_index.json

## Global Index Builder

Background task, monitors changes in search_index.json files and updates the global search index files

Hashes with a lot of matches can be removed, they usually indicates (AND, OR, THEN, IN)

Optimizations can be applied

Search data should be GZIPed in batches, for example month or year

## Search index batches

Load batches from CDN, latest first

- 2024_search_index.json
- 2023_search_index.json

## Client App search

Then the mini app starts it updates local search index in the background
starting from the latest index file first
Client searches in the local DB, 

## Server Side Search
if local search is not available, 
should be called server side API for the search
Server side API should load search index in the memory cache and search within
Server side can also use unhashed search index files

### S3 Data Tables

S3 has support of SQL Data Tables, what can be used in the server side searches 
or process automation, it can use SQL to access the data
  






   
