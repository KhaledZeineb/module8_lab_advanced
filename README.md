API Gateway Lab - README

Overview
This lab implements an API Gateway using Spring Cloud Gateway to manage and route requests to our microservices system. The gateway provides several advanced features including circuit breaking, rate limiting, request logging, and API documentation aggregation.

Prerequisites
Java 17+
Maven 3.6+
Redis server (for rate limiting)
Microservices from Module 7 running:
  - Eureka Server (port 8761)
  - Book Service (port 8081)
  - User Service (port 8082)
  - Analytics Service (port 8083)

Project Structure
api-gateway/
├── src/
│   ├── main/
│   │   ├── java/com/example/apigateway/
│   │   │   ├── config/          # Configuration classes
│   │   │   ├── controller/      # Controllers
│   │   │   ├── filter/         # Custom filters
│   │   │   └── ApiGatewayApplication.java
│   │   └── resources/
│   │       └── application.yml  # Gateway configuration
├── pom.xml                      # Maven dependencies
└── README.md

Setup Instructions
1. Clone and build the project:
   ```bash
   git clone [repository-url]
   cd api-gateway
   mvn clean install
   ```
2. Start Redis server:
   ```bash
   redis-server
   ```
3. Run the API Gateway:
   ```bash
   mvn spring-boot:run
   ```

Configuration
Key configuration in `application.yml`:

- Server port: `8080`
- Eureka client configuration
- Route definitions for:
  - Book Service (`/api/books/**`)
  - User Service (`/api/users/**`)
  - Analytics Service (`/api/analytics/**`)
- Circuit breaker settings
- Rate limiting configuration

Features
1. Service Routing
   - Routes requests to appropriate microservices based on path
   - Load balancing through Eureka
2. Circuit Breaker
   - Resilience4j implementation
   - Fallback responses when services are unavailable
   - Configurable thresholds:
     - Failure rate: 50%
     - Slow call threshold: 2 seconds
     - Open state duration: 10 seconds
3. Rate Limiting
   - Redis-based rate limiting
   - Configurable rates per service
   - Default: 10 requests/second with burst capacity of 20
4. Request Logging
   - Logs all incoming requests with:
     - Unique request ID
     - HTTP method
     - Path
     - Source IP
     - Response status
     - Processing time
5. CORS Support
   - Allows cross-origin requests from any domain
   - Configurable allowed methods and headers
6. API Documentation
   - Aggregates OpenAPI documentation from all services
   - Accessible at `/api-docs`
   - Individual service docs available at:
     - `/book-service/v3/api-docs`
     - `/user-service/v3/api-docs`
     - `/analytics-service/v3/api-docs`

Testing the Gateway
Basic Requests
```bash
# Create a user
curl -X POST http://localhost:8080/api/users -H "Content-Type: application/json" -d '{
  "username": "test_user",
  "email": "test@example.com",
  "password": "password123"
}'

# Get all books
curl http://localhost:8080/api/books

# Get analytics summary
curl http://localhost:8080/api/analytics/summary
```

Testing Circuit Breaker
1. Stop the Book Service.
2. Make a request:
   ```bash
   curl http://localhost:8080/api/books
   ```
3. Should receive fallback response:
   ```json
   {
     "status": "error",
     "message": "Book Service is currently unavailable. Please try again later."
   }
   ```

Testing Rate Limiting
Make rapid consecutive requests:
```bash
for i in {1..15}; do curl -v http://localhost:8080/api/books 2>&1 | grep "HTTP/1.1"; done
```
Should see some `429 Too Many Requests` responses after the limit is exceeded.

Checking Routes
```bash
# List all configured routes
curl http://localhost:8080/actuator/gateway/routes
```

Monitoring
- Eureka Dashboard: http://localhost:8761
- Gateway Actuator Endpoints:
  - Routes: `/actuator/gateway/routes`
  - Health: `/actuator/health`
  - Metrics: `/actuator/metrics`

Custom Filters
- `LoggingFilter`: Logs all requests and responses
- `MetricsFilter`: Collects timing and count metrics
- `AddRequestHeader`: Adds custom headers to requests

Scaling Services
To test load balancing:

1. Start additional instances of a service on different ports:
   ```bash
   java -jar book-service/target/book-service-0.0.1-SNAPSHOT.jar --server.port=8091
   ```
2. Make multiple requests and observe the `X-Instance-Id` header to see requests being distributed.

Troubleshooting
- If services aren't registering, check Eureka server is running.
- For rate limiting issues, ensure Redis server is running.
- Check logs for detailed error information.
- Verify all services are using compatible Spring Cloud versions.

Next Steps
- Containerize the gateway and services using Docker.
- Implement JWT authentication at the gateway level.
- Add distributed tracing with Sleuth/Zipkin.
- Configure more granular rate limiting per API endpoint.
