# Security Measures Implemented

## 1. Overview

This project focuses on improving the security of an existing Docker-based voting application.

Several security measures were introduced to strengthen network architecture, reduce unnecessary service exposure, and improve control over application traffic.

The main measures are:

- Nginx as a single public entry point.
- HTTP request rate limiting.
- Restriction of allowed HTTP methods.
- Docker network segmentation.
- Firewall rules for required inter-container communication.
- Reduction of unnecessary container privileges and exposed ports.

## 2. Nginx as a Single Entry Point

Nginx is used as a reverse proxy between external clients and the application services.

Instead of exposing the Vote and Result services directly, external HTTP requests are routed through Nginx.

The proxy forwards requests to the appropriate internal service:

| Public endpoint | Internal destination |
|---|---|
| `/vote` | Vote service |
| `/result` | Result service |

This architecture centralizes access to the application and reduces direct exposure of backend services.

## 3. HTTP Rate Limiting

Nginx rate limiting is configured to control the number of incoming requests.

The implemented configuration uses:

- Request rate: 10 requests per second.
- Burst capacity: 20 requests.

This measure helps reduce excessive request traffic and provides a basic protection mechanism against abusive request patterns.

Rate limiting is not a complete solution against distributed denial-of-service attacks.

## 4. HTTP Method Restrictions

The Nginx configuration restricts accepted HTTP methods to:

- `GET`
- `POST`

This limits the HTTP methods available through the public reverse proxy and helps reduce unnecessary application operations.

## 5. Docker Network Segmentation

Application services are separated into three dedicated Docker networks:

- `fr-net`: frontend services.
- `bk-net`: backend services.
- `db-net`: database service.

The database network is configured with `internal: true`.

This architecture reduces unnecessary network exposure and separates application components according to their roles.

See [Network Segmentation](network-segmentation.md) for the detailed architecture.

## 6. Inter-Container Traffic Filtering

Custom iptables rules are installed in the `DOCKER-USER` chain.

They explicitly authorize the required communication flows:

- Vote to Redis.
- Worker to PostgreSQL.
- Result to PostgreSQL.

The configuration also handles established and related connections.

The current implementation does not define a global default-deny policy for every other forwarded connection.

See [Firewall Configuration and Validation](firewall-validation.md) for the rules and test results.

## 7. Container Security Improvements

The project also considers container execution privileges and unnecessary service exposure.

The security configuration aims to:

- Run application services with non-root users where supported.
- Avoid unnecessary published ports.
- Remove unnecessary access to debugging interfaces from external clients.
- Keep internal services accessible through their required application networks.

These measures reduce the potential attack surface of the application.

## 8. Conclusion

The implemented measures strengthen the security of the original Docker architecture through centralized HTTP access, network segmentation, traffic filtering, and reduced service exposure.

The project demonstrates how an existing multi-container application can be progressively hardened through infrastructure and network security configurations.