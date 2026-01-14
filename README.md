# RESTful-Microservice-ReactiveMessagingService

A reactive microservice for managing messages targeted to specific users, identified by their email addresses. Built with Spring Boot WebFlux and MongoDB for reactive, non-blocking message handling.

## Table of Contents
- [Overview](#overview)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [API Endpoints](#api-endpoints)
- [Message Structure](#message-structure)
- [Setup and Installation](#setup-and-installation)
- [Usage Examples](#usage-examples)
- [Validation Rules](#validation-rules)

## Overview

ReactiveMessagingService is a standalone microservice that enables managing messages addressed to specific users. 
The service provides reactive endpoints for creating, retrieving, and filtering messages with support for pagination and sorting.

### Key Features
- ✅ Reactive programming with Project Reactor (Mono/Flux)
- ✅ MongoDB for persistent storage
- ✅ Email validation and normalization
- ✅ Pagination support for all query operations
- ✅ Server-Sent Events (SSE) for streaming responses
- ✅ Filtering by recipient, sender, and urgency
- ✅ Automatic timestamp generation
- ✅ Flexible JSON structure for additional details

## Technology Stack

- **Java 25**
- **Spring Boot** (WebFlux for reactive programming)
- **Spring Data MongoDB Reactive**
- **MongoDB** (containerized with Docker Compose)
- **Project Reactor** (Mono/Flux)
- **Gradle** (build tool)

## Architecture

### Project Structure
```
├── api/
│   └── ReactiveMessagingController.java    # REST API layer
├── logic/
│   ├── ReactiveMessagingService.java       # Service interface
│   └── ReactiveMessagingServiceImpl.java   # Service implementation
├── dal/
│   └── ReactiveMessageRepository.java      # MongoDB repository
├── model/
│   ├── MessageBoundary.java                # API DTO
│   ├── MessageEntity.java                  # Database entity
│   └── MessageConverter.java               # Entity-Boundary converter
└── error/
    └── BadRequestException.java            # Custom exception
```

### Design Patterns
- **Repository Pattern**: Data access abstraction
- **DTO Pattern**: Separation between API and database models
- **Converter Pattern**: Translation between layers
- **Reactive Streams**: Non-blocking I/O operations

## API Endpoints

All endpoints produce `text/event-stream` (Server-Sent Events) for reactive streaming.

### Core Endpoints

#### 1. Create Message
```http
POST /messages
Content-Type: application/json
```

Creates a new message and stores it in the database.

**Request Body:**
```json
{
  "target": "recipient@example.com",
  "sender": "sender@example.com",
  "title": "Message Title",
  "urgent": false,
  "moreDetails": {
    "key": "value"
  }
}
```

**Response:** Returns the created message with generated `id` and `publicationTimestamp`.

---

#### 2. Get All Messages
```http
GET /messages?page={page}&size={size}
```

Retrieves all messages with pagination.

**Query Parameters:**
- `page` (optional, default: 0) - Page number
- `size` (optional, default: 10) - Page size

**Sorting:** Descending by `publicationTimestamp`, then ascending by `id`

---

#### 3. Get Messages by Recipient
```http
GET /messages?search=byRecipient&value={recipientEmail}&page={page}&size={size}
```

Retrieves messages sent to a specific recipient.

**Query Parameters:**
- `search=byRecipient` (required)
- `value` (required) - Recipient's email address
- `page` (optional, default: 0)
- `size` (optional, default: 10)

---

#### 4. Get Messages by Sender
```http
GET /messages?search=bySender&value={senderEmail}&page={page}&size={size}
```

Retrieves messages sent by a specific sender.

**Query Parameters:**
- `search=bySender` (required)
- `value` (required) - Sender's email address
- `page` (optional, default: 0)
- `size` (optional, default: 10)

---

#### 5. Delete All Messages
```http
DELETE /messages
```

Deletes all messages from the database. Useful for testing purposes.

---

### Bonus Endpoints

#### 6. Get Message by ID
```http
GET /messages?search=byId&value={id}
```

Retrieves a specific message by its unique identifier.

**Query Parameters:**
- `search=byId` (required)
- `value` (required) - Message ID

---

#### 7. Get All Urgent Messages
```http
GET /messages?search=byUrgent&page={page}&size={size}
```

Retrieves only urgent messages (where `urgent=true`).

**Query Parameters:**
- `search=byUrgent` (required)
- `page` (optional, default: 0)
- `size` (optional, default: 10)

---

#### 8. Get Urgent Messages by Recipient
```http
GET /messages?search=urgentOnlyByRecipient&value={recipientEmail}&page={page}&size={size}
```

Retrieves urgent messages sent to a specific recipient.

**Query Parameters:**
- `search=urgentOnlyByRecipient` (required)
- `value` (required) - Recipient's email address
- `page` (optional, default: 0)
- `size` (optional, default: 10)

---

#### 9. Get Urgent Messages by Sender
```http
GET /messages?search=urgentOnlyBySender&value={senderEmail}&page={page}&size={size}
```

Retrieves urgent messages sent by a specific sender.

**Query Parameters:**
- `search=urgentOnlyBySender` (required)
- `value` (required) - Sender's email address
- `page` (optional, default: 0)
- `size` (optional, default: 10)

---

## Message Structure

### MessageBoundary (API Layer)

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "target": "jill@othercorp.org",
  "sender": "jane@corp.com",
  "title": "Hello Reactive World!",
  "publicationTimestamp": "2025-12-18T13:14:57.156Z",
  "urgent": false,
  "moreDetails": {
    "firstAttribute": "detailed text",
    "anotherAttribute": 42,
    "arrayAttribute": ["this", 8.8, {"nestedJson": true}]
  }
}
```

### Field Descriptions

| Field | Type | Description | Set By |
|-------|------|-------------|--------|
| `id` | String | Unique identifier (UUID format) | Service (auto-generated) |
| `target` | String | Recipient's email address | Client (required) |
| `sender` | String | Sender's email address | Client (required) |
| `title` | String | Message title | Client (required, not blank) |
| `publicationTimestamp` | ZonedDateTime | Timestamp when message was created | Service (auto-generated) |
| `urgent` | Boolean | Whether the message is urgent | Client (required) |
| `moreDetails` | Map<String, Object> | Flexible JSON for additional data | Client (optional) |

## Setup and Installation

### Prerequisites
- Java 17 or higher
- Docker and Docker Compose
- Gradle 7.x or higher

### 1. Start MongoDB
```bash
docker-compose up -d
```

This starts MongoDB on port 27017 with:
- Database: `messagingdb`
- Username: `root`
- Password: `secret`

### 2. Configure Application
Ensure your `application.properties` or `application.yml` contains:
```properties
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=messagingdb
spring.data.mongodb.username=root
spring.data.mongodb.password=secret
spring.data.mongodb.authentication-database=admin
```

### 3. Build and Run
```bash
./gradlew bootRun
```

The service will start on `http://localhost:9080` (or your configured port).

## Usage Examples

### Create a Message
```bash
curl -X POST http://localhost:9080/messages \
  -H "Content-Type: application/json" \
  -d '{
    "target": "user@example.com",
    "sender": "admin@myapp.com",
    "title": "Welcome to the Service",
    "urgent": true,
    "moreDetails": {
      "category": "onboarding",
      "priority": 1
    }
  }'
```

### Get All Messages (First Page)
```bash
curl "http://localhost:9080/messages?page=0&size=10"
```

### Get Messages by Recipient
```bash
curl "http://localhost:9080/messages?search=byRecipient&value=user@example.com&page=0&size=10"
```

### Get Urgent Messages
```bash
curl "http://localhost:9080/messages?search=byUrgent&page=0&size=10"
```

### Get Message by ID
```bash
curl "http://localhost:9080/messages?search=byId&value=550e8400-e29b-41d4-a716-446655440000"
```

### Delete All Messages
```bash
curl -X DELETE http://localhost:9080/messages
```

## Validation Rules

### Email Validation
- Must be a valid email format: `user@domain.tld`
- Automatically normalized: trimmed, lowercased, URL-decoded
- Both `target` and `sender` fields are validated

### Field Validation
- **target**: Required, must be a valid email
- **sender**: Required, must be a valid email
- **title**: Required, cannot be blank
- **urgent**: Required, must be `true` or `false` (not null)
- **moreDetails**: Optional, can be empty map or complex JSON structure

### Pagination Validation
- **page**: Must be ≥ 0 (default: 0)
- **size**: Must be > 0 (default: 10)

### Error Responses
Invalid requests return HTTP 400 (Bad Request) with error message:
```json
{
  "timestamp": "2025-01-14T10:30:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "target must be a valid email",
  "path": "/messages"
}
```

## Default Behavior

### Pagination Defaults
- Default page: `0`
- Default size: `10`

### Sorting
All queries sort results by:
1. `publicationTimestamp` (descending) - newest first
2. `id` (ascending) - deterministic ordering for same timestamp

### Empty Results
- Queries with no matching results return an empty Flux (no error)
- Pages beyond available data return empty Flux

## Database Schema

### Collection: MESSAGES
```javascript
{
  _id: "550e8400-e29b-41d4-a716-446655440000",
  target: "user@example.com",
  sender: "admin@example.com",
  title: "Message Title",
  publicationTimestamp: ISODate("2025-01-14T10:30:00.000Z"),
  urgent: false,
  moreDetails: {
    // flexible JSON structure
  }
}
```

## Testing

### Test Checklist
- ✅ Create message with valid data
- ✅ Create message with invalid email formats
- ✅ Create message with missing required fields
- ✅ Retrieve messages with pagination
- ✅ Filter by recipient email
- ✅ Filter by sender email
- ✅ Filter by urgent flag
- ✅ Retrieve specific message by ID
- ✅ Delete all messages

### Sample Test Flow
```bash
# 1. Clean database
curl -X DELETE http://localhost:9080/messages

# 2. Create test messages
curl -X POST http://localhost:9080/messages -H "Content-Type: application/json" \
  -d '{"target":"alice@test.com","sender":"bob@test.com","title":"Test 1","urgent":true}'

# 3. Verify creation
curl "http://localhost:9080/messages?page=0&size=10"

# 4. Test filtering
curl "http://localhost:9080/messages?search=byRecipient&value=alice@test.com"
```

Made with ❤️ by Maor Mordo
