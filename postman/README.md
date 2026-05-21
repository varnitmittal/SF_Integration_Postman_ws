# INTR BASICS Postman Collection

## Overview

This Postman collection provides comprehensive examples and documentation for integrating with Salesforce through both **REST** and **SOAP** APIs. The collection includes out-of-the-box (OotB) Salesforce endpoints and custom implementations for training and reference purposes.

---

## Table of Contents

- [Authentication](#authentication)
- [Collection Structure](#collection-structure)
- [REST API](#rest-api)
  - [Query Endpoint](#query-endpoint)
  - [SObject Endpoint](#sobject-endpoint)
  - [Actions Endpoint](#actions-endpoint)
  - [Composite Endpoint](#composite-endpoint)
  - [Limits Endpoint](#limits-endpoint)
  - [UI-API Endpoint](#ui-api-endpoint)
- [SOAP API](#soap-api)
- [Variables & Setup](#variables--setup)

---

## Authentication

### REST API Authentication

All REST API requests use **Bearer Token Authentication**:

```http
Authorization: Bearer {{MyBearerToken}}
Content-Type: application/json
```

**Variable Used:** `{{MyBearerToken}}`

Ensure this variable is populated with a valid Salesforce OAuth access token before making requests.

---

## Collection Structure

```
INTR BASICS/
├── REST/
│   ├── REST Auth.request.yaml
│   ├── Custom/
│   │   └── StudentRESTService/
│   │       └── StudentRESTService-GET.request.yaml
│   └── OotB/ (Out-of-the-Box Salesforce Endpoints)
│       ├── Query ep/
│       ├── SObject ep/
│       ├── Actions ep/
│       ├── Composite ep/
│       ├── Limits ep/
│       └── UI-API ep/
└── SOAP/
    ├── Soap Auth.request.yaml
    ├── Class/
    │   └── StudentSOAPService/
    │       ├── StudentSOAPService-getStudentByStudentId.request.yaml
    │       └── StudentSOAPService-getStudents.request.yaml
    ├── DMLs/
    │   ├── Insert new Student-1.request.yaml
    │   └── Insert new Student.request.yaml
    └── Queries/
        ├── SOAP Account and related Contacts Query.request.yaml
        ├── SOAP Query Latest Inserted Student.request.yaml
        └── SOAP School and all students.request.yaml
```

---

## REST API

### Query Endpoint

**API Path:** `/services/data/v64.0/query/`

The Query endpoint executes SOQL (Salesforce Object Query Language) queries against your Salesforce org through the REST API.

#### Base Endpoint

```http
GET /services/data/v64.0/query?q=<URL_ENCODED_SOQL>
```

You append URL-encoded SOQL after `q=` parameter.

#### Basic Example

```http
GET /services/data/v64.0/query?q=SELECT+Id,Name+FROM+Student__c
```

#### Processing Flow

| Step | Action      | Description                        |
| ---- | ----------- | ---------------------------------- |
| 1    | **Parse**   | Reads and validates SOQL query     |
| 2    | **Execute** | Executes query against database    |
| 3    | **Return**  | Returns JSON response with results |

#### Example Response

```json
{
  "totalSize": 2,
  "done": true,
  "records": [
    {
      "Id": "a1hXXXXXXXXXXXX",
      "Name": "St-0001"
    }
  ]
}
```

#### Important Response Fields

| Field            | Meaning                | Example                  |
| ---------------- | ---------------------- | ------------------------ |
| `totalSize`      | Total matching records | `2`                      |
| `done`           | Pagination complete?   | `true`                   |
| `records`        | Returned data rows     | `[{...}, {...}]`         |
| `nextRecordsUrl` | URL for next page      | `/query/01gXXXXXXXXXXXX` |

#### Query Clause Examples

| Clause       | Purpose            | Example                                                         |
| ------------ | ------------------ | --------------------------------------------------------------- |
| **LIMIT**    | Limit result count | `SELECT Id,Name FROM Student__c LIMIT 5`                        |
| **WHERE**    | Filter conditions  | `SELECT Id,Name FROM Student__c WHERE Name='St-0001'`           |
| **ORDER BY** | Sort results       | `SELECT Id,Name FROM Student__c ORDER BY CreatedDate DESC`      |
| **LIKE**     | Pattern matching   | `SELECT Id,Name FROM Student__c WHERE Name LIKE 'St%'`          |
| **COUNT()**  | Aggregate count    | `SELECT COUNT() FROM Student__c`                                |
| **GROUP BY** | Group aggregates   | `SELECT School__c,COUNT(Id) FROM Student__c GROUP BY School__c` |

##### LIMIT Clause Example

```http
GET /services/data/v64.0/query?q=SELECT+Id,Name+FROM+Student__c+LIMIT+5
```

##### WHERE Clause Example

```http
GET /services/data/v64.0/query?q=SELECT+Id,Name+FROM+Student__c+WHERE+Name='St-0001'
```

##### ORDER BY Clause Example

```http
GET /services/data/v64.0/query?q=SELECT+Id,Name+FROM+Student__c+ORDER+BY+CreatedDate+DESC
```

##### LIKE Operator Example

```http
GET /services/data/v64.0/query?q=SELECT+Id,Name+FROM+Student__c+WHERE+Name+LIKE+'St%'
```

##### COUNT() Function Example

```http
GET /services/data/v64.0/query?q=SELECT+COUNT()+FROM+Student__c
```

##### Aggregate Query Example

```http
GET /services/data/v64.0/query?q=SELECT+School__c,COUNT(Id)+studentCount+FROM+Student__c+GROUP+BY+School__c
```

#### Pagination

For large result sets, use the `Sforce-Query-Options` header to batch results:

**Pagination Headers:**

| Header                 | Value          | Purpose                   |
| ---------------------- | -------------- | ------------------------- |
| `Sforce-Query-Options` | `batchSize=10` | Sets max records per page |

**Step 1 - Initial Request:**

```http
GET /services/data/v64.0/query?q=SELECT+Id,Name+FROM+Student__c
Header: Sforce-Query-Options: batchSize=10
```

**Step 1 Response:**

```json
{
  "done": false,
  "totalSize": 25,
  "nextRecordsUrl": "/services/data/v64.0/query/01gXXXXXXXXXXXX"
}
```

**Step 2 - Fetch Next Page:**

```http
GET /services/data/v64.0/query/01gXXXXXXXXXXXX
```

#### URL Encoding Reference

SOQL queries must be URL encoded for the query parameter.

| Character(s) | Encoded As     | Example                               |
| ------------ | -------------- | ------------------------------------- |
| Space        | `+` (or `%20`) | `SELECT Id, Name` → `SELECT+Id,+Name` |
| Comma-space  | `+`            | `Id, Name` → `Id,+Name`               |
| Single quote | `'`            | Stay as-is in LIKE clauses            |

**Before Encoding:**

```sql
SELECT Id, Name FROM Student__c WHERE Name LIKE 'St%'
```

**After Encoding:**

```
SELECT+Id,+Name+FROM+Student__c+WHERE+Name+LIKE+'St%'
```

#### Endpoint Comparison

| Endpoint     | Purpose               | Operation                | Ideal For                             |
| ------------ | --------------------- | ------------------------ | ------------------------------------- |
| `/query/`    | Data retrieval engine | SELECT queries only      | Reporting, sync, filtering, bulk read |
| `/sobjects/` | Record CRUD           | GET, POST, PATCH, DELETE | Create/update/delete records          |

Most enterprise integrations heavily utilize Query APIs for data-intensive operations.

---

### SObject Endpoint

**API Path:** `/services/data/v64.0/sobjects/`

The SObject endpoint provides comprehensive access to work with Salesforce objects (sObjects) through REST API. This includes Salesforce's standard objects and custom objects.

#### Base Format

```
https://YOUR_DOMAIN.my.salesforce.com/services/data/v64.0/sobjects/
```

#### Supported Object Types

| Object Category      | Examples                                               | Custom? |
| -------------------- | ------------------------------------------------------ | ------- |
| **Standard Objects** | Account, Contact, Lead, Opportunity, Case, Opportunity | No      |
| **Custom Objects**   | Student**c, School**c, And others in your org          | Yes     |

#### Authentication

```http
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

#### CRUD Operations Summary

| Operation    | HTTP Verb | Purpose                | Request Body                |
| ------------ | --------- | ---------------------- | --------------------------- |
| **Create**   | POST      | Insert new record      | Required (field values)     |
| **Read**     | GET       | Fetch single record    | None                        |
| **List**     | GET       | Get all objects        | None                        |
| **Update**   | PATCH     | Modify existing record | Optional (fields to update) |
| **Delete**   | DELETE    | Remove record          | None                        |
| **Describe** | GET       | Get object metadata    | None                        |

---

#### 1. Get All Available Objects

Retrieve list of all sObjects in your Salesforce org:

**Request:**

```http
GET /services/data/v64.0/sobjects/
```

**cURL Example:**

```bash
curl -X GET "https://yourdomain.my.salesforce.com/services/data/v64.0/sobjects/" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

**Response:**

```json
{
  "sobjects": [
    {
      "name": "Account",
      "label": "Account",
      "updateable": true,
      "createable": true,
      "deletable": true
    },
    {
      "name": "Student__c",
      "label": "Student",
      "updateable": true,
      "createable": true,
      "deletable": true
    }
  ]
}
```

---

#### 2. Describe Object Metadata

Get comprehensive metadata about a specific object:

**Request:**

```http
GET /services/data/v64.0/sobjects/Student__c/describe
```

**Returns:**

| Item                 | Details                                  |
| -------------------- | ---------------------------------------- |
| **Fields**           | Name, label, type, length, required      |
| **Relationships**    | Parent/child relationships               |
| **Picklists**        | Valid values for picklist fields         |
| **Record Types**     | Available record types                   |
| **Permissions**      | Create, read, update, delete permissions |
| **Validation Rules** | Field-level validations                  |

**Example Response (Partial):**

```json
{
  "name": "Student__c",
  "label": "Student",
  "fields": [
    {
      "name": "Student_Name__c",
      "label": "Student Name",
      "type": "string",
      "length": 255,
      "required": false
    }
  ]
}
```

---

#### 3. Get Record by ID

Fetch a specific record:

**Request Pattern:**

```http
GET /services/data/v64.0/sobjects/{sObjectName}/{recordId}
```

**Example:**

```http
GET /services/data/v64.0/sobjects/Student__c/a1hgK000002LFR7QAO
```

**cURL:**

```bash
curl -X GET "https://yourdomain.my.salesforce.com/services/data/v64.0/sobjects/Student__c/a1hgK000002LFR7QAO" \
  -H "Authorization: Bearer ACCESS_TOKEN"
```

**Response:**

```json
{
  "Id": "a1hgK000002LFR7QAO",
  "Name": "St-0001",
  "School__c": "a1gXXXXXXXXXXXX",
  "CreatedDate": "2026-05-15T10:30:00.000+0000"
}
```

---

#### 4. Create Record

Create a new record in an object:

**Request Pattern:**

```http
POST /services/data/v64.0/sobjects/{sObjectName}/
```

**Example Request:**

```http
POST /services/data/v64.0/sobjects/Student__c/
Content-Type: application/json
Authorization: Bearer ACCESS_TOKEN

{
  "Name": "New Student",
  "School__c": "a1gXXXXXXXXXXXX"
}
```

**Response:**

```json
{
  "id": "a1hgK000002LFR7QAO",
  "success": true,
  "created": true
}
```

---

#### 5. Update Record

Modify an existing record (partial update):

**Request Pattern:**

```http
PATCH /services/data/v64.0/sobjects/{sObjectName}/{recordId}
```

**Example Request:**

```http
PATCH /services/data/v64.0/sobjects/Student__c/a1hgK000002LFR7QAO
Content-Type: application/json
Authorization: Bearer ACCESS_TOKEN

{
  "Name": "Updated Student Name"
}
```

**Response:**

```json
{
  "id": "a1hgK000002LFR7QAO",
  "success": true,
  "created": false
}
```

**Note:** Only include fields you want to update; other fields remain unchanged.

---

#### 6. Delete Record

Remove a record from an object:

**Request Pattern:**

```http
DELETE /services/data/v64.0/sobjects/{sObjectName}/{recordId}
```

**Example:**

```http
DELETE /services/data/v64.0/sobjects/Student__c/a1hgK000002LFR7QAO
Authorization: Bearer ACCESS_TOKEN
```

**Response:**

```json
{
  "id": "a1hgK000002LFR7QAO",
  "success": true
}
```

---

### Actions Endpoint

**Focus:** Invocable Apex, Flow Integration, and Actions API

This endpoint demonstrates how to execute Apex actions and orchestrate Salesforce Flows through REST API, enabling low-code + pro-code integration patterns.

#### Core Decorators

| Decorator            | Type           | Purpose                                                      | Example Attribute                  |
| -------------------- | -------------- | ------------------------------------------------------------ | ---------------------------------- |
| `@InvocableMethod`   | Class Method   | Makes Apex callable from Flows, Actions API, Process Builder | `label='Generate Welcome Message'` |
| `@InvocableVariable` | Class Property | Exposes variable to Flows as input/output                    | `required=true`                    |

#### @InvocableMethod Decorator

Makes an Apex method callable from:

- ✓ Salesforce Flows
- ✓ Actions API (REST)
- ✓ Process Builder
- ✓ Orchestration tools
- ✓ External systems (via Flow)

**Characteristics:**

| Attribute        | Typical Value                | Meaning                       |
| ---------------- | ---------------------------- | ----------------------------- |
| `label`          | `'Generate Welcome Message'` | Display name in Flow UI       |
| Return type      | `List<T>`                    | Must return a List            |
| Method parameter | `List<InputClass>`           | Accepts List of input objects |

#### @InvocableVariable Decorator

Exposes class properties as Flow inputs/outputs:

```apex
@InvocableVariable(required=true)
public String studentName;

@InvocableVariable(required=false)
public String email;
```

**Characteristics:**

| Attribute     | Options        | Default       | Purpose                |
| ------------- | -------------- | ------------- | ---------------------- |
| `required`    | `true`/`false` | `false`       | Field must be provided |
| `label`       | String         | Property name | Display label in Flow  |
| `description` | String         | -             | Help text in Flow UI   |

#### Example: StudentWelcomeService

**Apex Implementation:**

```apex
public with sharing class StudentWelcomeService {
    public class Request {
        @InvocableVariable(required=true, label='Student Name')
        public String studentName;
    }

    @InvocableMethod(label='Generate Welcome Message')
    public static List<String> generateMessage(List<Request> requests) {
        List<String> responses = new List<String>();
        for(Request req : requests) {
            responses.add(
                'Welcome ' + req.studentName + ' to Salesforce REST APIs!'
            );
        }
        return responses;
    }
}
```

---

#### Flow Configuration

**Flow Setup Table:**

| Property       | Value                   | Description           |
| -------------- | ----------------------- | --------------------- |
| Flow Type      | `Autolaunched Flow`     | REST-callable (no UI) |
| API Name       | `StudentWelcomeService` | Used in API calls     |
| Version Status | `Active`                | Must be activated     |

**Flow Input Variables:**

| Variable         | Type | Required | Purpose                          |
| ---------------- | ---- | -------- | -------------------------------- |
| `studentNameVar` | Text | Yes      | Receives input from REST request |

**Apex Action Mapping:**

| Flow Variable       | →       | Apex Parameter                 |
| ------------------- | ------- | ------------------------------ |
| `{!studentNameVar}` | Maps to | `studentName` in Request class |

**Flow Output Variables:**

| Variable     | Type | Source      | Purpose                   |
| ------------ | ---- | ----------- | ------------------------- |
| `resultText` | Text | Apex return | Returns generated message |

**Processing Flow:**

```
REST Request
    ↓ (studentNameVar)
Flow Input Variable
    ↓
Apex Action (@InvocableMethod)
    ↓
Generate Welcome Message
    ↓ (resultText)
Flow Output Variable
    ↓
REST Response
```

---

#### REST API Call

**Request:**

```http
POST /services/data/v64.0/actions/custom/flow/StudentWelcomeService
Authorization: Bearer {{MyBearerToken}}
Content-Type: application/json
```

**Request Body:**

```json
{
  "inputs": [
    {
      "studentNameVar": "John Smith"
    }
  ]
}
```

**Example Response:**

```json
{
  "results": [
    {
      "resultText": "Welcome John Smith to Salesforce REST APIs!"
    }
  ]
}
```

---

#### Architecture Benefits

| Benefit             | Description                              | Use Case                            |
| ------------------- | ---------------------------------------- | ----------------------------------- |
| **Low-code**        | Build Flows without writing Apex         | Simple data transformations         |
| **Pro-code Power**  | Leverage Apex for complex logic          | Business calculations, integrations |
| **REST Accessible** | Call from external systems via HTTP      | Integration with third-party apps   |
| **Reusable**        | Single Flow/Apex serves multiple clients | Build once, use everywhere          |
| **Error Handling**  | Flow error management built-in           | Robust orchestration                |

#### When to Use Actions Endpoint

✓ Need to call custom business logic from external systems  
✓ Want to combine Flow orchestration with Apex power  
✓ Building integration APIs  
✓ Need stateless, REST-callable operations

---

### Composite Endpoint

**API Path:** `/services/data/v64.0/composite/`

The Composite API allows you to execute **multiple Salesforce REST operations in a single HTTP request**, reducing API call overhead and enabling powerful orchestration patterns.

#### Composite API Types Comparison

| API Type    | Endpoint                    | Purpose                                       | Dependencies            | Request Limit |
| ----------- | --------------------------- | --------------------------------------------- | ----------------------- | ------------- |
| **Batch**   | `/composite/batch`          | Execute independent requests in parallel      | None                    | 25 requests   |
| **Chained** | `/composite`                | Execute sequential requests with data sharing | Yes - via `referenceId` | 25 requests   |
| **Tree**    | `/composite/tree/{sObject}` | Insert parent-child hierarchies               | Yes - record structure  | 200 records   |

#### Batch API Use Cases

✓ Multiple independent data retrievals  
✓ Parallel queries and lookups  
✓ Batch processing unrelated operations  
✓ When request order doesn't matter

#### Chained API Use Cases

✓ Create related records (Account → Contact)  
✓ Data dependencies between requests  
✓ Conditional logic based on previous results  
✓ Sequential workflows

#### Tree API Use Cases

✓ Inserting account with contacts and opportunities  
✓ Building object hierarchies  
✓ Bulk inserting parent-child records

---

#### 1. Composite Batch API

**Endpoint:**

```http
POST /services/data/v64.0/composite/batch
```

**Purpose:** Execute multiple unrelated requests in parallel.

**Batch Characteristics:**

| Aspect               | Detail                                          |
| -------------------- | ----------------------------------------------- |
| **Execution**        | Parallel (all at once)                          |
| **Data Sharing**     | Not supported                                   |
| **Order**            | Doesn't matter                                  |
| **Failure Handling** | Independent (one failure doesn't affect others) |
| **Max Requests**     | 25 per batch call                               |

**Example Request:**

```json
{
  "batchRequests": [
    {
      "method": "GET",
      "url": "v64.0/limits"
    },
    {
      "method": "GET",
      "url": "v64.0/query?q=SELECT+Id+FROM+Account"
    }
  ]
}
```

**Request Structure:**

| Field           | Type   | Required | Description                   |
| --------------- | ------ | -------- | ----------------------------- |
| `batchRequests` | Array  | Yes      | Array of request objects      |
| `method`        | String | Yes      | HTTP method (GET, POST, etc.) |
| `url`           | String | Yes      | Relative API URL              |
| `richInput`     | Object | No       | Request body for POST/PATCH   |

**Example Response:**

```json
{
  "hasErrors": false,
  "results": [
    {
      "statusCode": 200,
      "result": {
        "organizationId": "00D50000000IZ3Z"
      }
    },
    {
      "statusCode": 200,
      "result": {
        "totalSize": 5,
        "done": true,
        "records": []
      }
    }
  ]
}
```

---

#### 2. Composite (Chained Requests)

**Endpoint:**

```http
POST /services/data/v64.0/composite
```

**Purpose:** Execute requests sequentially with ability to reference results from previous requests.

**Chained Characteristics:**

| Aspect               | Detail                                |
| -------------------- | ------------------------------------- |
| **Execution**        | Sequential (one after another)        |
| **Data Sharing**     | Yes - via `referenceId`               |
| **Order**            | Matters (depends on previous results) |
| **Failure Handling** | `allOrNone` controls rollback         |
| **Max Requests**     | 25 per call                           |

**Reference Syntax:**

To use output from a previous request, use this pattern:

```
@{referenceId.propertyPath}
```

**Example Usage:**

```
@{newAccount.id}        → Get ID from newAccount
@{queryResult.records}  → Get records array
```

**Request Structure:**

| Field              | Type    | Required    | Description                       |
| ------------------ | ------- | ----------- | --------------------------------- |
| `allOrNone`        | Boolean | No          | If true = atomic (all or nothing) |
| `compositeRequest` | Array   | Yes         | Array of request objects          |
| `referenceId`      | String  | Yes         | Unique ID for this request        |
| `method`           | String  | Yes         | HTTP method                       |
| `url`              | String  | Yes         | Relative API URL                  |
| `body`             | Object  | Conditional | POST/PATCH bodies                 |

**Example Request: Create Account, Then Contact**

```json
{
  "allOrNone": false,
  "compositeRequest": [
    {
      "method": "POST",
      "url": "/services/data/v64.0/sobjects/Account",
      "referenceId": "newAccount",
      "body": {
        "Name": "Acme Corp",
        "BillingCity": "San Francisco"
      }
    },
    {
      "method": "POST",
      "url": "/services/data/v64.0/sobjects/Contact",
      "referenceId": "newContact",
      "body": {
        "LastName": "Smith",
        "FirstName": "John",
        "AccountId": "@{newAccount.id}"
      }
    }
  ]
}
```

**Processing Flow:**

```
Step 1: Create Account
  ↓ Returns ID: a1hXXXXXXXXXXXX
Step 2: Create Contact
  ↓ Uses @{newAccount.id} = a1hXXXXXXXXXXXX
  ↓ Links Contact to Account
Step 3: Return combined results
```

**allOrNone Parameter:**

| Value   | Behavior                                          | Use Case                       |
| ------- | ------------------------------------------------- | ------------------------------ |
| `true`  | Atomic - if any fails, all rollback               | Critical multi-step operations |
| `false` | Independent - failed requests don't affect others | Allowed partial success        |

---

#### 3. Composite Tree API

**Endpoint:**

```http
POST /services/data/v64.0/composite/tree/{sObjectName}
```

**Purpose:** Insert parent-child hierarchical records together efficiently.

**Tree Characteristics:**

| Aspect           | Detail                                   |
| ---------------- | ---------------------------------------- |
| **Execution**    | Single hierarchical insert               |
| **Structure**    | Parent-child parent-child hierarchy      |
| **Record Limit** | 200 records per call                     |
| **Optimization** | Efficient bulk insert of related records |

**Example Structure:**

```
Account (Parent)
├── Contact 1
├── Contact 2
└── Opportunity
```

**Request Pattern:**

```http
POST /services/data/v64.0/composite/tree/Account
Content-Type: application/json
Body: { records: [...] }
```

**Typical Use Case:**

Creating an Account with multiple Contacts and Opportunities in one call instead of 3+ separate API calls.

---

### Limits Endpoint

**API Path:** `/services/data/v64.0/limits/`

Monitor your Salesforce organization's API usage limits and resource consumption.

---

### UI-API Endpoint

**API Path:** `/services/data/ui-api/`

Access UI-specific data and configuration through the UI API for advanced frontend integrations.

---

## SOAP API

The SOAP API provides traditional XML-based web service integration with Salesforce.

### SOAP vs REST Comparison

| Aspect             | SOAP                                   | REST                               |
| ------------------ | -------------------------------------- | ---------------------------------- |
| **Protocol**       | XML-based web services                 | HTTP (REST principles)             |
| **Format**         | XML envelope                           | JSON typically                     |
| **Complexity**     | Higher (WSDL, namespaces)              | Lower (HTTP methods, URLs)         |
| **Best For**       | Enterprise integrations, complex logic | Modern apps, mobile, microservices |
| **Speed**          | Slower (XML parsing overhead)          | Faster (lighter protocol)          |
| **Learning Curve** | Steeper                                | Gentler                            |

### Collection Structure

**Custom SOAP Services:**

| Path                             | Purpose                 | Methods                                |
| -------------------------------- | ----------------------- | -------------------------------------- |
| `SOAP/Class/StudentSOAPService/` | Custom SOAP web service | `getStudentByStudentId`, `getStudents` |

**DML Operations:**

| Request                             | Operation | Purpose                    |
| ----------------------------------- | --------- | -------------------------- |
| `Insert new Student-1.request.yaml` | INSERT    | Create new Student records |
| `Insert new Student.request.yaml`   | INSERT    | Alternative insert example |

**Query Operations:**

| Request                                   | Description                                  |
| ----------------------------------------- | -------------------------------------------- |
| `SOAP Account and related Contacts Query` | Retrieve Account and related Contact records |
| `SOAP Query Latest Inserted Student`      | Query most recently inserted Student         |
| `SOAP School and all students`            | Retrieve School with all related Students    |

### Learning Path

| Step | Action            | Request                              | Purpose                  |
| ---- | ----------------- | ------------------------------------ | ------------------------ |
| 1    | **Authenticate**  | `Soap Auth.request.yaml`             | Obtain session ID        |
| 2    | **Test Query**    | `SOAP Query Latest Inserted Student` | Verify connection        |
| 3    | **Create Record** | `Insert new Student.request.yaml`    | Understand DML           |
| 4    | **Query Related** | `SOAP School and all students`       | Practice complex queries |
| 5    | **Custom Apex**   | `StudentSOAPService-getStudents`     | Call custom logic        |

### Authentication

SOAP API authentication methods:

| Method         | Credentials                      | Session Validity | Use Case                |
| -------------- | -------------------------------- | ---------------- | ----------------------- |
| **Session ID** | Username + Password → Session ID | Limited (hours)  | Postman testing         |
| **OAuth**      | Access Token                     | Configurable     | Production integrations |

**Initial Setup:**

1. Run `Soap Auth.request.yaml` to obtain Session ID
2. Session ID is stored in `SessionIdLatest` variable
3. Re-authenticate when session expires

---

### Request Format

SOAP requests use XML structure:

```xml
<?xml version="1.0" encoding="utf-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/>
  <soapenv:Header>
    <SessionHeader xmlns="urn:enterprise.soap.sforce.com">
      <sessionId>{{SessionIdLatest}}</sessionId>
    </SessionHeader>
  </soapenv:Header>
  <soapenv:Body>
    <!-- SOAP request body -->
  </soapenv:Body>
</soapenv:Envelope>
```

---

## Variables & Setup

### Collection Variables Reference

**Authentication Variables:**

| Variable          | Required? | Purpose                    | Example/Format                         | Set By    |
| ----------------- | --------- | -------------------------- | -------------------------------------- | --------- |
| `ServerURL`       | ✓ Yes     | Salesforce instance domain | `https://yourdomain.my.salesforce.com` | Manual    |
| `MyBearerToken`   | ✓ Yes     | REST API Bearer token      | `00D50...` (60+ chars)                 | Auth flow |
| `SessionIdLatest` | For SOAP  | SOAP session ID            | `00D50...~...`                         | SOAP auth |

**OAuth2 Configuration Variables:**

| Variable        | Required? | Purpose                 | Example              | Set By |
| --------------- | --------- | ----------------------- | -------------------- | ------ |
| `client_id`     | ✓ Yes     | Connected App Client ID | `3MVG9...`           | Manual |
| `client_secret` | ✓ Yes     | Connected App secret    | `9876543210...`      | Manual |
| `grant_type`    | Optional  | OAuth grant type        | `client_credentials` | Manual |

**Auto-Populated Variables (from responses):**

| Variable                  | Populated By    | Updated When            | Purpose                          |
| ------------------------- | --------------- | ----------------------- | -------------------------------- |
| `LatestStudentId`         | Query response  | Running Student queries | Reference latest queried Student |
| `latestInsertedStudentId` | Insert response | Creating Students       | Reference newly created Student  |
| `RootUrl`                 | Auth response   | Authenticating          | Store API root URL               |

**Dynamic Variables:**

| Variable    | Used For         | Example             |
| ----------- | ---------------- | ------------------- |
| `RESTPARAM` | Query parameters | Various test params |

---

### Initial Setup Checklist

**Step 1️⃣ : Configure Organization Variables**

```
[ ] Get your Salesforce domain from login page
[ ] Set ServerURL = https://yourdomain.my.salesforce.com
```

**Step 2️⃣ : Create Connected App (if using OAuth)**

```
[ ] In Salesforce: Setup → Apps → App Manager
[ ] Create New Connected App
[ ] Set OAuth scopes: api, refresh_token
[ ] Copy Client ID → client_id variable
[ ] Copy Client Secret → client_secret variable
```

**Step 3️⃣ : Test Authentication**

| Protocol | Action           | Request                  | Time    |
| -------- | ---------------- | ------------------------ | ------- |
| **REST** | Get Bearer Token | Run auth request         | 1-2 min |
| **SOAP** | Get Session ID   | `Soap Auth.request.yaml` | 1-2 min |

**Step 4️⃣ : Verify Connection**

```
HTTP GET /services/data/
  ✓ Should return 200 OK
  ✓ Shows available API versions
```

**Step 5️⃣ : Run a Test Query**

```
✓ REST: Run any Query ep request
✓ SOAP: Run SOAP Query Latest Inserted Student
✓ Verify you get data back
```

---

### Variable Lifecycle

**Flow During Authentication:**

```
1. User inputs client_id, client_secret
            ↓
2. Auth request executes
            ↓
3. Salesforce returns access_token
            ↓
4. Postman captures token
            ↓
5. Token stored in MyBearerToken variable
            ↓
6. Subsequent requests use {{MyBearerToken}}
            ↓
7. Token expires (24 hours typically)
            ↓
8. Re-authenticate to get new token
```

---

### Best Practices

**For Token Management:**

✅ Do: Set tokens to collection variable (reuse across requests)  
✅ Do: Set expiration reminders  
✅ Do: Use different environments for dev/test/prod  
❌ Don't: Hardcode tokens in requests  
❌ Don't: Share token-populated exports

**For Confidential Data:**

✅ Use Postman Environments for sensitive values  
✅ Mark variables as 'secret' if Postman supports it  
✅ Use connection pooling to minimize token fetches  
❌ Don't store production credentials in version control  
❌ Don't log tokens in test runs

---

### Troubleshooting Variables

**Problem: Token not working**

| Symptom          | Cause               | Solution                          |
| ---------------- | ------------------- | --------------------------------- |
| 401 Unauthorized | Expired token       | Re-run auth request               |
| 401 Unauthorized | Wrong variable name | Verify `{{MyBearerToken}}` syntax |
| Empty token      | Auth failed         | Check ServerURL and credentials   |

**Problem: Query returns wrong data**

| Symptom       | Cause        | Solution                   |
| ------------- | ------------ | -------------------------- |
| Empty records | Wrong org    | Verify ServerURL           |
| SOQL error    | Query syntax | Check URL encoding of SOQL |

---

### Persistent vs Temporary Variables

| Scope           | Persistence          | When to Use          | Example                       |
| --------------- | -------------------- | -------------------- | ----------------------------- |
| **Collection**  | Saved in collection  | Long-lived constants | `ServerURL`, `client_id`      |
| **Environment** | Saved in environment | Environment-specific | Different `ServerURL` per env |
| **Global**      | Session-only         | Temporary data       | Current user ID               |
| **Local**       | Request-only         | Request-specific     | Function parameters           |

---

## Quick Start Guide

### 5-Minute Setup

**Prerequisite:** Access to Salesforce org

| Time     | Step                   | What                                  |
| -------- | ---------------------- | ------------------------------------- |
| **0:00** | 1. Get credentials     | Admin → Retrieve login credentials    |
| **1:00** | 2. Configure variables | Update `ServerURL`, `client_id`, etc. |
| **2:00** | 3. Authenticate        | Run REST Auth request                 |
| **3:00** | 4. Verify token        | Check `MyBearerToken` is populated    |
| **4:00** | 5. Run test query      | Try any Query ep request              |
| **5:00** | 6. Success!            | Should see records returned           |

---

## Using the Collection

### REST API Workflow

**Typical Flow:**

```
1. Authenticate
    ↓
2. Query data (SELECT)
    ↓
3. Create/Update records (CRUD)
    ↓
4. Composite batch operations (if needed)
    ↓
5. Refresh token (as needed)
```

**Common Tasks:**

| Task              | Endpoint                    | HTTP Method | Notes               |
| ----------------- | --------------------------- | ----------- | ------------------- |
| List records      | `/query/`                   | GET         | Read-only           |
| Get single record | `/sobjects/{type}/{id}`     | GET         | Most specific       |
| Create record     | `/sobjects/{type}/`         | POST        | Returns new ID      |
| Update record     | `/sobjects/{type}/{id}`     | PATCH       | Partial update      |
| Delete record     | `/sobjects/{type}/{id}`     | DELETE      | Cannot be undone    |
| Get metadata      | `/sobjects/{type}/describe` | GET         | Technical info      |
| Multiple ops      | `/composite/batch`          | POST        | Parallel requests   |
| Dependent ops     | `/composite/`               | POST        | Sequential requests |

---

### SOAP API Workflow

**Typical Flow:**

```
1. Soap Auth to get SessionID
    ↓
2. Execute queries or DML
    ↓
3. Process XML responses
    ↓
4. Return to REST for more work
```

**Common Tasks:**

| Task         | Request                              | Protocol | Format       |
| ------------ | ------------------------------------ | -------- | ------------ |
| Authenticate | `Soap Auth`                          | SOAP     | XML envelope |
| Query data   | `SOAP Query Latest Inserted Student` | SOAP     | XML-SObject  |
| Insert data  | `Insert new Student`                 | SOAP     | XML-SObject  |
| Call Apex    | `StudentSOAPService-getStudents`     | SOAP     | XML-SObject  |

---

### REST vs SOAP Decision Tree

```
Do you need to call external APIs?
  YES → Use REST (easier integration)
  NO  → Continue

Do you have complex CRUD with dependencies?
  YES → Use Composite API
  NO  → Continue

Is this a quick test/prototype?
  YES → Use REST (simpler syntax)
  NO  → Continue

Do you need enterprise-grade XML integration?
  YES → Use SOAP
  NO  → Stay with REST
```

---

## Best Practices

### ✅ DO:

**Authentication:**

- ✅ Re-authenticate before token might expire
- ✅ Use separate credentials per environment (dev/test/prod)
- ✅ Rotate client secrets regularly

**Queries:**

- ✅ Use LIMIT to prevent retrieving massive result sets
- ✅ Implement pagination for production data retrieval
- ✅ Use field filtering in SELECT (don't use SELECT \*)

**Bulk Operations:**

- ✅ Use Composite API (batch/chained) to reduce API calls
- ✅ Use Tree API for hierarchical inserts
- ✅ Group independent operations in batch

**Error Handling:**

- ✅ Check HTTP status codes (200, 400, 401, etc.)
- ✅ Parse error messages in response body
- ✅ Implement retry logic for transient errors

**Monitoring:**

- ✅ Track API usage via Limits endpoint
- ✅ Log important requests/responses
- ✅ Test with realistic data volumes

---

### ❌ DON'T:

**Credentials:**

- ❌ Don't hardcode access tokens in requests
- ❌ Don't share token-populated exports
- ❌ Don't commit credentials to version control
- ❌ Don't use production credentials for testing

**Queries:**

- ❌ Don't query without WHERE conditions on large objects
- ❌ Don't ignore pagination
- ❌ Don't make the same query repeatedly without caching

**Operations:**

- ❌ Don't delete production data without backup
- ❌ Don't miss error handling
- ❌ Don't assume requests always succeed

**Performance:**

- ❌ Don't make sequential calls when parallel is possible
- ❌ Don't fetch data you won't use
- ❌ Don't ignore rate limits

---

## API Limits & Best Practices

### Request Limits Table

| Limit Type               | Value            | Applies To  | Notes                 |
| ------------------------ | ---------------- | ----------- | --------------------- |
| **Composite Batch Size** | 25 requests/call | Batch API   | Max per single call   |
| **Composite Requests**   | 25 requests/call | Chained API | Per transaction       |
| **Tree Records**         | 200 records/call | Tree API    | Per single insert     |
| **Query Results**        | Unlimited        | Query API   | Use pagination        |
| **API Callout Timeout**  | 120 seconds      | All APIs    | Request must complete |

### Monitoring API Usage

**Check Limits Endpoint:**

```http
GET /services/data/v64.0/limits/
Authorization: Bearer {{MyBearerToken}}
```

**Response includes:**

| Metric                     | Meaning                        |
| -------------------------- | ------------------------------ |
| `DailyApiRequests`         | REST API calls allowed per day |
| `DailyAsyncApexExecutions` | Async jobs per day             |
| `DailyBatchApexExecutions` | Batch jobs per day             |

**Response Format:**

```json
{
  "DailyApiRequests": {
    "Max": 15000,
    "Remaining": 14250
  }
}
```

---

## Troubleshooting Guide

### Common Issues

**Problem: 401 Unauthorized**

| Cause         | Fix                                   |
| ------------- | ------------------------------------- |
| Token expired | Re-run authentication request         |
| Wrong header  | Use `Authorization: Bearer {{token}}` |
| Invalid org   | Verify `ServerURL` is correct         |

**Problem: 400 Bad Request**

| Cause              | Fix                                       |
| ------------------ | ----------------------------------------- |
| SOQL syntax error  | Check query in each step                  |
| URL encoding issue | Verify spaces as `+` in query param       |
| Invalid field name | Verify field exists via describe endpoint |

**Problem: 404 Not Found**

| Cause                  | Fix                                            |
| ---------------------- | ---------------------------------------------- |
| Wrong API version      | Use `/v64.0/` in paths                         |
| Wrong sObject type     | Verify `Student__c` exists (not `Student`)     |
| Missing trailing slash | POST to `/sobjects/Type/` not `/sobjects/Type` |

**Problem: Composite request fails**

| Cause                | Fix                                       |
| -------------------- | ----------------------------------------- |
| referenceId mismatch | Verify exact spelling in `@{...}`         |
| allOrNone=true       | Use `false` if partial success acceptable |
| Wrong request order  | Dependencies must be ordered correctly    |

---

### Debug Checklist

Before asking for help, verify:

| Item            | Check                                           |
| --------------- | ----------------------------------------------- |
| **Credentials** | ✓ ServerURL correct? ✓ Token valid?             |
| **Syntax**      | ✓ URL spelling? ✓ JSON formatting?              |
| **Encoding**    | ✓ SOQL URL encoded? ✓ Special chars escaped?    |
| **Order**       | ✓ Dependencies sequential? ✓ Right variables?   |
| **Fields**      | ✓ Field exists? ✓ Case-sensitive? ✓ Accessible? |
| **Limits**      | ✓ Org API limits reached? ✓ Rate limited?       |
| **Response**    | ✓ Status code? ✓ Error message? ✓ Request ID?   |

---

## API Architecture Overview

### Salesforce API Layers

```
┌─────────────────────────────────────────────────────┐
│            External Systems / Apps                  │
└────────────────────────┬────────────────────────────┘
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
  ┌────────┐         ┌────────┐        ┌─────────┐
  │ REST   │         │ SOAP   │        │ GraphQL │
  │ API    │         │API     │        │ API     │
  └────────┘         └────────┘        └─────────┘
      │                  │                  │
      └──────────────────┼──────────────────┘
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
  ┌────────┐         ┌────────┐        ┌─────────┐
  │ Query  │         │ CRUD   │        │Metadata │
  │ Engine │         │Engine  │        │Engine   │
  └────────┘         └────────┘        └─────────┘
      │                  │                  │
      └──────────────────┼──────────────────┘
                         │
      ┌──────────────────▼──────────────────┐
      │                                     │
      │      Salesforce Database            │
      │     (Orgs, Objects, Records)        │
      │                                     │
      └─────────────────────────────────────┘
```

### This Collection's Endpoints

**Which endpoints this collection covers:**

| Layer        | Endpoint       | Status     | Location   |
| ------------ | -------------- | ---------- | ---------- |
| Query Engine | `/query/`      | ✓ Included | REST/OotB/ |
| CRUD Engine  | `/sobjects/`   | ✓ Included | REST/OotB/ |
| Composite    | `/composite/*` | ✓ Included | REST/OotB/ |
| Actions      | `/actions/`    | ✓ Included | REST/OotB/ |
| SOAP         | Web Services   | ✓ Included | SOAP/      |
| Metadata     | `/describe/`   | ✓ Included | In SObject |
| Limits       | `/limits/`     | ✓ Included | REST/OotB/ |
| UI-API       | `/ui-api/`     | ✓ Included | REST/OotB/ |

---

## Reference Tables

### HTTP Status Codes

| Code    | Meaning                          | Action                         |
| ------- | -------------------------------- | ------------------------------ |
| **200** | OK - Request succeeded           | Check response data            |
| **201** | Created - Record inserted        | Location header has new URL    |
| **204** | No Content - Success, no body    | Request completed              |
| **400** | Bad Request - Invalid syntax     | Review request body/params     |
| **401** | Unauthorized - Auth failed       | Re-authenticate                |
| **403** | Forbidden - No permission        | Check field/record permissions |
| **404** | Not Found - Resource missing     | Verify URL and IDs             |
| **429** | Too Many Requests - Rate limited | Wait and retry                 |
| **500** | Server Error - Salesforce issue  | Contact Salesforce support     |

### Common SOQL Keywords

| Keyword    | Type     | Example                           |
| ---------- | -------- | --------------------------------- |
| `SELECT`   | Clause   | `SELECT Id, Name FROM Student__c` |
| `FROM`     | Clause   | From object (required)            |
| `WHERE`    | Clause   | `WHERE Status = 'Active'`         |
| `AND`      | Operator | Combine conditions                |
| `OR`       | Operator | Combine conditions                |
| `LIKE`     | Operator | Pattern matching with %           |
| `IN`       | Operator | `WHERE Id IN (...)`               |
| `NOT IN`   | Operator | Exclude items                     |
| `ORDER BY` | Clause   | Sort results ASC/DESC             |
| `LIMIT`    | Clause   | Maximum records to return         |
| `OFFSET`   | Clause   | Pagination offset                 |
| `GROUP BY` | Clause   | Aggregate results                 |
| `HAVING`   | Clause   | Post-aggregation filter           |
| `COUNT()`  | Function | Aggregate count                   |
| `SUM()`    | Function | Aggregate sum                     |
| `AVG()`    | Function | Aggregate average                 |

### Common Salesforce Objects

| Object API Name | Object Label | Use For                  |
| --------------- | ------------ | ------------------------ |
| `Account`       | Account      | Companies, organizations |
| `Contact`       | Contact      | People, individuals      |
| `Lead`          | Lead         | Prospects                |
| `Opportunity`   | Opportunity  | Sales deals              |
| `Case`          | Case         | Support tickets          |
| `Task`          | Task         | Activities               |
| `Event`         | Event        | Calendar events          |
| `Student__c`    | Student      | Custom (this collection) |

---

## Document Sections Quick Links

| Section                  | Purpose                  | Location                |
| ------------------------ | ------------------------ | ----------------------- |
| **Authentication**       | How to get access tokens | Top of README           |
| **Collection Structure** | Visual directory tree    | Top section             |
| **Query Endpoint**       | SOQL query execution     | REST API section        |
| **SObject Endpoint**     | CRUD operations          | REST API section        |
| **Actions Endpoint**     | Invocable Apex + Flows   | REST API section        |
| **Composite API**        | Batch & chained requests | REST API section        |
| **SOAP API**             | XML web services         | SOAP section            |
| **Variables Setup**      | Configuration guide      | Setup section           |
| **Usage Guide**          | How to use collection    | Usage section           |
| **Best Practices**       | Do's and don'ts          | Practices section       |
| **Troubleshooting**      | Common issues & fixes    | Troubleshooting section |

---

## Additional Resources

### Official Salesforce Documentation

| Resource              | Link                                                                                                                                                                                         | For                         |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| **REST API Guide**    | [developer.salesforce.com/docs/atlas.en-us.api_rest](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/)                                                              | Detailed REST API reference |
| **SOAP API Guide**    | [developer.salesforce.com/docs/atlas.en-us.api](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/)                                                                             | Complete SOAP API reference |
| **SOQL Reference**    | [developer.salesforce.com/docs/atlas.en-us.soql_sosl](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/)                                                           | SOQL syntax and examples    |
| **Composite API**     | [developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite.htm](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_composite.htm) | Batch/Chained requests      |
| **Invocable Actions** | [developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/actions_invocable](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/actions_invocable.htm)         | Actions API reference       |

### External Tools & Resources

| Tool                    | Type        | Purpose                               |
| ----------------------- | ----------- | ------------------------------------- |
| **Postman**             | HTTP Client | Making API requests (this collection) |
| **Insomnia**            | HTTP Client | REST/GraphQL client alternative       |
| **VS Code REST Client** | Extension   | Make requests from VS Code            |
| **Workbench**           | Web Tool    | Salesforce API explorer               |
| **SFDX**                | CLI         | Salesforce DevOps tool                |

### Learning Resources

| Resource              | Type      | Best For                   |
| --------------------- | --------- | -------------------------- |
| **Trailhead Modules** | Tutorial  | Salesforce-guided learning |
| **API Documentation** | Reference | Implementation details     |
| **YouTube Tutorials** | Video     | Visual learning            |
| **Developer Forums**  | Community | Q&A and issue solving      |
| **Sample Code**       | Gists     | Copy-paste examples        |

---

## Glossary

| Term                | Definition                        | Example                                   |
| ------------------- | --------------------------------- | ----------------------------------------- |
| **API**             | Application Programming Interface | Code interface for software communication |
| **REST**            | Representational State Transfer   | HTTP-based architecture style             |
| **SOAP**            | Simple Object Access Protocol     | XML-based web service protocol            |
| **SOQL**            | Salesforce Object Query Language  | SQL-like query language for Salesforce    |
| **sObject**         | Salesforce Object                 | Any database table (Account, Contact)     |
| **Record**          | Individual sObject instance       | One specific Account record               |
| **Bearer Token**    | Authentication credential         | Access token for REST API                 |
| **Session ID**      | Authentication credential         | Access token for SOAP API                 |
| **referenceId**     | Identifier in composite requests  | Reference previous request results        |
| **Composite API**   | Multi-operation API               | Batch multiple requests in one call       |
| **callout**         | External API call                 | REST request to external service          |
| **Gateway Timeout** | Network error                     | Request took too long                     |
| **Rate Limiting**   | API throttling                    | Too many requests too quickly             |

---

## Support & Feedback

**Having Issues?**

1. Check the [Troubleshooting Guide](#troubleshooting-guide) above
2. Review [Best Practices](#best-practices) for common pitfalls
3. Consult [Official Salesforce Documentation](#official-salesforce-documentation)
4. Check request/response details in Postman console

**Collection Feedback**

- Found an issue? Document it with: request name, error message, steps to reproduce
- Want to add requests? Follow existing request naming conventions
- Want to improve docs? Edit the YAML definition files in `.resources/`

---

**Collection Metadata:**

| Item                       | Value                         |
| -------------------------- | ----------------------------- |
| **Collection Name**        | INTR BASICS                   |
| **Salesforce API Version** | v64.0                         |
| **Last Updated**           | May 2026                      |
| **Primary Use**            | Training & Reference          |
| **Protocols**              | REST, SOAP                    |
| **Documentation**          | Embedded in README.md         |
| **Source**                 | YAML-based Postman collection |

---

## Document Information

- **Format:** Markdown (.md)
- **Version:** 1.0
- **Last Revised:** May 20, 2026
- **Printable:** Yes (optimized for print layout)
- **Total Endpoints Documented:** 25+
- **Code Examples:** 50+
- **Tables:** 40+
