# AlgoCode Evaluator Service

## Overview

AlgoCode Evaluator Service is a scalable, secure backend system designed to dynamically evaluate user-submitted algorithms and code. Built with Node.js and TypeScript, the service leverages a message queue for high-throughput submission handling and Docker for sandboxed, isolated code execution.

## High-Level Design (HLD)

The system architecture follows a decoupled, asynchronous producer-consumer model to guarantee fault tolerance and scalability under heavy submission loads.

* **API Gateway (Producer):** An Express.js REST interface that receives code submissions, validates the payload, and enqueues the job.
* **Message Queue:** Buffers incoming requests, ensuring the evaluation workers are not overwhelmed during traffic spikes.
* **Worker Nodes (Consumer):** Processes pulled from the queue that orchestrate the execution lifecycle.
* **Sandboxed Execution Environment:** Ephemeral Docker containers are provisioned for each job to safely execute untrusted user code (currently supporting Python) with strict hardware constraints, such as time and memory limits.

## Tech Stack

* **Runtime & Framework:** Node.js, Express.js
* **Language:** TypeScript
* **Containerization:** Docker
* **Code Quality & Tooling:** ESLint, Prettier, tsconfig

## Core Implementation Details

* **Submission Queueing:** Jobs are delegated to a queue layer. This allows the main server thread to remain unblocked, ensuring rapid API response times and preventing request timeouts.
* **Docker Integration:** The application programmatically interfaces with the Docker daemon to spin up isolated containers, map execution files, run commands, and extract standard output and standard error streams.
* **Payload Validation:** Incoming API requests pass through a strict validation layer (via a validator middleware) to reject malformed or potentially malicious data before processing.
* **Stateless Architecture:** The core service is designed to be completely stateless. It can be horizontally scaled by simply provisioning additional worker nodes and Docker host instances.

## How It Works (Execution Flow)

1. **Ingestion:** A client submits a code payload and language identifier via the API.
2. **Validation:** The service checks the payload against schema requirements and language support policies.
3. **Queueing:** The validated payload is pushed to the internal submission queue, and the API immediately responds with an acknowledgment.
4. **Provisioning:** A worker node consumes the task from the queue and initializes a sandboxed Docker container tailored to the requested language environment.
5. **Execution:** The user's code is executed inside the container against the provided test cases.
6. **Teardown & Verdict:** Execution outputs (stdout/stderr) are captured and evaluated. The container is immediately destroyed to free system resources, and the final verdict (e.g., Accepted, Runtime Error, Time Limit Exceeded) is recorded.

## API Documentation

### 1. Health Check

Verifies the active status and availability of the evaluator service.

* **Endpoint:** `GET /ping`
* **Response:** `200 OK`

### 2. Submit Code

Submits a new algorithm for asynchronous evaluation.

* **Endpoint:** `POST /submit`
* **Payload:**
```json
{
  "language": "python",
  "code": "print('Hello World')",
  "testCases": [...]
}

```


* **Response:** `202 Accepted` (Returns a job tracking identifier)

## Setup and Installation

### Prerequisites

* Node.js (v18 or higher recommended)
* Docker Daemon (Must be actively running on the host machine)
* npm

### Local Initialization

1. **Install Dependencies:**
```bash
npm install

```


2. **Environment Configuration:** Create necessary environment configurations as defined in `Setup.md`.
3. **Compile TypeScript:**
```bash
npm run build

```


4. **Start the Service:**
```bash
npm start

```


5. **Linting & Formatting (Optional):**
```bash
npm run lint

```
