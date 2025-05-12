
# Lab - Implementing API Gateway with Spring Cloud Gateway

## Objective
Integrate Spring Cloud Gateway into our microservices system and implement circuit breaker patterns to improve resilience.

## Prerequisites
- Completed Module 7 lab with the following microservices:
  - Book Service (port 8081)
  - User Service (port 8082)
  - Analytics Service (port 8083)
  - Eureka Server (port 8761)
- Java JDK 11+
- Maven 3.6+
- Redis server (for rate limiting)
- Postman or curl for testing

## Project Structure
```
api-gateway/
├── src/
│   ├── main/
│   │   ├── java/com/example/apigateway/
│   │   │   ├── config/
│   │   │   │   ├── CircuitBreakerConfiguration.java
│   │   │   │   └── CorsConfig.java
│   │   │   ├── controller/
│   │   │   │   ├── ApiDocsController.java
│   │   │   │   └── FallbackController.java
│   │   │   ├── filter/
│   │   │   │   ├── AddRequestHeaderGatewayFilterFactory.java
│   │   │   │   ├── LoggingFilter.java
│   │   │   │   └── MetricsFilter.java
│   │   │   └── ApiGatewayApplication.java
│   │   └── resources/
│   │       └── application.yml
├── pom.xml
└── README.txt
```

## Setup Instructions

1. **Clone the project** (if not already created):
   ```bash
   git clone [repository-url]
   cd api-gateway
   ```

2. **Install dependencies**:
   ```bash
   mvn clean install
   ```

3. **Start Redis server** (required for rate limiting):
   ```bash
   redis-server
   ```

4. **Start all services in order**:
   - Eureka Server
   - Config Server (if applicable)
   - Book Service
   - User Service
   - Analytics Service
   - API Gateway

5. **Verify services are registered**:
   - Access Eureka dashboard: http://localhost:8761
   - Check registered services

## Configuration Highlights

### Key application.yml settings:
```yaml
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: book-service
          uri: lb://book-service
          predicates:
            - Path=/api/books/**
          filters:
            - CircuitBreaker
            - RequestRateLimiter
            - AddRequestHeader=X-Source,api-gateway

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

### Circuit Breaker Configuration:
- Sliding window size: 10 requests
- Failure rate threshold: 50%
- Wait duration in open state: 10 seconds
- Timeout duration: 3 seconds

## Testing the API Gateway

### Basic Tests:
```bash
# Get all users
curl http://localhost:8080/api/users

# Create a book
curl -X POST http://localhost:8080/api/books -H "Content-Type: application/json" -d '{"title":"Spring Cloud","author":"John Doe"}'

# Get analytics
curl http://localhost:8080/api/analytics/summary
```

### Circuit Breaker Test:
1. Stop the book service
2. Make a request:
   ```bash
   curl http://localhost:8080/api/books
   ```
3. Verify fallback response

### Rate Limiting Test:
```bash
# Make rapid requests to test rate limiting
for i in {1..30}; do curl -I http://localhost:8080/api/books 2>/dev/null | grep HTTP; sleep 0.1; done
```

### API Documentation:
Access aggregated API docs:
```
http://localhost:8080/api-docs
```

## Monitoring

### Actuator Endpoints:
```
http://localhost:8080/actuator/gateway/routes
http://localhost:8080/actuator/health
http://localhost:8080/actuator/metrics
```

## Custom Filters
- **LoggingFilter**: Logs all incoming requests with timing
- **MetricsFilter**: Collects metrics for Prometheus
- **AddRequestHeader**: Adds custom header to all requests

## Troubleshooting

1. **Services not registering with Eureka**:
   - Verify Eureka server is running
   - Check service configuration for correct Eureka URL
   - Ensure services have Eureka client dependency

2. **Gateway routes not working**:
   - Check /actuator/gateway/routes endpoint
   - Verify service names in route definitions match Eureka registration

3. **Circuit breaker not triggering**:
   - Verify service is actually down
   - Check circuit breaker configuration
   - Monitor logs for resilience4j events

## Next Steps
- Containerize the application with Docker
- Implement distributed tracing
- Add JWT authentication

