## 🍔 **GrubDash: Scalable Cloud-Based Food Delivery Application**

![Architecture Diagram](https://github.com/satyakamala03/GrubDash/blob/main/assets/ArchitectureDiagram.png)

[![Watch the demo](https://img.youtube.com/vi/n3nJnzkFb3k/0.jpg)](https://www.youtube.com/watch?v=n3nJnzkFb3k)


### **Overview**

**GrubDash** is a serverless, event-driven, and highly scalable food delivery platform deployed on **AWS**.
It integrates microservices for customers, restaurants, and delivery partners, ensuring seamless communication and real-time order tracking.
The application handles complex workflows like **payment processing**, **order batching**, **delivery partner assignment**, and **live delivery tracking**, all powered by **AWS Lambda** and **event-driven architecture**.

---

## ⚙️ **High-Level Architecture**

GrubDash is composed of four core services:

1. **Signup/Login Service**
2. **Customer Service**
3. **Order Management Service**
4. **Delivery Management Service**
5. **Recommendation Engine**

Each service operates independently but communicates through **API Gateway**, **SQS FIFO queues**, and **WebSocket APIs** for asynchronous, real-time event handling.

---

## 🧩 **Service-Level Breakdown**

### 1. **Signup/Login Service**

* **Purpose:** Handles user authentication and authorization.
* **Flow:**

  * Users log in or sign up via **Amazon Cognito**.
  * Cognito issues **JWT tokens** for session management.
  * User details and roles are stored in **DynamoDB**.
* **Key AWS Components:**

  * Amazon Cognito
  * AWS Lambda
  * Amazon DynamoDB

---

### 2. **Customer Service**

* **Purpose:** Enables customers to browse restaurants, view menus, and manage profiles.
* **Flow:**

  * Performs CRUD operations via **Lambda** and **API Gateway**.
  * Fetches restaurant data and customer preferences from **DynamoDB**.
  * Integrates with the **Recommendation Engine** for personalized results.
* **Key AWS Components:**

  * AWS Lambda
  * Amazon API Gateway
  * Amazon DynamoDB
  * Amazon S3 (for static assets)

---

### 3. **Order Management Service**

* **Purpose:** Manages the order lifecycle — from payment initiation to restaurant confirmation.
* **Flow:**

  1. **Checkout & Payment**

     * User initiates checkout → API Gateway → Lambda → Stripe payment intent creation.
     * Stripe returns `client_secret`, which is passed back to frontend.
  2. **Payment Confirmation**

     * Stripe webhook triggers **Payments Processor Lambda** → updates **Orders DB**.
  3. **Order Processing**

     * Each order event is pushed into a **FIFO SQS Queue** for strict sequencing.
     * Orders Processor Lambda updates status in DynamoDB and triggers notifications.
* **Key AWS Components:**

  * Amazon SQS (FIFO Queue)
  * AWS Lambda
  * Amazon DynamoDB
  * Amazon API Gateway
  * WebSocket API (for real-time status updates)

---

### 4. **Delivery Management Service**

* **Purpose:** Manages delivery partner assignment, tracking, and status updates.
* **Flow:**

  1. **Batching Process**

     * After restaurants confirm orders, the system groups them geographically.
     * Uses **Redis (Elasticache)** to store restaurant coordinates and available delivery partners.
  2. **DP Assignment**

     * **Round-robin scheduling** and **geo radius search** determine the most suitable delivery partner.
     * Once assigned, order status updates to *DP_Assigned*.
  3. **Delivery Partner Simulation**

     * Simulated via **Lambda**, mimicking live movement updates to Redis.
     * “Start live tracking” sends periodic location updates through Lambda.
  4. **Completion**

     * When marked “Delivered,” DP is removed from the Redis cache and marked offline.
     * Offline event triggers **Delivery Events Processor Lambda** to finalize updates in DB.
* **Key AWS Components:**

  * AWS Lambda
  * Amazon SQS (FIFO Queue)
  * Amazon ElastiCache (Redis)
  * Amazon DynamoDB

---

### 5. **Recommendation Engine**

* **Purpose:** Provides personalized restaurant or dish recommendations based on user behavior.
* **Flow:**

  * Logs user interactions through **Lambda**.
  * Stores aggregated data in **S3** and **DynamoDB**.
  * Lambda fetches and serves recommendations in real-time.
* **Key AWS Components:**

  * AWS Lambda
  * Amazon S3
  * Amazon DynamoDB

---

## 🛰️ **Detailed System Workflow**

### **1. Order Creation and Payment**

* User initiates checkout via **API Gateway**.
* Lambda creates **Stripe payment intent** and returns `client_secret`.
* Frontend confirms payment using Stripe SDK.
* Stripe webhook → Lambda → updates payment status and pushes to **Order Events FIFO Queue**.

### **2. Order Confirmation and Batching**

* Restaurants confirm or reject orders.
* Confirmed orders are sent to **Batching Queue**.
* Batching Lambda groups orders based on pickup zones using **Redis**.
* If batching fails, message retries (up to `MAX_RETRIES`) before reassignment.

### **3. Delivery Partner Assignment**

* DP assignment Lambda performs **geo radius search + weighted round robin**:

  * **Higher weight:** closer proximity, recently seen
  * **Lower weight:** recently assigned
* Once DP accepts → order state changes to *DP_Assigned* → customer sees update.

### **4. Live Tracking Simulation**

* DP’s simulated location updates are triggered via Lambda to Redis.
* Redis updates push live coordinates to frontend map (simulated WebSocket).
* When delivery completes → Redis entry removed → DP marked offline.

### **5. Delivery Completion**

* Delivery completion triggers **Delivery Events Processor Lambda**.
* Updates delivery DB and sends notification to customer and restaurant.

---

## ☁️ **AWS Services Summary**

| **Service**                    | **Purpose**                                                           |
| ------------------------------ | --------------------------------------------------------------------- |
| **AWS Lambda**                 | Core compute layer for all backend logic (stateless microservices).   |
| **Amazon API Gateway**         | Exposes REST and WebSocket endpoints for frontend communication.      |
| **Amazon DynamoDB**            | Stores user profiles, orders, restaurant data, and delivery statuses. |
| **Amazon SQS (FIFO Queues)**   | Ensures ordered message processing for state transitions.             |
| **Amazon ElastiCache (Redis)** | Used for delivery partner tracking, geo-search, and caching.          |
| **Amazon Cognito**             | Handles user authentication and JWT-based authorization.              |
| **Amazon S3**                  | Stores static content, logs, and recommendation datasets.             |
| **Stripe API (External)**      | Handles secure payment intent creation and confirmation.              |

---

## 🚀 **Key Highlights**

* Fully **serverless architecture** with auto-scaling capabilities.
* **FIFO queues** guarantee event ordering and consistent state transitions.
* **Redis caching** ensures high-speed delivery partner lookup and batching efficiency.
* **Simulated real-time tracking** demonstrates scalability and event-driven updates.
* **Decoupled microservices** make the system resilient and easy to maintain.


