# Splunk Log Collector

[![Python Version](https://img.shields.io/badge/python-3.11.6-blue.svg)](https://www.python.org/)
[![AWS SAM](https://img.shields.io/badge/AWS%20SAM-Serverless%20Application%20Model-orange.svg)](https://aws.amazon.com/serverless/sam/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](#)

The **Splunk Log Collector** is a Python-based solution designed to collect identity logs from multiple sources (e.g., Ping, MyID, Okta) via Splunk's REST API. It consolidates logs from different identity sources, processes them, and sends the data to AWS EventBridge and S3 for further analysis.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
  - [Architecture Diagram](#architecture-diagram)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [AWS Deployment](#aws-deployment)
- [Usage](#usage)
  - [Triggering the Collector](#triggering-the-collector)
  - [Sample Events](#sample-events)
  - [Sample Step Function Execution Flow](#sample-step-function-execution-flow)
- [Configuration](#configuration)
  - [AWS Secrets Manager](#aws-secrets-manager)
  - [Splunk Queries](#splunk-queries)
  - [Environment Variables](#environment-variables)
- [Repository Structure](#repository-structure)
  - [Directory and File Descriptions](#directory-and-file-descriptions)
- [Testing](#testing)
  - [Running Tests](#running-tests)
  - [Generating Coverage Reports](#generating-coverage-reports)
  - [Writing Tests](#writing-tests)
  - [Mocking Services](#mocking-services)
- [Development Guidelines](#development-guidelines)
  - [Code Style](#code-style)
  - [Git Workflow](#git-workflow)
  - [Dependency Management](#dependency-management)
  - [AWS Lambda Powertools](#aws-lambda-powertools)
- [Mermaid.js Flowchart Diagrams](#mermaidjs-flowchart-diagrams)
  - [High-Level System Architecture](#high-level-system-architecture)
  - [AWS Step Functions Workflow](#aws-step-functions-workflow)
  - [Lambda Function Interactions](#lambda-function-interactions)
  - [Class Inheritance Diagram](#class-inheritance-diagram)

---

## Overview

The **Splunk Log Collector** streamlines the collection of identity logs from multiple sources using Splunk's REST API. By consolidating logs from various identity sources, it enables centralized analysis and real-time processing, enhancing scalability and ease of adding new identity sources.

**Key Objectives:**

- **Consolidate Identity Logs:** Unify logs from various identity sources for centralized analysis.
- **Real-Time Processing:** Enable near real-time log collection and processing.
- **Scalability:** Efficiently handle large volumes of data.
- **Extensibility:** Easily add support for new identity sources.

## Architecture

The system architecture leverages several AWS services to orchestrate, process, and store log data efficiently.

- **AWS Step Functions (State Machine):** Orchestrates the workflow of initiating Splunk queries, polling for job completion, and processing results.
- **AWS Lambda Functions:** Handle communication with Splunk, process data, and send it to AWS EventBridge and S3.
- **Splunk REST API:** Used to query and retrieve logs from Splunk instances.
- **AWS EventBridge:** Receives processed events from Lambda functions for downstream processing.
- **AWS S3 Buckets:** Stores raw logs from each identity source.
- **AWS Secrets Manager:** Securely stores credentials for accessing Splunk instances.
- **AWS Lambda Powertools:** Utilized for logging, metrics, and tracing within Lambda functions.

### Architecture Diagram

```mermaid
flowchart TD
    subgraph AWS
        A[Step Functions State Machine] -->|Start Query| B[Lambda: Initiate Splunk Job]
        B --> C[Lambda: Poll for Job Completion]
        C -->|Job Ready| D[Lambda: Process Results]
        D --> E[EventBridge]
        D --> F[S3 Bucket]
    end
    D -->|Store Logs| F
    D -->|Send Events| E
    B --- SSM[AWS Secrets Manager]
    B -->|Retrieve Credentials| SSM
    B -->|Splunk REST API| Splunk[Splunk Instances]
    C -->|Check Job Status| Splunk
    D -->|Fetch Results| Splunk
```

## Features

- **Real-Time Log Collection:** Collect logs in near real-time by polling Splunk for job completion and reading results in chunks.
- **Scalable Design:** Efficiently handle large volumes of data by chunking results and managing concurrent read requests.
- **Extensible Collectors:** Easily add new identity sources by extending the generic collector.
- **PEP8 Compliant Code:** Follows Python's PEP8 style guidelines, enforced via linters like Flake8 and Black.
- **Automated Testing:** Uses `pytest`, `pytest-cov`, and `moto` for unit tests and coverage.
- **Dependency Management:** Managed via Poetry, ensuring consistent environments across development and deployment.

## Technologies Used

- **Python 3.11.6**
- **AWS Lambda**
- **AWS Step Functions**
- **AWS EventBridge**
- **AWS S3**
- **AWS Secrets Manager**
- **AWS Lambda Powertools**
- **Splunk REST API**
- **Poetry** for dependency management
- **AWS SAM CLI** for deployment
- **Pytest** for testing
- **Moto** for mocking AWS services
- **Flake8** and **Black** for code style and formatting

## Getting Started

### Prerequisites

- **Python 3.11.6**
- **AWS Account** with permissions to deploy resources (Lambda, Step Functions, EventBridge, S3, Secrets Manager).
- **Access to Splunk Instances** with REST API enabled.
- **Poetry** for dependency management.
- **AWS SAM CLI** for deployment.
- **Git**

### Installation

1. **Clone the Repository**

   ```bash
   git clone https://github.com/yourusername/splunk-log-collector.git
   cd splunk-log-collector
   ```

2. **Install Poetry**

   Follow the instructions on the [Poetry website](https://python-poetry.org/docs/#installation) to install Poetry.

3. **Install Dependencies**

   ```bash
   poetry install
   ```

4. **Activate the Virtual Environment**

   ```bash
   poetry shell
   ```

5. **Set Up Environment Variables**

   Create a `.env` file or export the following environment variables:

   ```bash
   export AWS_ACCESS_KEY_ID=<your-access-key-id>
   export AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
   export AWS_DEFAULT_REGION=<your-aws-region>
   ```

### AWS Deployment

This project uses AWS SAM (Serverless Application Model) for deployment.

1. **Build the SAM Application**

   ```bash
   sam build
   ```

2. **Deploy the SAM Application**

   ```bash
   sam deploy --guided
   ```

   During deployment, you will be prompted to specify:

   - **Stack Name:** e.g., `splunk-log-collector-stack`
   - **AWS Region:** e.g., `us-east-1`
   - **Confirm Changes Before Deploying:** `Y`
   - **Allow SAM CLI IAM Role Creation:** `Y`
   - **Save Arguments to samconfig.toml:** `Y`

3. **Post-Deployment Configuration**

   - **AWS Secrets Manager:** Ensure your Splunk credentials are stored with the correct secret names.
   - **EventBridge Configuration:** Verify that EventBridge rules are set up correctly to receive events.

## Usage

### Triggering the Collector

The log collection process is initiated via the AWS Step Functions State Machine. You can trigger it manually or set up an AWS CloudWatch Event rule to trigger it on a schedule.

**Manual Trigger:**

1. Navigate to the AWS Step Functions console.
2. Select the **SplunkLogCollectorStateMachine**.
3. Click on **Start Execution**.
4. Provide the necessary input (see sample below) and click **Start Execution**.

### Sample Events

```json
{
  "secret_name": "splunk/ping",
  "query": "search index=ping_identity sourcetype=ping:logs earliest=-15m@m latest=now",
  "timeframe": "-15m@m to now"
}
```

### Sample Step Function Execution Flow

```mermaid
stateDiagram
    [*] --> InitiateSplunkJob
    InitiateSplunkJob --> PollJobStatus
    PollJobStatus --> PollJobStatus: Job Not Ready
    PollJobStatus --> ProcessResults: Job Ready
    ProcessResults --> SendToEventBridge
    SendToEventBridge --> StoreInS3
    StoreInS3 --> [*]
```

## Configuration

### AWS Secrets Manager

Store your Splunk credentials securely in AWS Secrets Manager.

- **Secret Name:** As specified in your configuration (e.g., `splunk/ping`, `splunk/myid`).
- **Secret Value (JSON):**

  ```json
  {
    "username": "your-splunk-username",
    "password": "your-splunk-password",
    "host": "your-splunk-host",
    "port": "8089"
  }
  ```

### Splunk Queries

Configure Splunk queries for each identity source in the `config.py` file or via environment variables.

```python
# src/utils/config.py

QUERIES = {
    "ping": "search index=ping_identity sourcetype=ping:logs earliest=-15m@m latest=now",
    "myid": "search index=myid_identity sourcetype=myid:logs earliest=-15m@m latest=now"
}
```

### Environment Variables

Set additional environment variables required for your application in the AWS Lambda function configuration or in a `.env` file for local testing.

- **LOG_LEVEL:** Set the logging level (e.g., `INFO`, `DEBUG`).
- **MAX_CONCURRENT_READS:** Maximum number of concurrent read requests to Splunk.

## Repository Structure

```plaintext
splunk-log-collector/
├── README.md
├── .gitignore
├── .flake8
├── pyproject.toml
├── poetry.lock
├── template.yaml             # AWS SAM template
├── src/
│   ├── collector/
│   │   ├── __init__.py
│   │   ├── generic_collector.py
│   │   ├── ping_collector.py
│   │   ├── myid_collector.py
│   │   └── handler.py        # Lambda function entry points
│   └── utils/
│       ├── __init__.py
│       ├── config.py
│       └── logger.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py           # Fixtures for tests
│   ├── test_generic_collector.py
│   ├── test_ping_collector.py
│   └── test_myid_collector.py
└── docs/
    ├── architecture.md
    ├── development_guide.md
    └── images/
        └── architecture.png
```

### Directory and File Descriptions

- **`src/`**: Contains the source code.
  - **`collector/`**: Modules for the generic and source-specific collectors.
    - **`generic_collector.py`**: Implements the base collector class.
    - **`ping_collector.py`**: Extends the generic collector for Ping.
    - **`myid_collector.py`**: Extends the generic collector for MyID.
    - **`handler.py`**: Entry points for AWS Lambda functions.
  - **`utils/`**: Utility modules.
    - **`config.py`**: Configuration settings and constants.
    - **`logger.py`**: Logging configuration using AWS Lambda Powertools.
- **`tests/`**: Contains unit tests.
  - **`conftest.py`**: Fixtures and common test utilities.
- **`docs/`**: Additional documentation.
  - **`architecture.md`**: Detailed architecture documentation.
  - **`development_guide.md`**: Guidelines for development.

## Testing

We use `pytest` for running tests and `pytest-cov` for coverage reports.

### Running Tests

```bash
poetry run pytest
```

### Generating Coverage Reports

```bash
poetry run pytest --cov=src --cov-report=html
```

Open `htmlcov/index.html` in a browser to view the coverage report.

### Writing Tests

- **Unit Tests:** Should cover all functions and methods.
- **Coverage Requirement:** Aim for at least **85%** code coverage.

### Mocking Services

We use `moto` to mock AWS services and `unittest.mock` for external services like Splunk.

```python
# tests/test_generic_collector.py

from moto import mock_secretsmanager
from unittest.mock import patch

@mock_secretsmanager
def test_get_splunk_credentials():
    # Test implementation
```

## Development Guidelines

### Code Style

- **PEP8 Compliance:** Code must adhere to PEP8 standards.
- **Linters:**
  - **Flake8:** For code style checks.

    ```bash
    poetry run flake8 src/
    ```

  - **Black:** For code formatting.

    ```bash
    poetry run black src/
    ```

- **Docstrings:** Use consistent and clear docstrings for modules, classes, and functions.

### Git Workflow

- **Branching Strategy:** Follow GitFlow model.
  - **Main Branches:**
    - `main` for production-ready code.
    - `develop` for integration of features.
  - **Supporting Branches:**
    - `feature/*` for new features.
    - `bugfix/*` for fixes.
- **Commit Messages:** Use Conventional Commits format.
  - **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
  - **Example:** `feat(collector): add support for new identity source`
- **Pull Requests:**
  - Ensure all tests pass.
  - Maintain or improve code coverage.
  - Request at least one code review before merging.
- **Code Reviews:**
  - Use GitLab's merge request system.
  - Address all comments before approval.

### Dependency Management

- **Adding Dependencies:**

  ```bash
  poetry add <package-name>
  ```

- **Adding Dev Dependencies:**

  ```bash
  poetry add --group dev <package-name>
  ```

### AWS Lambda Powertools

Utilize AWS Lambda Powertools for structured logging, metrics, and tracing.

```python
# src/collector/handler.py

from aws_lambda_powertools import Logger, Metrics, Tracer

logger = Logger(service="splunk-log-collector")
metrics = Metrics(namespace="SplunkLogCollector")
tracer = Tracer()

@logger.inject_lambda_context(log_event=True)
@metrics.log_metrics
@tracer.capture_lambda_handler
def lambda_handler(event, context):
    # Handler code
```

## Mermaid.js Flowchart Diagrams

### High-Level System Architecture

```mermaid
flowchart TD
    subgraph AWS
        A[Step Functions State Machine] -->|Start Query| B[Lambda: Initiate Splunk Job]
        B --> C[Lambda: Poll for Job Completion]
        C -->|Job Not Ready| C
        C -->|Job Ready| D[Lambda: Process Results]
        D -->|Parse and Send Events| E[AWS EventBridge]
        D -->|Store Raw Logs| F[S3 Bucket]
        D -->|Cleanup| G[Cancel Splunk Job]
    end
    B --- SSM[AWS Secrets Manager]
    B -->|Retrieve Credentials| SSM
    B -->|Initiate Job| Splunk[Splunk REST API]
    C -->|Check Job Status| Splunk
    D -->|Fetch Results| Splunk
```

### AWS Step Functions Workflow

```mermaid
stateDiagram-v2
    [*] --> InitiateSplunkJob
    InitiateSplunkJob --> PollJobStatus
    PollJobStatus --> PollJobStatus: [Job Not Ready]
    PollJobStatus --> ProcessResults: [Job Ready]
    ProcessResults --> SendEvents
    SendEvents --> StoreLogs
    StoreLogs --> Cleanup
    Cleanup --> [*]
```

### Lambda Function Interactions

```mermaid
sequenceDiagram
    participant SF as Step Functions
    participant L1 as Lambda: InitiateSplunkJob
    participant Splunk
    participant L2 as Lambda: PollJobStatus
    participant L3 as Lambda: ProcessResults
    participant EB as AWS EventBridge
    participant S3 as AWS S3
    SF->>L1: Start InitiateSplunkJob
    L1->>Splunk: Initiate Job
    L1-->>SF: Return Job ID
    SF->>L2: Start PollJobStatus with Job ID
    L2->>Splunk: Check Job Status
    alt Job Not Ready
        L2-->>SF: Not Ready
        SF->>L2: Wait and Retry
    else Job Ready
        L2-->>SF: Job Ready
    end
    SF->>L3: Start ProcessResults with Job ID
    L3->>Splunk: Fetch Results in Chunks
    L3->>EB: Send Events
    L3->>S3: Store Raw Logs
    L3->>Splunk: Cancel Job
```

### Class Inheritance Diagram

```mermaid
classDiagram
    class GenericSplunkCollector{
        +run_query(query, earliest_time)
        +get_job(sid)
        +connect()
    }
    class PingCollector{
        +process_ping_logs()
    }
    class MyIDCollector{
        +process_myid_logs()
    }
    GenericSplunkCollector <|-- PingCollector
    GenericSplunkCollector <|-- MyIDCollector
```

---

Feel free to explore the diagrams and code snippets for a better understanding of the system's workflow and architecture.
