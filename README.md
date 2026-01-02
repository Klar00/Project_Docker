# Project_Docker


# 3-Container Docker Application (Oracle + Python)

This repository contains a **final project** presenting a three-container application built with **Docker Compose**, **Oracle Database XE**, and two **Python microservices** (Writer and Analyzer). The project demonstrates a microservice-based architecture, separation of responsibilities, and implementation of business logic directly in the database layer using **PL/SQL**.

---

## Project Goal

The main goals of the project were:

* to design and run a **three-container Docker application**,
* to separate responsibilities between application and database layers,
* to implement business logic using **PL/SQL**,
* to enable communication between containers using **Docker Compose**.

---

## System Architecture

The application consists of three independent Docker containers:

### 1. Oracle XE (`oracle-xe`)

* Oracle Database 21c Express Edition
* Stores application data
* Implements business logic using PL/SQL packages

### 2. Writer Service (`writer`)

* Python-based service
* Generates sample data (orders)
* Creates processing jobs
* Calls PL/SQL procedures responsible for data processing
* Acts as an orchestrator (no business logic inside the service)

### 3. Analyzer Service (`analyzer`)

* Python-based analytical service
* Reads data from the Oracle database
* Calculates business metrics
* Stores metrics in the database and exports results to CSV/PNG files

---

## Communication

* Containers communicate via the Docker Compose network
* Writer and Analyzer connect to Oracle via TCP (port 1521)
* Data is shared **only through the database**

---

## Data Model

The Oracle database includes the following key tables:

* `APP_ORDER` – orders
* `APP_JOB` – job queue for processing
* `APP_INVOICE` – generated invoices
* `APP_METRICS` – calculated business metrics
* `APP_REJECTED_ROWS` – invalid or rejected records

The model is based on a **job queue mechanism**, where each order generates a separate job for processing.

---

## Business Logic (PL/SQL)

Core system logic is implemented in Oracle using PL/SQL packages:

### `PKG_BILLING`

Responsible for:

* fetching jobs with status `QUEUED`,
* atomically claiming jobs (`IN_PROGRESS`),
* processing orders,
* generating invoices,
* handling exceptions.

Example function:

* `CLAIM_JOB(p_worker)` – atomically claims a single job for processing.

### `PKG_AUDIT`

Audit package responsible for:

* logging system events,
* storing diagnostic information.

All packages are maintained in `VALID` state.

---

## Writer Service Details

Responsibilities:

* generating sample orders,
* creating jobs in `APP_JOB`,
* cyclic execution of PL/SQL procedures.

Results:

* all jobs transition from `QUEUED` to `DONE`,
* each job generates a corresponding invoice in `APP_INVOICE`.

---

## Analyzer Service Details

Responsibilities:

* reading data from Oracle,
* calculating metrics such as:

  * total number of orders,
  * total revenue,
  * most frequently sold product,
  * number of invalid records,
* saving metrics to `APP_METRICS`,
* exporting results to:

  * `metrics.csv`,
  * `revenue.png`.

The Analyzer runs cyclically and does **not modify business data**.

---

## Verification & Results

Correct system behavior is confirmed by:

* jobs with status `DONE`,
* generated invoices in `APP_INVOICE`,
* growing number of records in `APP_METRICS`,
* correctly generated CSV and PNG artifacts,
* no invalid PL/SQL packages.

The system runs stably and according to project assumptions.

---

## Summary

The project fulfills all requirements:

* three-container architecture,
* Oracle + PL/SQL used for business logic,
* clear separation of responsibilities between services,
* automated data processing and analysis,
* fully automated Docker-based environment.

This project demonstrates practical usage of **Docker**, **Python**, and **Oracle** in a microservice-oriented architecture.
