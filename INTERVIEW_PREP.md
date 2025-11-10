# Technical Interview Preparation - Quick Guide

## 1. OS (3%)

### Folder Structure
**Q: What are /etc, /var, /bin directories for?**
- `/etc` - Configuration files
- `/var` - Variable data (logs, temp files)
- `/bin` - Essential binaries (commands)

### Processes
**Q: What's the difference between process and thread?**
- Process: Independent program with own memory
- Thread: Lightweight process sharing memory

### Commands
```bash
ps aux          # List all processes
kill -9 1234    # Force kill process
htop            # Interactive process viewer
df -h           # Disk usage
tcpdump         # Network packet analyzer
```

---

## 2. Networking (10%)

### OSI Model (20%)
**Q: Name the 7 layers**
1. Physical → 2. Data Link → 3. Network → 4. Transport → 5. Session → 6. Presentation → 7. Application

### TCP vs UDP (25%)
**Q: When to use TCP vs UDP?**
- **TCP**: Reliable, ordered (HTTP, email, file transfer)
- **UDP**: Fast, no guarantee (streaming, gaming, DNS)

### HTTP/HTTPS (40%)
**Q: Common status codes?**
- 200 OK, 201 Created
- 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
- 500 Internal Server Error

**Q: What's HTTPS?**
- HTTP + TLS/SSL encryption

---

## 3. Git (5%)

### Basics (35%)
```bash
git init                    # Initialize repo
git clone <url>            # Clone repo
git add .                  # Stage changes
git commit -m "message"    # Commit
git push origin main       # Push to remote
git pull                   # Fetch + merge
git branch feature-x       # Create branch
git checkout feature-x     # Switch branch
git merge feature-x        # Merge branch
```

### Workflows (30%)
**Q: Explain Feature Branch Workflow**
1. Create branch from main
2. Make changes
3. Commit to feature branch
4. Create Pull Request
5. Review and merge

### Advanced (35%)
```bash
git rebase main            # Rebase onto main
git stash                  # Save changes temporarily
git stash pop              # Restore stashed changes
```

---

## 4. Programming (10%)

### Data Types
```go
// Basic
int, float, string, bool

// Collections
array := [3]int{1,2,3}
slice := []int{1,2,3}
map := map[string]int{"a": 1}

// Struct
type Person struct {
    Name string
    Age  int
}
```

### OOP
**Q: Explain key OOP concepts**
- **Class**: Blueprint for objects
- **Object**: Instance of class
- **Inheritance**: Child class from parent
- **Method**: Function in class
- **Overload**: Same method name, different parameters

### Concurrency
```go
go myFunction()     // Goroutine
ch := make(chan int)  // Channel
```

### Big O Notation
- O(1) - Constant
- O(log n) - Logarithmic
- O(n) - Linear
- O(n²) - Quadratic

---

## 5. Database - SQL (7%)

### Normalization (6%)
**Q: What is 1NF, 2NF, 3NF?**
- **1NF**: Atomic values, no repeating groups
- **2NF**: 1NF + no partial dependencies
- **3NF**: 2NF + no transitive dependencies

### DDL
```sql
CREATE TABLE users (id INT, name VARCHAR(50));
ALTER TABLE users ADD email VARCHAR(100);
DROP TABLE users;
```

### DML/DQL
```sql
INSERT INTO users VALUES (1, 'John');
UPDATE users SET name = 'Jane' WHERE id = 1;
DELETE FROM users WHERE id = 1;

SELECT * FROM users 
WHERE age > 18
GROUP BY country
ORDER BY name
LIMIT 10 OFFSET 20;  -- Pagination
```

### Joins
```sql
-- INNER: Only matching rows
SELECT * FROM orders INNER JOIN users ON orders.user_id = users.id;

-- LEFT: All from left + matching right
SELECT * FROM users LEFT JOIN orders ON users.id = orders.user_id;

-- RIGHT: All from right + matching left
SELECT * FROM orders RIGHT JOIN users ON orders.user_id = users.id;
```

### Indexes
**Q: Why use indexes?**
- Speed up queries
- Types: B-tree (default), Hash (equality), GiST, GIN (full-text)

### Replication
**Q: What is database replication?**
- Master-Slave: Write to master, read from slaves
- Purpose: High availability, load balancing

### Sharding
**Q: What is sharding?**
- Split data across multiple databases
- Example: Users 1-1000 → DB1, 1001-2000 → DB2

---

## 6. NoSQL (5%)

### CAP Theorem
**Q: What is CAP?**
- **C**onsistency: All nodes see same data
- **A**vailability: Every request gets response
- **P**artition tolerance: System works despite network failures
- Can only have 2 of 3

### Document DB
```json
{
  "_id": "123",
  "name": "John",
  "orders": [
    {"product": "Laptop", "price": 1000}
  ]
}
```

### Denormalization
**Q: Why denormalize in NoSQL?**
- Faster reads (no joins)
- Embed related data

---

## 7. Caching (10%)

### Types
**Q: Caching strategies?**
- **In-Memory**: Redis, Memcached (fast access)
- **Distributed**: Shared across servers
- **CDN**: Cache static content near users

### Eviction Policies
- **LRU**: Least Recently Used
- **LFU**: Least Frequently Used
- **FIFO**: First In First Out
- **TTL**: Time To Live

---

## 8. Docker (7%)

### Concepts (15%)
**Q: What is Docker?**
- Containerization platform
- Packages app + dependencies
- Runs consistently anywhere

### Dockerfile (15%)
```dockerfile
FROM node:16
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Docker Compose (15%)
```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
```

**Commands:**
```bash
docker build -t myapp .
docker run -p 3000:3000 myapp
docker-compose up -d
```

---

## 9. API - REST (15%)

### RESTful Concepts
**Q: What is REST?**
- Architectural style for APIs
- Stateless
- Resource-based URLs

### HTTP Methods
```
GET    /users      → List users
GET    /users/1    → Get user 1
POST   /users      → Create user
PUT    /users/1    → Update user 1 (full)
PATCH  /users/1    → Update user 1 (partial)
DELETE /users/1    → Delete user 1
```

### Status Codes
- 2xx: Success (200, 201, 204)
- 4xx: Client error (400, 401, 403, 404)
- 5xx: Server error (500, 503)

### CORS
**Q: What is CORS?**
- Cross-Origin Resource Sharing
- Security mechanism
- Controls which domains can access API

---

## 10. GraphQL (10%)

### Query
```graphql
query {
  user(id: "1") {
    name
    email
    posts {
      title
    }
  }
}
```

### Mutation
```graphql
mutation {
  createUser(name: "John", email: "john@example.com") {
    id
    name
  }
}
```

**Q: GraphQL vs REST?**
- GraphQL: Request exactly what you need, single endpoint
- REST: Fixed responses, multiple endpoints

---

## 11. gRPC (10%)

### Protocol Buffers
```protobuf
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

service UserService {
  rpc GetUser(UserRequest) returns (User);
  rpc CreateUser(User) returns (User);
}
```

**Q: gRPC vs REST?**
- gRPC: Binary, faster, HTTP/2, strongly typed
- REST: Text (JSON), HTTP/1.1, flexible

---

## 12. API Gateway (10%)

### Purpose
**Q: What does API Gateway do?**
- Single entry point
- Routing requests
- Load balancing
- Authentication
- Rate limiting
- Request/response transformation

### Example Flow
```
Client → API Gateway → Auth Service
                    → User Service
                    → Order Service
```

---

## 13. Serverless (25%)

### Concepts
**Q: What is serverless?**
- No server management
- Event-driven
- Pay per execution
- Auto-scaling

### OpenFaaS Example
```yaml
functions:
  hello:
    lang: node
    handler: ./hello
    image: hello:latest
```

---

## Quick Interview Tips

### Common Questions

**1. "Tell me about a technical challenge you faced"**
- Describe problem
- Your approach
- Solution implemented
- Results/learnings

**2. "How do you optimize database queries?"**
- Add indexes
- Optimize JOINs
- Use EXPLAIN ANALYZE
- Cache results
- Denormalize if needed

**3. "Explain your development workflow"**
- Git feature branch
- Write code + tests
- Local testing
- Pull request
- Code review
- CI/CD pipeline
- Deploy

**4. "How do you debug production issues?"**
- Check logs
- Monitor metrics
- Reproduce locally
- Use profiling tools
- Add logging
- Test fix
- Deploy

### System Design Questions

**"Design a URL shortener"**
```
Components:
- API: Create short URL, redirect
- Database: Store mapping (hash → long URL)
- Cache: Redis for fast lookups
- Analytics: Track clicks

Flow:
1. POST /shorten → Generate hash → Store in DB
2. GET /{hash} → Check cache → Query DB → Redirect
```

**"Design a chat application"**
```
Components:
- WebSocket server
- Message queue (Kafka)
- Database (messages)
- Cache (online users)
- CDN (media files)
```

---

## Practice Questions

### OS
1. How do you find a process using a specific port?
2. What's the difference between soft and hard links?
3. Explain process scheduling

### Networking
1. What happens when you type a URL in browser?
2. Explain the 3-way TCP handshake
3. How does HTTPS work?

### Database
1. Design a schema for e-commerce
2. Write a query to find 2nd highest salary
3. When to use NoSQL vs SQL?

### API
1. Design RESTful API for blog
2. How to handle authentication in API?
3. What's rate limiting and why use it?

---

**Good luck with your interview! 🚀**

