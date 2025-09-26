E-Commerce Order Processing System
Architecture Overview
This system implements a hybrid approach combining:

Direct database access for product search and browsing (synchronous)
Kafka-based event processing for order creation (asynchronous)
The separation ensures:

Product searches get real-time data with minimal latency
Order processing remains reliable and scalable
Database load is optimized by separating read/write operations
Product Dashboard and Search Flow (Synchronous Path)
Components Involved
ProductController.java - Handles HTTP requests for product data
ProductRepository.java - Database interface for product queries
Product.java - JPA entity mapping to database table
Workflow
Frontend Initiation
When a user searches for products:

React frontend (dashboard.js) makes GET request to /api/products
Optional search parameter can be included
Backend Processing
The ProductController:

For general browsing: Calls productRepository.findAll()
For searches: Calls findByNameContainingIgnoreCaseOrDescriptionContainingIgnoreCase()
Response Handling

Formats results as JSON
Returns HTTP 200 with product data
Handles errors with appropriate status codes
Key Characteristics
Real-time inventory/pricing data
Sub-100ms response times
Simple request-response cycle
Order Items (Asynchronous Path)
Kafka Order Processing - Module Responsibilities
Producer Service Flow (Frontend → Kafka)
1. OrderController.java
Input: HTTP POST request with OrderRequest payload

Function:

Validates request structure (customer info, items, payment)
Generates unique order number (UUID)
Transforms OrderRequest → OrderEvent DTO Output: Passes OrderEvent to KafkaProducerService
2. KafkaProducerService.java
Input: OrderEvent from Controller

Function:

Serializes OrderEvent to JSON
Publishes to "orders" Kafka topic
Implements retry logic (3 attempts)
Manages Kafka transactions
Output: Message written to Kafka topic

3. OrderEvent.java
Structure:

Order metadata (number, timestamp)
Customer details (name, email, address)
Item list (product IDs, quantities, prices)
Payment method
Role: Standardized message format for Kafka

Consumer Service Flow (Kafka → Database)
1. OrderConsumerService.java
Input: Messages from "orders" Kafka topic

Function:

Deserializes JSON → OrderEvent
Checks for duplicate orders (idempotency)
Orchestrates processing pipeline:
Converts OrderEvent → Order entity
Saves to database via OrderRepository
Updates inventory
Sends notifications
Output:

Success: Order persisted in DB
Failure: Message moved to DLQ
2. OrderRepository.java
Input: Order entity from Consumer

Function:

Handles all database operations
Uses pessimistic locking for concurrent updates
Implements idempotency check
Output: Order records in MySQL

3. Order.java & OrderItem.java
Role:

Order: Main entity with customer/shipping details
OrderItem: Individual products in the order
Function: Map Kafka messages to persistent entities

Supporting Modules
OrderRequest.java
Role: API request schema

Contains: Raw frontend input before validation

OrderResponse.java
Role: API response format

Contains: Order number + status for frontend

Error Handling
DLQ Service: Manages failed messages

Retry Logic: Exponential backoff for transient failures

End-to-End Flow
Frontend submits order → OrderController
Controller validates → KafkaProducerService
Producer publishes → Kafka "orders" topic
Consumer reads → Processes via:
OrderRepository (persistence)
InventoryService (stock update)
NotificationService (emails)
Results:

Success: Order in DB + confirmation

Failure: DLQ entry + alerts

Key Characteristics
Module	Responsibility	Data Flow
OrderController	API Gateway	HTTP → Kafka
KafkaProducerService	Reliable message publishing	Java → Kafka
OrderConsumerService	Transactional processing	Kafka → DB
OrderRepository	Data persistence	Objects → MySQL
DTOs (OrderEvent)	Schema enforcement	Inter-service contracts
This architecture ensures:

Frontend gets instant response

Orders are never lost (Kafka retention)

Database stays consistent

Services remain loosely coupled

Components Involved
OrderController.java - Receives order submissions
KafkaProducerService.java - Publishes order events
OrderConsumerService.java - Processes order events
OrderRepository.java - Saves orders to database
Order.java, OrderItem.java - Order entity models
Workflow
Order Submission

Frontend POSTs order data to /api/orders
Includes customer info, items, and payment method
Initial Validation

Validates required fields
Generates unique order number
Creates OrderEvent DTO
Event Publishing

Serializes to JSON
Publishes to "orders" Kafka topic
Immediately returns 200 response
Background Processing
Consumer service:

Listens to "orders" topic
Converts DTO to JPA entity
Persists to database
Handles automatic retries
Key Characteristics
Immediate frontend acknowledgment
Message durability during outages
Parallel processing capability
Automatic failure recovery
Failure Handling
Scenario	Product Search	Order Processing
Database unavailable	Immediate error to user	Orders queue in Kafka
High traffic	Slower responses	Processes during lulls
Validation failure	Instant feedback	Notification system
Monitoring
Product Search
API response times
Database query performance
Error rate monitoring
Order Processing
Kafka consumer lag
End-to-end processing time
Failed order rate
Consumer health checks
Key Benefits
Performance: Fast browsing despite order volume
Reliability: No lost orders during outages
Scalability: Independent scaling of components
Maintainability: Clear separation of concerns
