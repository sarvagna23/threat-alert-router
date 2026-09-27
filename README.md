# Threat Alert Router

A Spring Boot microservice that receives mock security alert payloads (simulating a SIEM feed), classifies them by severity using rule-based logic, and routes each alert to its own Kafka topic for downstream handlers. All incoming alerts are persisted to PostgreSQL.

## Features

- REST endpoint (`POST /alerts`) for ingesting mock SIEM alert payloads
- Rule-based severity classifier (`critical` / `high` / `medium` / `low`)
- Kafka producer that routes each alert to a severity-specific topic
- PostgreSQL persistence of all incoming alerts
- JUnit unit tests for classifier and routing logic
- Dockerized, with CI/CD via GitHub Actions

## Stack

- Java, Spring Boot
- Apache Kafka
- PostgreSQL
- Docker
- GitHub Actions

## API

### `POST /alerts`

Accepts a mock security alert payload and returns the classified severity plus the Kafka topic it was routed to.

```json
{
  "sourceIp": "10.0.4.22",
  "eventType": "brute_force_login",
  "affectedHost": "prod-web-03",
  "failedAttempts": 47,
  "timestamp": "2026-09-27T15:10:00Z"
}
```

Example response:

```json
{
  "id": "a1b2c3d4",
  "severity": "critical",
  "routedTopic": "alerts.critical",
  "status": "accepted"
}
```

## Severity Classification

Rule-based classification runs against payload fields (event type, failed attempt counts, affected host criticality, etc.) to assign one of four severity tiers, each mapped to its own Kafka topic:

| Severity | Kafka Topic       |
|----------|-------------------|
| Critical | `alerts.critical` |
| High     | `alerts.high`     |
| Medium   | `alerts.medium`   |
| Low      | `alerts.low`      |

## Project Structure

```
threat-alert-router/
├── src/
│   ├── main/
│   │   ├── java/com/sai/threatalertrouter/
│   │   │   ├── controller/      # REST controllers (POST /alerts)
│   │   │   ├── service/         # Alert ingestion / orchestration
│   │   │   ├── classifier/      # Severity classification rules
│   │   │   ├── kafka/           # Kafka producer + topic routing
│   │   │   ├── model/           # Entities and DTOs
│   │   │   ├── repository/      # PostgreSQL persistence (JPA)
│   │   │   └── config/          # Kafka / DB configuration
│   │   └── resources/           # application.yml, schema, etc.
│   └── test/
│       └── java/com/sai/threatalertrouter/   # JUnit tests
├── .github/workflows/           # CI/CD pipeline
├── Dockerfile
├── docker-compose.yml
├── pom.xml
└── README.md
```

## Build & Run

```bash
mvn clean install
mvn spring-boot:run
```

## Run with Docker

```bash
docker compose up --build
```

## Tests

```bash
mvn test
```

## License

MIT
