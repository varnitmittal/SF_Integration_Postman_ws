# INTR BASICS Salesforce API Study Notes

This document is a comprehensive study reference for Salesforce API concepts extracted from the workspace `definition.yaml` files. It is organized to help you learn the key API layers, endpoint patterns, object behaviors, and developer tooling concepts without relying on Postman-specific details.

---

## How to Use This Document

- Read the top-level comparison tables first to understand API roles.
- Use each section to review one API layer at a time.
- Focus on the endpoint patterns, data flows, and key object concepts.
- Study the tables for quick memorization, and use examples for practical context.

---

## API Layer Comparison

| API Layer | Primary Purpose | Common Objects | Typical Use Cases | Output Format |
| --- | --- | --- | --- | --- |
| REST API | Business data access and CRUD | Account, Contact, Lead, Student__c | Query data, create/update records, describe objects | JSON |
| Tooling API | Developer tooling and metadata | ApexClass, ApexTrigger, ApexLog, TraceFlag | Debug logs, metadata inspection, anonymous Apex | JSON |
| SOAP API | XML web service integration | Enterprise SOAP objects | Legacy integrations, WSDL-based clients | XML |
| Metadata API | Configuration deployment and metadata management | CustomObject, Flow, Layout, PermissionSet | CI/CD, org migration, deployment | XML |

---

## Key Concepts to Memorize

| Concept | Why it matters | Exam focus |
| --- | --- | --- |
| `Query` vs `SObject` | Query is for read-only SOQL retrieval; SObject is for CRUD operations | Distinguish data retrieval vs record manipulation |
| `Composite` types | Batch = independent; Composite = chained; Tree = hierarchical inserts | Know when to reduce API calls and manage dependencies |
| `Actions API` | Invoke Flow and Apex via REST | Understand low-code orchestration and integration patterns |
| `Execute Anonymous` | Run Apex on the fly without saving | Know temporary execution and debug output behavior |
| `TraceFlag` + `ApexLog` | Enable and capture debug logs | Essential for troubleshooting and production support |
| Tooling vs REST | Tooling works on platform metadata; REST works on business data | Know which API to choose for developer vs application workflows |

---

## Authentication Overview

### REST Authentication

- REST uses OAuth 2.0 bearer tokens.
- Authorization header format:

```http
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

- Tokens expire; refresh or re-authenticate as needed.
- REST authentication is required for `query`, `sobjects`, `actions`, and `composite` endpoints.

### SOAP Authentication

- SOAP uses a session ID or OAuth token inside the SOAP header.
- Example SOAP header:

```xml
<SessionHeader xmlns="urn:enterprise.soap.sforce.com">
  <sessionId>SESSION_ID</sessionId>
</SessionHeader>
```

- SOAP authentication is separate from message payload and is usually obtained via a login call.
- SOAP is often used in older enterprise integrations and WSDL-based clients.

### Tooling Authentication

- Tooling API also uses bearer tokens in the Authorization header.
- Example:

```http
Authorization: Bearer ACCESS_TOKEN
```

- Tooling requests are subject to the same OAuth session rules as REST.

---

## Salesforce REST API

### 1. Query Endpoint

#### Endpoint Pattern

```http
GET /services/data/v64.0/query?q=<URL_ENCODED_SOQL>
```

#### Purpose

- Execute SOQL queries.
- Retrieve data across objects.
- Support filtering, sorting, aggregation, and pagination.

#### Response Model

| Field | Description |
| --- | --- |
| `totalSize` | Total matching rows for the query |
| `done` | `true` if all rows were returned; `false` if pagination remains |
| `records` | Array of returned rows |
| `nextRecordsUrl` | Path to fetch the next page of results |

#### Common SOQL Patterns

| Pattern | Example | Use |
| --- | --- | --- |
| Basic select | `SELECT Id, Name FROM Student__c` | Retrieve rows |
| Filter | `WHERE Name='St-0001'` | Narrow results |
| Sort | `ORDER BY CreatedDate DESC` | Order rows |
| Limit | `LIMIT 5` | Restrict row count |
| Like | `WHERE Name LIKE 'St%'` | Pattern matches |
| Aggregate | `SELECT COUNT() FROM Student__c` | Compute totals |
| Group | `GROUP BY School__c` | Aggregate by field |

#### URL Encoding

- SOQL must be URL encoded when passed as a query string.
- Spaces typically become `+`.
- Example:

```text
SELECT Id, Name FROM Student__c
```

becomes:

```text
SELECT+Id,+Name+FROM+Student__c
```

#### Pagination

- Use the header:

```http
Sforce-Query-Options: batchSize=10
```

- If the response contains `done: false`, use `nextRecordsUrl`.
- Example:

```http
GET /services/data/v64.0/query/01gXXXXXXXXXXXX
```

#### Example Queries

| Goal | Query Example |
| --- | --- |
| Basic retrieval | `SELECT Id, Name FROM Student__c` |
| Filtering | `SELECT Id, Name FROM Student__c WHERE Name='St-0001'` |
| Sorted results | `SELECT Id, Name FROM Student__c ORDER BY CreatedDate DESC` |
| Pattern search | `SELECT Id, Name FROM Student__c WHERE Name LIKE 'St%'` |
| Count rows | `SELECT COUNT() FROM Student__c` |
| Group and count | `SELECT School__c, COUNT(Id) studentCount FROM Student__c GROUP BY School__c` |

---

### 2. SObject Endpoint

#### Endpoint Pattern

```http
/services/data/v64.0/sobjects/{sObjectName}/
```

#### Purpose

- Work with Salesforce objects.
- Support CRUD and metadata operations.
- Handle both standard and custom objects.

#### Supported Object Types

| Type | Example |
| --- | --- |
| Standard | `Account`, `Contact`, `Lead` |
| Custom | `Student__c`, `School__c` |

#### Common Operations

| Operation | HTTP Method | Endpoint | Purpose |
| --- | --- | --- | --- |
| List objects | GET | `/sobjects/` | Find available objects |
| Retrieve record | GET | `/sobjects/{ObjectName}/{Id}` | Fetch a single record |
| Create record | POST | `/sobjects/{ObjectName}/` | Insert a record |
| Update record | PATCH | `/sobjects/{ObjectName}/{Id}` | Modify a record |
| Delete record | DELETE | `/sobjects/{ObjectName}/{Id}` | Remove a record |
| Describe metadata | GET | `/sobjects/{ObjectName}/describe` | Get field/relationship metadata |

#### Describe Metadata

- `describe` returns object structure, fields, relationships, and permissions.
- Important for dynamic integrations and UI generation.

Example response items:

- `name`
- `label`
- `fields`
- `picklistValues`
- `recordTypeInfos`

#### Record Behavior

| Action | Successful Response |
| --- | --- |
| Create | `201 Created` with `id` and `success: true` |
| Update | `204 No Content` |
| Delete | `204 No Content` or success payload depending on call |

#### Example Create Request

```json
POST /services/data/v64.0/sobjects/Student__c/
Content-Type: application/json

{
  "Student_Name__c": "Rahul",
  "Class_Enrolled__c": 10
}
```

Example response:

```json
{
  "id": "a1hXXXXXXXXXXXX",
  "success": true,
  "errors": []
}
```

#### Example Update Request

```http
PATCH /services/data/v64.0/sobjects/Student__c/{recordId}
Content-Type: application/json

{
  "Student_Name__c": "Updated Name"
}
```

Successful update returns:

```http
204 No Content
```

#### Example Delete Request

```http
DELETE /services/data/v64.0/sobjects/Student__c/{recordId}
```

Example response:

```json
{
  "id": "a1hgK000002LFR7QAO",
  "success": true
}
```

---

### 3. Actions API / Flow / Invocable Apex

#### Endpoint Pattern

```http
POST /services/data/v64.0/actions/custom/flow/{FlowApiName}
```

#### Purpose

- Invoke Salesforce Flows from REST.
- Execute invocable Apex logic.
- Support orchestrated business processes.
- Provide a bridge between external systems and Flow/Apex logic.

#### Key Concepts

| Term | Meaning |
| --- | --- |
| `@InvocableMethod` | Apex method usable by Flow and Actions API |
| `@InvocableVariable` | Apex variable exposed to Flow input/output |
| `Autolaunched Flow` | Flow type with no UI, suitable for API invocation |
| `outputValues` | Values returned from Flow to the caller |

#### Example Apex Pattern

```apex
public with sharing class StudentWelcomeService {
    public class Request {
        @InvocableVariable(required=true)
        public String studentName;
    }

    @InvocableMethod(label='Generate Welcome Message')
    public static List<String> generateMessage(List<Request> requests) {
        List<String> responses = new List<String>();
        for(Request req : requests) {
            responses.add('Welcome ' + req.studentName + ' to Salesforce REST APIs!');
        }
        return responses;
    }
}
```

#### Flow Variables

| Variable | Role | Example |
| --- | --- | --- |
| `studentNameVar` | Flow input variable | `Text` input from REST request |
| `resultText` | Flow output variable | Message returned to caller |

#### REST Request Example

```json
POST /services/data/v64.0/actions/custom/flow/StudentWelcomeService
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "inputs": [
    {
      "studentNameVar": "Alex Smith"
    }
  ]
}
```

#### REST Response Example

```json
[
  {
    "actionName": "StudentWelcomeService",
    "isSuccess": true,
    "outputValues": {
      "resultText": "Welcome Alex Smith to Salesforce REST APIs!",
      "Flow__InterviewStatus": "Finished"
    },
    "version": 1
  }
]
```

#### Action Flow Summary

```text
REST request → Actions API → Autolaunched Flow → Invocable Apex → REST response
```

#### When to use Actions API

- When you need a REST-accessible business process.
- When logic should run through Flow or invocable Apex.
- When you want reusable, declarative orchestration with Apex extension.

---

### 4. Composite API

#### Overview

Composite API groups multiple REST operations into a single HTTP request. It is designed to reduce API call count and support dependent operations.

#### Composite Types

| Composite Type | Endpoint | Behavior | Best for |
| --- | --- | --- | --- |
| Batch | `/composite/batch` | Independent requests executed in one payload | Parallel retrieval or unrelated operations |
| Chained | `/composite` | Sequential requests with data sharing via `referenceId` | Multi-step workflows and dependent record creation |
| Tree | `/composite/tree/{sObject}` | Hierarchical parent-child record inserts | Bulk insert of related records |

#### Batch API

- Requests are independent and cannot share data.
- Useful for retrieving multiple resources in one call.

Example:

```json
{
  "batchRequests": [
    {"method": "GET", "url": "v64.0/limits"},
    {"method": "GET", "url": "v64.0/query?q=SELECT+Id+FROM+Account"}
  ]
}
```

#### Chained API

- Uses `referenceId` to share earlier results with later requests.
- Useful for creating related records in sequence.

Example:

```json
{
  "compositeRequest": [
    {
      "method": "POST",
      "url": "/services/data/v64.0/sobjects/Account",
      "referenceId": "newAccount",
      "body": {"Name": "Acme Corp"}
    },
    {
      "method": "POST",
      "url": "/services/data/v64.0/sobjects/Contact",
      "referenceId": "newContact",
      "body": {
        "LastName": "Smith",
        "AccountId": "@{newAccount.id}"
      }
    }
  ]
}
```

#### Tree API

- Insert parent-child data in one request.
- Useful for hierarchical objects and bulk creation.

Example structure:

```text
Account
├── Contact
├── Contact
└── Opportunity
```

Example request:

```json
{
  "records": [
    {
      "attributes": {"type": "Account", "referenceId": "refAccount1"},
      "Name": "Acme Corporation"
    }
  ]
}
```

#### Composite Benefits

- fewer API calls
- lower latency
- reduced network traffic
- better orchestration support
- stronger mobile performance

#### Comparison Summary

| API | Use case | Data sharing | Hierarchy support |
| --- | --- | --- | --- |
| `/batch` | independent requests | no | no |
| `/composite` | dependent requests | yes | no |
| `/tree` | hierarchical inserts | yes | yes |

---

## Salesforce Tooling API

### 1. Tooling API Overview

#### Purpose

Tooling API is a developer/platform engineering API used for:

- code inspection
- debugging
- metadata querying
- org analysis
- lightweight metadata operations

#### Base Endpoint

```http
/services/data/v64.0/tooling/
```

### Tooling vs REST vs Metadata

| API | Primary Use | Common Objects | Best for |
| --- | --- | --- | --- |
| REST | Business data | Account, Contact, Student__c | CRUD and data retrieval |
| Tooling | Developer metadata | ApexClass, ApexLog, TraceFlag | Debugging, code inspection |
| Metadata | Org deployment | CustomObject, Flow, Layout | Deploying metadata and config |

---

### Most Important Tooling Objects

| Object | Purpose |
| --- | --- |
| ApexClass | Apex source and metadata |
| ApexTrigger | Trigger metadata |
| ApexLog | Debug logs from execution |
| TraceFlag | Logging configuration |
| Flow | Flow metadata |
| CustomObject | Custom object definition |
| CustomField | Custom field definition |
| Layout | Page layout metadata |
| RecordType | Record type metadata |
| PermissionSet | Permission set metadata |
| AsyncApexJob | Async Apex execution status |

---

### Common Tooling Uses

| Use Case | Tooling API Component |
| --- | --- |
| Debug Apex | ApexLog + TraceFlag |
| Read org code | ApexClass |
| Inspect flows | Flow |
| Build schema explorers | EntityDefinition |
| Monitor async jobs | AsyncApexJob |
| Run quick Apex | Execute Anonymous |

---

### Example — Query Apex Classes

```http
GET /services/data/v64.0/tooling/query?q=SELECT+Name+FROM+ApexClass
```

Purpose: retrieve Apex class metadata.

---

### Example — Retrieve Debug Logs

```http
GET /services/data/v64.0/tooling/query?q=SELECT+Id,Operation,Status+FROM+ApexLog
```

Purpose: read log metadata and identify records for details.

---

## 2. Execute Anonymous

### Purpose

Execute Anonymous runs temporary Apex code immediately without creating an Apex class, trigger, or flow.

### Endpoint

```http
GET /services/data/v64.0/tooling/executeAnonymous/
```

### Key behaviors

- executes Apex instantly
- does not persist the code
- useful for testing, debugging, data fixes, and admin tasks
- often used for one-off scripts

### Example

```http
GET /services/data/v64.0/tooling/executeAnonymous/?anonymousBody=System.debug('Hello Tooling API');
```

### Example — Insert Record

```http
GET /services/data/v64.0/tooling/executeAnonymous/?anonymousBody=Account%20a%20%3D%20new%20Account(Name%3D'Test%20Account')%3B%20insert%20a%3B
```

### Response Fields

| Field | Meaning |
| --- | --- |
| `compiled` | whether the Apex code compiled successfully |
| `success` | whether the execution succeeded |
| `line` | error line number if compilation failed |
| `column` | error column if compilation failed |

### Important Limitation

- code is temporary and disappears after execution
- only the results remain in the org
- no persisted Apex source is created

### Debugging Relationship

- Debug output is captured to `ApexLog` only if a `TraceFlag` is active.
- This means execute anonymous code can run successfully but still require logging configuration to inspect debug messages.

---

## 3. TraceFlag

### Purpose

TraceFlag configures Salesforce debug logging for a user, class, or trigger.

### Endpoint

```http
/services/data/v64.0/tooling/sobjects/TraceFlag
```

### Key Fields

| Field | Purpose |
| --- | --- |
| `TracedEntityId` | ID of the user, class, or trigger being logged |
| `DebugLevelId` | Logging detail configuration |
| `LogType` | Type of log such as `USER_DEBUG` |
| `StartDate` | When logging begins |
| `ExpirationDate` | When logging stops |

### Why TraceFlag matters

- Without a TraceFlag, `ApexLog` entries may not be generated.
- TraceFlag controls what is logged and for whom.
- It is essential for production troubleshooting and developer debugging.

### Typical Flow

```text
Create DebugLevel → Create TraceFlag → Execute transaction → ApexLog generated → Retrieve ApexLog
```

### DebugLevel relationship

- `DebugLevel` defines logging verbosity for:
  - Apex code
  - SOQL queries
  - DML operations
  - workflow actions

---

## 4. ApexLog

### Purpose

ApexLog stores Salesforce debug logs for executed transactions.

### Query logs

```http
GET /services/data/v64.0/tooling/query?q=SELECT+Id,Operation,Status,StartTime+FROM+ApexLog
```

### Key fields

| Field | Meaning |
| --- | --- |
| `Id` | log record ID |
| `Operation` | execution type |
| `Status` | `Success` or `Error` |
| `StartTime` | when the log entry started |

### Retrieve full body

```http
GET /services/data/v64.0/tooling/sobjects/ApexLog/{LogId}/Body
```

### Log body content

- `USER_DEBUG` statements
- SOQL queries executed
- DML operations executed
- exceptions and stack traces
- governor limit usage

### Important dependency

- Logs are created only when a TraceFlag is enabled.
- This is the most important prerequisite for Apex debugging using Tooling API.

### Debug flow

```text
Enable TraceFlag → Run transaction → ApexLog generated → Retrieve log → Analyze issue
```

### Importance in practice

ApexLog is one of the most used Tooling API objects for:

- debugging
- issue investigation
- performance analysis
- production support

---

## Salesforce SOAP API

### Overview

- SOAP is an XML-based web services API.
- It is often used for legacy integrations and WSDL clients.
- SOAP operations are transmitted in an XML envelope with a `Header` and a `Body`.

### Common SOAP Operations

| Operation | Purpose |
| --- | --- |
| `login` | Authenticate and obtain a session ID |
| `query` | Execute SOQL and retrieve records |
| `create` | Insert records via SOAP |
| `update` | Update records via SOAP |
| `delete` | Delete records via SOAP |
| `describeSObjects` | Retrieve object metadata |

### Authentication

- SOAP authentication is typically provided by a session ID.
- The session ID is included in the SOAP header, not in HTTP headers.

Example SOAP header:

```xml
<soapenv:Header>
  <SessionHeader xmlns="urn:enterprise.soap.sforce.com">
    <sessionId>{{SessionIdLatest}}</sessionId>
  </SessionHeader>
</soapenv:Header>
```

### Request/Response Flow

1. `login` or OAuth obtains a session token.
2. Session token is placed in the SOAP header.
3. SOAP body contains the business operation.
4. Response is XML with nested result details.

### Why SOAP still matters

- Required when clients depend on WSDL contracts.
- Useful for enterprise systems with strict XML schemas.
- Provides strong typed integration for some legacy tooling.

---

## Practical Study Tips

- Memorize endpoint patterns and the purpose of each API layer.
- Practice explaining when to use REST vs Tooling vs Metadata.
- Learn the `Composite` differences by comparing batch, chained, and tree.
- Remember that `Execute Anonymous` is temporary and requires `TraceFlag` to capture debug output.
- Use `ApexLog` and `TraceFlag` together for effective debugging.

---

## Notes

- Some definition files in the workspace contained only metadata names or collection configuration without additional descriptive content.
- This document includes all available descriptive notes from the workspace sources.
