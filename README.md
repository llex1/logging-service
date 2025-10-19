# LOGGING SERVICE
***

*Implementation depends on the situation, business requirements, budget, development team expertise, and available time for implementation. Since we don’t have any specific details, let’s assume the following case without limitations:*

## Case Description  
An error logging system is being developed for a manufacturing and warehouse facility to capture equipment failures, sensor data (e.g. temperature, vibration, load), and operational conflicts during item movement, dispatch, or intake—enabling fast incident response, root cause analysis, and process optimization.

## Backend Technologies:
 - NestJS (TypeScript) – Chosen for its modular architecture, built-in dependency injection, guards, and interceptors. Compared to Express, it offers better structure for scalable projects and centralized error handling.
 - PostgreSQL – Reliable relational database with JSONB support, ideal for storing both structured metadata (users, devices, log types) and semi-structured logs. Preferred over MongoDB due to schema clarity and relational integrity.
 - Redis – Used for caching recent logs, rate limiting, TTL for temporary sensor data, and powering BullMQ queues. Chosen for its speed and seamless integration with NestJS.
 - BullMQ – Selected over RabbitMQ for simpler setup and native Redis compatibility.
 - Logstash – Preferred over Fluentd due to better performance with large log volumes and stable Elasticsearch integration.
 - Elasticsearch – Chosen over ClickHouse because it excels at full-text search and complex filtering.
 - REST API – Used for log ingestion from SDKs and devices. 
 - GraphQL – Used for dashboard queries (filtering, pagination, aggregation).
 - Socket.IO – Enables real-time updates on the dashboard (e.g. live error feed). Preferred over raw WebSocket API due to built-in reconnect, fallback, and easy NestJS integration.
 - Prometheus – Collects metrics for system health and performance. Chosen over StatsD for its ecosystem and Grafana compatibility.
 - Grafana – Selected for its flexible dashboards and native Prometheus support.
 - Docker – Containerization for consistent deployment across environments. Industry standard for isolating services.
 - GitHub Actions – CI/CD pipeline for automated testing and deployment. Chosen under the assumption that GitHub is the selected Git service.
 - Terraform – Infrastructure as code for reproducible cloud setup and scaling. Preferred over Pulumi due to broader provider support and mature AWS/GCP modules.

## Frontend Technologies:
 - React
 - Vite
 - TypeScript
 - Tailwind CSS
 - React Router
 - Apollo Client
 - TanStack Table
 - Chart.js
 - Socket.IO
 - NextAuth or Auth0
 - Zustand or Redux Toolkit
 - ESLint
 - Prettier
 - Vitest or Jest

## The real-time email alert mechanism for critical errors will be implemented as follows:
1.  The backend exposes a REST API endpoint for receiving logs. After validating the payload, the system checks the severity level. If the log is marked as ERROR or FATAL, it is pushed into a Redis-backed queue using BullMQ.
2.  Each log is enqueued as a separate job with metadata such as timestamp, error type, source, device ID, or user ID. To prevent duplicate alerts, a deduplication mechanism is applied based on a hash of the log message.
3.  A dedicated BullMQ worker processes the queue. It aggregates logs by application or user and checks if the number of critical logs exceeds a configured threshold (e.g. 5 errors within 1 minute). If the threshold is met, an email alert is triggered.
4.  Email delivery is handled via SMTP using Nodemailer or through a third-party service such as SendGrid, depending on the environment. The email includes timestamp, error type, description, and a link to view the log in the dashboard.
5.  A throttling mechanism ensures that no more than one email is sent per user within a 5-minute window. Mute windows can be configured to suppress alerts during specific time intervals.
6.  Alert thresholds, delivery channels (email, Slack, webhook), and mute windows are configurable via the web dashboard. Configuration data is stored in PostgreSQL.
7.  System metrics such as queue size, alert frequency, and delivery status are monitored via Prometheus and visualized in Grafana. If email delivery fails, a WARN-level log is generated for internal tracking.
