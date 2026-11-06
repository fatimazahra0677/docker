# Network Segmentation

## 1. Objective

The objective of network segmentation is to isolate the application's services into separate network zones in order to reduce unnecessary communication and limit lateral movement between containers.

## 2. Network Architecture

The application is organized into three Docker bridge networks.

| Network | Subnet | Connected services |
|---|---|---|
| fr-net | 172.20.1.0/24 | Nginx, Vote, Result |
| bk-net | 172.20.2.0/24 | Redis, Worker |
| db-net | 172.20.3.0/24 | PostgreSQL |

## 3. Network Configuration

### Frontend network – fr-net
- Nginx: 172.20.1.10
- Vote: 172.20.1.20
- Result: 172.20.1.30

### Backend network – bk-net
- Redis: 172.20.2.10
- Worker: 172.20.2.20

### Database network – db-net
- PostgreSQL: 172.20.3.10
- Internal network: enabled

## 4. Security Benefits

- Separation of application components into dedicated network zones.
- Reduction of unnecessary network exposure.
- Isolation of the database network from external connectivity by default.
- Improved control over communication between services.