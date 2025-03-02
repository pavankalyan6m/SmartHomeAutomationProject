# Smart Home Automation System - End-to-End Documentation

## 1. Overview

This project is a **Smart Home Automation System** built using **Spring Boot (latest version), Spring Cloud, MySQL, MongoDB, Feign Client for synchronous communication, and Docker for deployment**. It consists of four microservices:

- **User Service (MySQL)**: Handles authentication, user management, and role-based access.
- **Device Service (MongoDB)**: Manages smart home devices and their status.
- **Control Service (MySQL)**: Processes commands to control devices.
- **Analytics Service (MongoDB)**: Logs device usage and generates analytics reports.

## 2. Technology Stack

- **Spring Boot** (Microservices architecture)
- **Spring Cloud (Eureka, Config Server, API Gateway)**
- **MySQL (for User and Control Service)**
- **MongoDB (for Device and Analytics Service)**
- **Feign Client/WebClient** (Synchronous communication)
- **Spring Security & JWT** (Authentication & Authorization)
- **Docker** (Containerization and Deployment)
- **GitHub** (Version Control & CI/CD setup)

---

## 3. Microservices Architecture

```
[ User Service ] ---> [ Control Service ] ---> [ Device Service ]
                                  |                |
                                  V                V
                           [ Analytics Service ]
```

- **User Service** validates user requests.
- **Control Service** processes ON/OFF commands.
- **Device Service** updates device states.
- **Analytics Service** logs device usage.

---

## 4. Microservices Patterns

### Common Microservices Patterns

1. **API Gateway Pattern**: Manages and routes requests from clients to microservices.
2. **Service Registry & Discovery Pattern**: Uses Eureka for dynamic service registration.
3. **Synchronous Communication (Request-Response Pattern)**: Uses Feign Client/WebClient for inter-service calls.
4. **Database per Microservice Pattern**: Each service has its own database to ensure data isolation.
5. **Centralized Configuration (Config Server Pattern)**: Manages configurations centrally.
6. **Circuit Breaker Pattern**: Handles failures using Resilience4J or Hystrix.
7. **Saga Pattern**: Manages distributed transactions by using **choreography (event-driven) or orchestration (centralized controller)**.

### **Best Pattern Choice for This Project: Synchronous Request-Response with API Gateway**

#### **Reason for Choosing This Pattern:**

- **Simplicity**: Synchronous communication is easier to implement and manage.
- **Immediate Responses**: Suitable for a real-time system like home automation.
- **API Gateway**: Centralized routing, authentication, and load balancing.
- **Scalability**: Works well with database-per-service architecture.

### **Why Not Using Saga Pattern?**

While **Saga Pattern** is useful for handling distributed transactions, it is more suitable for systems requiring strong consistency across multiple microservices (e.g., financial transactions). Since **our project focuses on real-time control of devices** rather than long-running transactions, a **request-response model is more efficient and simpler** to implement.

---

## 5. Project Structure

```
smart-home-automation/
│── user-service/
│── control-service/
│── device-service/
│── analytics-service/
│── config-server/
│── eureka-server/
│── api-gateway/
│── docker-compose.yml
```

Each microservice follows a layered architecture:

```
/src/main/java/com/example/service
    ├── controller/
    ├── service/
    ├── repository/
    ├── model/
    ├── dto/
```

---

## 6. Setting Up the Environment

### Prerequisites

- Java 17+
- MySQL & MongoDB
- Docker & Docker Compose
- Postman (for API testing)
- GitHub account (for VCS & CI/CD)

---

## 7. Microservice Implementation

### 7.1 **User Service (MySQL)**

Handles **user authentication and access control**.

#### **Dependencies** (pom.xml)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### **Entity: User.java**

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String role;
    // Getters and Setters
}
```

#### **Authentication (Spring Security & JWT)**

- **Spring Security config**
- **JWT token generation & validation**

---

### 7.2 **Device Service (MongoDB)**

Manages smart devices (lights, fans, etc.).

#### **Entity: Device.java**

```java
@Document(collection = "devices")
public class Device {
    @Id
    private String id;
    private String name;
    private String status; // ON/OFF
}
```

---

### 7.3 **Control Service (MySQL)**

Processes device control requests.

#### **Feign Client for Synchronous Calls**

```java
@FeignClient(name = "device-service")
public interface DeviceClient {
    @PostMapping("/devices/{id}/update")
    ResponseEntity<String> updateDeviceStatus(@PathVariable String id, @RequestBody Device device);
}
```

---

### 7.4 **Analytics Service (MongoDB)**

Logs device usage for reports.

#### **Entity: DeviceLog.java**

```java
@Document(collection = "device_logs")
public class DeviceLog {
    @Id
    private String id;
    private String deviceId;
    private String action;
    private LocalDateTime timestamp;
}
```

---

## 8. Service Discovery & API Gateway

### **Eureka Server Setup**

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### **API Gateway Setup**

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/users/**
```

---

## 12. Conclusion

This document provides a **complete step-by-step guide** to building a **Smart Home Automation System** with **Spring Boot microservices**, **MySQL & MongoDB**, **Feign Client communication**, **Spring Security & JWT**, **Docker**, and **GitHub CI/CD**. 🚀

Let me know if you need any modifications or additions! 😊

