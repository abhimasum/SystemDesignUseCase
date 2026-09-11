# Cloud Architecture Practice Guide for AI Coding Agents (GitHub Copilot, Cursor, Codeium)

Use this guide to practice building the backend infrastructure for the scenarios we discussed. Copy the **Prompt Templates** below and paste them into your AI coding agent to start generating real, deployable code.

## 1. IoT Edge Batching to Cloud
**Tech Stack:** Python, SQLite, AWS Boto3 (S3), AWS SQS
**AI Prompt Template:**
> "Act as a Senior Cloud Engineer. I want to build a simulated IoT pipeline. First, write a Python script that runs locally, generates mock temperature/occupancy data, batches it into an in-memory queue of 60 records, and saves it to a local SQLite database using WAL mode. Second, write a cron-job script that compresses the SQLite file, uploads it to AWS S3, and deletes the local file only upon a 200 OK response."

## 2. Distributed Ticket Booking (Redis Locks)
**Tech Stack:** Node.js / Express, Redis (ioredis), PostgreSQL
**AI Prompt Template:**
> "I am building a high-concurrency ticket booking backend. Create a Node.js Express endpoint `/book-seat`. Connect to a local Redis instance. Implement a Distributed Lock using Redis `SETNX` with a TTL of 300 seconds for a specific `seat_id`. If the lock is acquired, return success. If the seat is already locked, immediately return an HTTP 409 Conflict. Ensure the code handles edge cases where Redis might fail."

## 3. FinTech Wallet (Saga Pattern & Idempotency)
**Tech Stack:** Go or Python (FastAPI), PostgreSQL
**AI Prompt Template:**
> "Design a basic Saga Orchestrator in Python for a wallet transfer system. Create an endpoint `/transfer`. The code must enforce Idempotency by checking an `idempotency_key` header against a database table before proceeding. Simulate a failure in 'Step 2' (Adding funds to User B) and write the compensating transaction logic to automatically refund User A."

## 4. E-Commerce CDC Pipeline
**Tech Stack:** Docker Compose, PostgreSQL, Debezium, Kafka, Elasticsearch
**AI Prompt Template:**
> "Write a complete `docker-compose.yml` file that sets up a local environment for Change Data Capture (CDC). It needs PostgreSQL, Apache Kafka, Zookeeper/KRaft, Debezium, and Elasticsearch. Then, write a brief README explaining how to configure Debezium to watch a `products` table in Postgres and automatically stream row updates into an Elasticsearch index."

## 5. Enterprise RAG (LangChain / Vector DB)
**Tech Stack:** Python, LangChain, OpenAI API, ChromaDB or pgvector
**AI Prompt Template:**
> "I want to build a secure Enterprise RAG pipeline using LangChain. Write a Python script that takes a sample text document, chunks it using RecursiveCharacterTextSplitter, and embeds it into a local Vector DB. Crucially, show me how to add metadata to these chunks (e.g., `clearance_level: 'tier_3'`). Finally, write the retrieval function that strictly filters by this metadata before passing the context to the LLM."

## 6. Async Image Generation Queue
**Tech Stack:** Python (FastAPI), Celery, Redis, AWS S3
**AI Prompt Template:**
> "Implement an asynchronous API pattern using FastAPI and Celery. Create a `POST /generate` endpoint that accepts a prompt, instantly returns a `202 Accepted` with a `job_id`, and pushes the task to a Celery worker. Write the Celery worker function to simulate a 10-second sleep (mock GPU generation), generate a dummy text file, upload it to AWS S3 using boto3, and update the task status in a database."
