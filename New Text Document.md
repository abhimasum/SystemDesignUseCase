# System Design Masterclass: Industry Scenarios & Architectures

## 1. Smart Home Automation (IoT Edge & Batching)
**The Problem:** Streaming millions of sensor pings per second directly to the cloud is astronomically expensive and fails when the internet drops.
**The Solution:** Use Edge computing with a bounded buffer. Process real-time triggers locally, and batch historical data for the cloud.

**Architecture Diagram:**
[Sensors] --> [Edge Gateway & In-Memory Buffer] --> Local DB [SQLite]
                                                        | (Every 10 min)
                                                        v
[Blob Storage [AWS: Amazon Simple Storage Service (S3), Azure: Azure Blob Storage, GCP: Google Cloud Storage]]
                                                        |
[Message Queue [AWS: Amazon Simple Queue Service (SQS), Azure: Azure Queue Storage, GCP: Google Cloud Pub/Sub]]
                                                        |
[Worker VMs / Containers] --> [Time-Series DB [AWS: Amazon Timestream, Azure: Azure Data Explorer, GCP: Google Cloud Bigtable]]

---

## 2. Ride-Hailing GPS Tracking (Uber/Ola)
**The Problem:** 100,000 drivers sending GPS pings every 5 seconds will crash standard REST APIs and relational databases.
**The Solution:** Use high-throughput event streaming for ingestion, a geospatial in-memory cache for live maps, and batch storage for billing.

**Architecture Diagram:**
[Driver App] --> [API Gateway] --> [Event Stream [AWS: Amazon Kinesis, Azure: Azure Event Hubs, GCP: Google Cloud Pub/Sub]]
                                      |
       +------------------------------+-------------------------------+
       v (Fast Path)                                                  v (Slow Path)
[In-Memory Cache [AWS: Amazon ElastiCache (Redis),          [Blob Storage [AWS: Amazon S3, Azure: Azure Blob, 
 Azure: Azure Cache for Redis, GCP: Google Cloud Memorystore]]     GCP: Google Cloud Storage]] --> [Nightly Billing DB]

---

## 3. High-Concurrency Ticket Booking (BookMyShow)
**The Problem:** 1 million users trying to buy 50,000 seats simultaneously will crash servers (Thundering Herd) and cause double-bookings.
**The Solution:** Block 95% of traffic at the CDN (Waiting Room) and use single-threaded distributed locks with time-to-live (TTL).

**Architecture Diagram:**
[1M Users] --> [CDN / Edge Network [AWS: Amazon CloudFront, Azure: Azure Front Door, GCP: Cloud CDN]] (Waiting Room)
                    | (Allows 50k users)
                    v
[Web Servers] --> [In-Memory Cache (Redis)] (Uses SETNX for 5-minute Seat Locks)
                    | (On Payment Success)
                    v
[Relational DB [AWS: Amazon Aurora (PostgreSQL), Azure: Azure Database for PostgreSQL, GCP: Cloud SQL]]

---

## 4. E-Commerce Search (Amazon)
**The Problem:** Searching 100 million products with typos and filters using SQL `LIKE` takes 10+ seconds and crashes the database.
**The Solution:** Use an Inverted Index (Search Engine) and sync data asynchronously using Change Data Capture (CDC).

**Architecture Diagram:**
[Primary Relational DB] --> [CDC Tool (Debezium)] --> [Event Stream [AWS: Amazon Managed Streaming for Apache Kafka (MSK), Azure: Azure Event Hubs, GCP: Google Cloud Pub/Sub]]
                                                              |
                                                              v
[User Search Query] ---------------------> [Full-Text Search Engine [AWS: Amazon OpenSearch Service, Azure: Azure AI Search, GCP: Google Cloud Search]]

---

## 5. Social Media News Feed (Twitter)
**The Problem:** Pushing a celebrity's tweet (100M followers) to individual feeds crashes the write database. Pulling everyone's tweets crashes the read database.
**The Solution:** Hybrid Fan-out. Push normal user tweets; pull celebrity tweets on the fly.

**Architecture Diagram:**
[Tweet] --> Is Celebrity? 
            ├── YES --> [Global Cache [AWS: Amazon ElastiCache, Azure: Azure Cache for Redis, GCP: Cloud Memorystore]]
            └── NO  --> [Message Queue] --> Push to Follower's [Personal In-Memory Feed Cache]
            
[App Opens] --> Merges [Personal Feed] + [Celebrity Cache] in memory < 100ms.

---

## 6. Digital Wallet Transactions (FinTech / Saga Pattern)
**The Problem:** Transferring money across distributed microservices risks losing money if the network fails midway.
**The Solution:** Use an Orchestrator (Saga Pattern) with compensating transactions (refunds) and Idempotency Keys to prevent double-charging.

**Architecture Diagram:**
[Transaction Orchestrator]
       ├── 1. Deduct $50 --> [Wallet DB A] (Success)
       └── 2. Add $50    --> [Wallet DB B] (Fails!)
       └── 3. Compensation -> [Wallet DB A] (Refund $50 automatically)

---

## 7. Distributed API Rate Limiter
**The Problem:** Hackers sending 10,000 requests/sec across 50 load-balanced servers.
**The Solution:** Use the Token Bucket algorithm stored in a centralized, ultra-fast in-memory cache.

**Architecture Diagram:**
[API Servers (x50)] --> [In-Memory Cache [AWS: Amazon ElastiCache, Azure: Azure Cache for Redis, GCP: Cloud Memorystore]]
                          (Stores: { "user_123_tokens": 99 }) -> Rejects if 0.

---

## 8. Video Streaming (YouTube)
**The Problem:** Serving large 5GB video files directly from one server causes buffering and maxes out bandwidth.
**The Solution:** Transcode video into multiple resolutions, slice into 10-second chunks, and cache globally.

**Architecture Diagram:**
[Raw Video] --> [Transcoder / Media Service [AWS: AWS Elemental MediaConvert, Azure: Azure Media Services, GCP: Transcoder API]]
                    | (Generates 10s chunks)
                    v
[CDN / Edge Network [AWS: Amazon CloudFront, Azure: Azure Front Door, GCP: Cloud CDN]] --> [User Device]

---

## 9. Real-Time Chat (WhatsApp)
**The Problem:** HTTP polling crashes servers. Users are connected to different physical servers.
**The Solution:** Persistent WebSockets and a Pub/Sub routing bus.

**Architecture Diagram:**
[User A] <--(WebSocket)--> [Chat Server 1] 
                               |
                        [Pub/Sub Message Bus (Redis/Kafka)] <--(Routes message)--> [Chat Server 2] <--(WebSocket)--> [User B]

---

## 10. Enterprise RAG (Secure AI Chatbot)
**The Problem:** The LLM hallucinates, cannot fit 1M documents in context, and might leak CEO secrets to interns.
**The Solution:** Chunking, Embeddings, and Vector Databases with strict Metadata Filtering.

**Architecture Diagram:**
[User Query] --> [Embedding Model] --> [Vector DB [AWS: Amazon OpenSearch Serverless (Vector Engine), Azure: Azure Cosmos DB for PostgreSQL (pgvector), GCP: Vertex AI Vector Search]]
                                          (Filters by user_clearance_level)
                                          | (Returns 5 safe paragraphs)
                                          v
                              [LLM [AWS: Amazon Bedrock, Azure: Azure OpenAI, GCP: Vertex AI]]

---

## 11. Autonomous AI Agents (Customer Support)
**The Problem:** LLMs are text generators and cannot execute real-world actions safely.
**The Solution:** Tool Use (Function Calling), Multi-Agent orchestration, and Human-in-the-Loop gateways.

**Architecture Diagram:**
[Email] --> [Orchestrator Agent] --> Routes to --> [Logistics Agent] (Calls FedEx API Tool)
                                                        |
                                                        v
[Human-in-the-Loop Gateway] <--- [Finance Agent] (Calls Stripe Refund Tool)

---

## 12. AI Image Generation (Async Queue)
**The Problem:** Generating images takes 15 seconds. Standard HTTP APIs time out. You have 50k requests but only 500 GPUs.
**The Solution:** Asynchronous API, message queues, and worker pulling.

**Architecture Diagram:**
[User] --> [API] (Returns 202 Accepted & Job_ID) --> [Message Queue [AWS: Amazon SQS, Azure: Azure Service Bus, GCP: Cloud Pub/Sub]]
                                                             | (Pulled by)
[User UI Polls Status] <--- [Database] <--- [GPU Workers] (Upload image to Blob Storage)

---

## 13. Collaborative Editor (Google Docs)
**The Problem:** Database locks prevent live typing. Network latency causes screens to go out of sync.
**The Solution:** Operational Transformation (OT) or CRDTs via WebSockets.

**Architecture Diagram:**
[User A] (Sends: Insert 'x' at index 3) ---> [Collaboration Node (In-Memory)] <--- [User B] (Sends: Insert 'y' at index 3)
                                                 (Transforms math indices)
                                                 (Broadcasts merged state)

---

## 14. Video Recommendation (TikTok)
**The Problem:** Scoring 50 million videos with a Deep Neural Network for every swipe takes too long.
**The Solution:** A two-stage funnel (Candidate Generation + Heavy Ranking) updated by real-time stream processing.

**Architecture Diagram:**
[User Swipe] -> [Event Stream] -> Updates User Profile in Cache.
[Stage 1: Fast Retrieval] -> Pulls 1,000 videos from Vector DB.
[Stage 2: Heavy AI Ranker] -> Scores 1,000 videos, returns top 5 to user.
