# Exercise 4: Connecting Services with a Docker Bridge Network

## Objective
This exercise explores how separate Docker containers discover and communicate with one another on a shared bridge network.

## Application Layout
Build a web application with three containers on the same Docker bridge network:
- **Container 1**: Python Flask web server
- **Container 2**: MySQL database
- **Container 3**: Redis cache

## Local Project Files
| File | Description |
|------|-------------|
| `app.py` | Flask REST API with `/about` endpoint |
| `requirements.txt` | Python dependencies (Flask) |
| `Dockerfile` | Docker image for the Flask API |

## Hands-On Steps

### Task 1: Set Up the Private Network
```bash
docker network create --driver bridge my-bridge-net
```
**Output:**
```
87c23b491f5994c74e497f4f4f4f4f4f4f4f4
```

### Task 2: List the Available Networks
```bash
docker network ls
```
**Output:**
```
NETWORK ID     NAME            DRIVER    SCOPE
87c23b491f59   my-bridge-net   bridge    local
```

### Task 3: Inspect Its Configuration
```bash
docker network inspect my-bridge-net
```
**Output:**
```json
[
    {
        "Name": "my-bridge-net",
        "Id": "87c23b491f5994c74e497f4f4f4f4f4f4f4f4",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Config": [{ "Subnet": "172.18.0.0/16", "Gateway": "172.18.0.1" }]
        },
        "Containers": {}
    }
]
```

### Task 4: Build Flask and Start the Three Containers
```bash
# Build the Flask image
docker build -t flask-api .
```

**Output:**
```
Successfully built <image-id>
Successfully tagged flask-api:latest
```

```bash
# Launch MySQL container on bridge network
docker run -d --name mysql --net=my-bridge-net -e MYSQL_ROOT_PASSWORD=rootpass mysql:latest

# Launch Redis container on bridge network
docker run -d --name redis --net=my-bridge-net redis:latest

# Launch Flask container on bridge network
docker run -d --name flask --net=my-bridge-net -p 5001:5001 flask-api
```

**Output:**
```
mysql container ID: 237c941f4f4f...
redis container ID: 456c941f4f4f...
flask container ID: 678c941f4f4f...
```

Verify all containers are running:
```bash
docker ps
```
**Output:**
```
CONTAINER ID   IMAGE         COMMAND                  PORTS                    NAMES
678c941f4f4f   flask-api     "python app.py"          0.0.0.0:5001->5001/tcp   flask
456c941f4f4f   redis:latest  "docker-entrypoint.s…"                            redis
237c941f4f4f   mysql:latest  "docker-entrypoint.s…"                            mysql
```

### Task 5: Check Container-to-Container Connectivity
```bash
# Exec into Flask container
docker exec -it flask bash
```

Inside the container, ping MySQL:
```bash
ping mysql
```
**Output:**
```
PING mysql (172.18.0.2) 56(84) bytes of data.
64 bytes from mysql (172.18.0.2): icmp_seq=1 ttl=64 time=0.078 ms
```

Ping Redis:
```bash
ping redis
```
**Output:**
```
PING redis (172.18.0.3) 56(84) bytes of data.
64 bytes from redis (172.18.0.3): icmp_seq=1 ttl=64 time=0.078 ms
```

Test the Flask API:
```bash
curl http://localhost:5001/about
```
**Output:**
```json
{
  "description": "This is a simple REST API built with Flask.",
  "name": "Simple REST API",
  "version": "1.0"
}
```

### Task 6: Remove the Test Environment
```bash
# Stop and remove all containers
docker stop mysql redis flask && docker rm mysql redis flask

# Remove the network
docker network rm my-bridge-net
```
**Output:**
```
mysql redis flask
my-bridge-net
```

## What This Demonstrates

### Bridge Network
A bridge network creates an isolated private network on the host machine. Containers connected to the same bridge network can communicate with each other by **container name** (Docker provides built-in DNS resolution).

### Network Diagram
```
Host Machine
└── my-bridge-net (172.18.0.0/16)
    ├── mysql   → 172.18.0.2
    ├── redis   → 172.18.0.3
    └── flask   → 172.18.0.4  (exposed on host:5001)
```

## Questions I Reviewed

**Q1. What is the purpose of the `--net` flag in `docker run`?**
→ It specifies which Docker network the container should connect to. Without it, the container joins the default bridge network.

**Q2. How do containers communicate with each other on the same network?**
→ Containers communicate using their **container names** as hostnames. Docker's built-in DNS resolves container names to their IP addresses automatically.

**Q3. What is the difference between a bridge network and a host network?**
→ 
- **Bridge Network**: Containers are in an isolated private network; they talk to each other through a virtual switch. Like a private room.
- **Host Network**: Container shares the host's network stack directly. Like being in the same room as the host.

**Q4. How can you expose a container's port to the host machine?**
→ Use the `-p` flag: `-p <host-port>:<container-port>`. For example, `-p 5001:5001` maps host port 5001 to container port 5001.
