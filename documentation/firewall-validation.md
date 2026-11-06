# Firewall Configuration and Validation

## 1. Objective

The objective of firewall configuration is to control communication between Docker application services and allow only the required traffic for the main application workflows.

The rules are implemented in the `DOCKER-USER` chain using `iptables`.

## 2. Authorized Network Flows

The following rules explicitly authorize the required service-to-service communications:

| Source service | Source IP | Destination service | Destination IP | Protocol / Port |
|---|---|---|---|---|
| Vote | 172.20.1.20 | Redis | 172.20.2.10 | TCP/6379 |
| Worker | 172.20.2.20 | PostgreSQL | 172.20.3.10 | TCP/5432 |
| Result | 172.20.1.30 | PostgreSQL | 172.20.3.10 | TCP/5432 |

An additional `ESTABLISHED,RELATED` rule allows packets belonging to established or related connections.

## 3. Firewall Rule Persistence

A dedicated `firewall-init` service is included in the Docker Compose configuration.

Its responsibilities are:

- Access the host network namespace.
- Check whether each required firewall rule already exists.
- Insert missing rules into `DOCKER-USER`.
- Avoid creating duplicate rules when the initialization script runs again.

The service uses `justincormack/nsenter1` with elevated privileges to access the host's iptables configuration in the Docker Desktop WSL2 environment.

**Security consideration:** This service requires privileged access to the host environment. Such permissions should be restricted and carefully reviewed in production environments.

## 4. Validation Tests

### 4.1 Authorized communication: Vote to Redis

A TCP connection test was executed from the Vote container to the Redis service:

```bash
docker exec example-voting-app-vote-1 python -c "import socket; s=socket.create_connection(('172.20.2.10',6379),5); print('CONNEXION AUTORISEE : Vote -> Redis'); s.close()"
```

**Observed result:** The TCP connection succeeded.

This confirms that the Vote service can reach Redis on TCP port 6379.

### 4.2 Unauthorized communication: Test container to Redis

A temporary test container was attached only to the frontend network, using IP address `172.20.1.99`.

A TCP connection attempt to Redis was performed:

```bash
docker exec test-unauthorized sh -c 'nc -z -w 5 172.20.2.10 6379; echo EXIT_CODE=$?'
```

**Observed result:**

```text
EXIT_CODE=1
```

The TCP connection attempt failed.

This confirms that the test container could not establish a connection to Redis during this test. It does not, by itself, identify which firewall rule or Docker network mechanism caused the failure.

### 4.3 Worker and Result database access

The corresponding authorization rules were found in the `DOCKER-USER` chain.

Packet counters were also observed for both rules:

- Worker to PostgreSQL: matching packets observed.
- Result to PostgreSQL: matching packets observed.

These counters confirm that traffic matched the corresponding rules. They do not independently prove successful database transactions.

## 5. Validation Summary

| Test | Expected behavior | Observed result |
|---|---|---|
| Vote to Redis | Connection allowed | Successful TCP connection |
| Test container to Redis | Connection unavailable | TCP connection failed |
| Worker to PostgreSQL | Traffic explicitly allowed | Rule matches observed |
| Result to PostgreSQL | Traffic explicitly allowed | Rule matches observed |

## 6. Limitations

The current configuration defines explicit authorization rules for required application flows.

However, these rules alone do not implement a global default-deny policy for all other forwarded traffic. Docker Desktop also maintains its own firewall and network rules.

Further validation and a complete review of the effective forwarding policy would be necessary before claiming comprehensive network access control.

## 7. Conclusion

The firewall configuration introduces explicit rules for the main communication flows between application services.

The tests demonstrate successful Vote-to-Redis connectivity and a failed connection attempt from a temporary frontend test container to Redis.

Together with Docker network segmentation, these measures improve the control and visibility of inter-container communication.