---
title: "Package by Layer vs Package by Feature (and Where Clean Architecture Fits In)"
date: 2025-10-25
author: "Jim Shingler"
description: "A practical guide to structuring your Spring Boot applications — exploring package-by-layer, package-by-feature, and how Clean Architecture and Hexagonal Architecture fit together."
tags: ["Spring Boot", "Architecture", "Clean Architecture", "Hexagonal Architecture", "Software Design", "java"]
---

# Package by Layer vs Package by Feature  
## And How Clean Architecture Fits In

In Java Spring Boot development, one of the most impactful architectural choices you’ll make is how to **organize your code**.  
Should you package by *layer* — like controller, service, and repository — or by *feature* — like order, customer, and inventory?

And where do **Clean Architecture** and **Hexagonal Architecture** fit into all of this?

Let’s unpack the trade-offs, principles, and evolution path — with a touch of wisdom inspired by [Dan Vega](https://danvega.dev), who’s known for his pragmatic approach to clean, maintainable Spring Boot applications.

---

## 🧱 Package by Layer — The Traditional Approach

### Example Structure

```text
com.example.orders
├── controller
│   └── OrderController.java
├── service
│   └── OrderService.java
├── repository
│   └── OrderRepository.java
└── model
    └── Order.java
```

### Philosophy

Group classes by their *technical role* — all controllers together, all services together, all repositories together.

### Pros

- Clear separation of concerns  
- Familiar and easy for newcomers  
- Works fine for small or simple apps

### Cons

- **Low cohesion:** Each feature’s code is scattered  
- **Tight coupling:** Shared services and models blur boundaries  
- **Poor scalability:** Harder to isolate or modularize features  

### When to Use

Package-by-layer works well when:
- You’re building a **small app or prototype**  
- The **team is small and co-located**  
- You don’t anticipate splitting into modules or services later  

---

## 🧩 Package by Feature — The Modern Approach

### Example Structure

```text
com.example.orders
├── order
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   └── Order.java
├── customer
│   ├── CustomerController.java
│   ├── CustomerService.java
│   ├── CustomerRepository.java
│   └── Customer.java
```

### Philosophy

Group code by **business capability** or **domain concept**, not by technical role.

### Pros

- High cohesion — each feature is self-contained  
- Easier to reason about, test, and refactor  
- Natural fit for **domain-driven design (DDD)**  
- Enables modular ownership and microservice extraction  

### Cons

- Less conventional — can confuse new developers  
- Requires discipline to avoid cross-feature coupling  
- May duplicate small utilities across features (by design)  

### When to Use

Use package-by-feature when:
- You’re building a **larger or growing system**  
- You expect **multiple teams or domains**  
- You want to evolve toward **modularity or microservices**

---

## ⚖️ Package-by-Layer vs Package-by-Feature

| Aspect | Package by Layer | Package by Feature |
|--------|------------------|--------------------|
| **Cohesion** | Low | High |
| **Coupling** | High | Low |
| **Refactorability** | Harder | Easier |
| **Testing** | Broad integration tests | Isolated feature tests |
| **Scalability** | Monolithic | Modular |
| **Team Fit** | Centralized ownership | Cross-functional ownership |

---

## 🧭 Enter Clean Architecture

**Creator:** Robert C. Martin (Uncle Bob)

**Core Rule:**  
> Business logic should not depend on frameworks, databases, or UI. Dependencies point **inward**, toward the domain.

### Conceptual View

```text
Frameworks & UI (Spring, REST)
        ↓
Interface Adapters (DTOs, Mappers)
        ↓
Application Services (Use Cases)
        ↓
Domain Entities (Business Rules)
```

### Example Layout

```text
com.example.order
├── domain
│   └── Order.java
├── application
│   └── PlaceOrderUseCase.java
├── interfaces
│   └── web
│       └── OrderController.java
└── infrastructure
    └── JpaOrderRepository.java
```

### Key Idea

Your **domain** and **application** code should know *nothing* about Spring, HTTP, or JPA.  
They’re just Java. The outer layers handle frameworks and adapters.

---

## 🔶 Hexagonal Architecture (Ports & Adapters)

**Creator:** Alistair Cockburn

**Goal:**  
> Make your application independent of its runtime environment.

### Conceptual Diagram

```text
           +------------------------+
           |        Adapters        |
           | (REST, DB, Kafka, etc) |
           +------------------------+
             ↑                ↓
         Inbound Port     Outbound Port
             ↑                ↓
           +----------------------+
           |      Domain Core     |
           |  Entities & UseCases |
           +----------------------+
```

### Example Structure

```text
com.example.order
├── application
│   ├── port
│   │   ├── inbound
│   │   │   └── PlaceOrderUseCase.java
│   │   └── outbound
│   │       └── OrderRepository.java
│   └── service
│       └── PlaceOrderService.java
├── domain
│   └── Order.java
├── adapters
│   ├── inbound
│   │   └── OrderController.java
│   └── outbound
│       └── JpaOrderRepository.java
```

### Focus

Clean architecture focuses on *dependency direction*.  
Hexagonal focuses on *communication boundaries (ports and adapters).*  
In practice, most modern teams blend both.

---

## 🧩 The Hybrid: Package-by-Feature + Clean/Hexagonal Inside

Here’s the *best of both worlds*:

```text
com.example.shop
├── order
│   ├── domain
│   ├── application
│   ├── adapters
│   │   ├── inbound
│   │   └── outbound
│   └── infrastructure
├── customer
│   ├── domain
│   ├── application
│   ├── adapters
│   └── infrastructure
└── shared
    ├── config
    └── common
```

Each **feature**:
- Owns its own domain, application, and adapters  
- Respects Clean/Hexagonal layering internally  
- Can evolve independently or be extracted later  

---

## 🧠 Why It Works

| Principle | Benefit |
|------------|----------|
| Feature-first | Keeps code cohesive and understandable |
| Layering within feature | Enforces clean boundaries |
| Isolation | Easier testing and refactoring |
| Scalability | Each feature can evolve separately |
| Spring alignment | Works naturally with component scanning |

This hybrid structure is exactly what many experienced Spring Boot engineers (Dan Vega included) use for production systems.

---

## 🧩 Take on All This

It boils down to **practicality over purity**.  
Remember do not to overcomplicate your first commit.

### 1. Start Simple
> “You don’t earn the right to a Clean Architecture until you have complexity to clean.”

Begin with package-by-feature. Refactor toward layers *as needed*.

### 2. Use Spring Idiomatically
Don’t fight the framework — let Spring handle DI and config. Keep your domain pure, but use `@Service`, `@Repository`, and `@RestController` where appropriate.

### 3. Organize for Humans
> “Your package structure should tell the story of your app — what it does, not just what tech it uses.”

### 4. Think in Use Cases
Replace CRUD-style services with explicit, business-driven use cases.

```java
interface PlaceOrderUseCase {
    OrderResponse placeOrder(OrderRequest request);
}
```

### 5. Design for Testability
> “If it’s hard to test, it’s in the wrong place.”

Keep domain logic testable without Spring.  
Controllers and repos should have their own slice tests.

### 6. Optimize Developer Experience
> “If your app takes longer to boot than it takes to pour a coffee, fix your architecture.”

Use fast feedback loops, Spring DevTools, and modular scanning.

### 7. Refactor Toward Clean
> “Good architecture is what lets you keep shipping features without hating yourself in six months.”

---

## 🧩 The Evolution Path

| Stage | Structure | Pros | Cons |
|--------|------------|------|------|
| **1. Package-by-layer** | `controller/service/repository` | Simple | Scattered logic |
| **2. Package-by-feature (flat)** | `order/OrderService.java` | Feature cohesion | Weak boundaries |
| **3. Package-by-feature + internal layers** | `order/domain/...` | Cohesion + discipline | Slightly more setup |

That’s the natural journey — and it’s exactly what Dan Vega (and most senior Spring engineers) would recommend.

---

## 💡 Key Takeaways

- **Package by feature** for cohesion.  
- **Layer within each feature** for clean boundaries.  
- **Evolve gradually** — architecture maturity comes with complexity.  
- **Use Spring idiomatically** — it already supports this style.  
- **Design for humans and testability** — that’s the real “clean.”

---

> “Group by business capability, structure by architectural boundary.”  
> — A principle for every scalable Spring Boot codebase.

---
