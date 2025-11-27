# Global Architecture

## Justification
- **High load (1M+ visitors/day)** → offload static content to **AWS S3 + CloudFront**.
- **Next.js**: SSR/SSG/ISR for fast loading & SEO.
- **TailwindCSS**: fast UI development.
- **NestJS**: modular backend, SOLID principles.
- **Auth**: JWT (access + refresh), optional OAuth (Google, Apple, Steam). REST APIs preferred.

---

## Tech Stack

**Frontend (Next.js)**  
- Next.js (App Router, SSR/SSG/ISR)  
- TypeScript, TailwindCSS  
- REST API, optional React Query / SWR  
- Auth: NextAuth / OAuth  
- Static assets via **S3 + CloudFront**  

**Backend (NestJS)**  
- NestJS (modular, DI, REST)  
- Node.js LTS  
- Campaigns, personalization, A/B testing, auth  
- Auth: JWT, optional RBAC/ABAC  
- MongoDB (dynamic data), Redis (caching, sessions, AB rules)  
- Async tasks: **AWS SQS + Worker**  

**Worker / Queue Processor**  
- Handles emails, notifications, media processing, A/B logs  
- Updates MongoDB after tasks completion  

---

## Databases & Storage
- MongoDB: users, campaigns, AB logs, progress  
- Redis: caching, rate limiting, sessions  
- ORM: TypeScript Mongoose / Prisma  
- AWS S3: static assets  
- AWS SQS: message queues  

---

## Cloud & Infrastructure
- CDN: AWS CloudFront  
- SSL: Certbot / AWS Certificate Manager  
- IaC: AWS CDK  

**Networking & Proxy**: Nginx (reverse proxy + SSL termination)  

---

## CI / CD
- GitHub Actions: lint, test, build, deploy  
- Docker + Docker-compose  

---

## Monitoring & Observability
- Sentry (frontend & backend)  
- Prometheus + Grafana or CloudWatch  
- Healthcheck endpoints optional  
- OpenTelemetry optional  
- Rate limiting / throttling via NestJS + Redis  
- Alerts via CloudWatch Alarms  

---

## Testing
- Unit: Jest  
- Frontend: Playwright / Cypress  

---

## Developer Experience
- ESLint + Prettier  
- Husky + Lint-staged  
- Recommended VSCode settings & extensions

---
# QA

> Framework Choice: Which frontend framework would you choose, and why? Consider static vs dynamic content needs

- Hybrid Rendering: Next.js supports SSR (Server-Side Rendering), SSG (Static Site Generation), and ISR (Incremental Static Regeneration). This flexibility allows you to optimize for both static content (fast-loading, SEO-friendly pages) and dynamic content (user-specific dashboards, personalized campaigns).
- SEO Benefits: Server-rendered pages improve indexing and search engine performance, crucial for content-heavy or public-facing platforms.
- Developer Productivity: Built on React, widely adopted with large ecosystem; combined with TailwindCSS, it accelerates frontend development.
- Scalability: Supports code splitting, image optimization, and CDN integration (like AWS CloudFront) for handling high traffic.

> Static Content Delivery: How would you handle the large volume of static content (images, CSS, JavaScript)? Be specific about tools and techniques. (Think about build processes, optimization, etc.)

- Store all static assets (images, videos, CSS, JS) in AWS S3.
- Distribute globally via AWS CloudFront (CDN) to reduce latency and backend load.
- Optimize and convert to modern formats (WebP, AVIF) during build using tools like Next.js
or using Sharp on Backend service.
- Pre-generate static pages where possible (SSG) and update periodically (ISR) to reduce server load.
- Load images/videos lazily to reduce initial page weight.
- Serve assets directly from CDN URLs to bypass backend servers entirely.

> Personalization: How would you integrate personalized content into the frontend, considering data is provided by a REST API based service?

- Frontend requests personalized content from the backend via REST endpoints (e.g., /user/profile, /campaigns/personalized).
- Store API responses in a state management layer or caching library:
- React Query or SWR for caching, revalidation, and background updates.
- Design modular React components that can accept user-specific props (e.g., banners, recommended products, A/B variants).
- Cache common personalization rules in Redis to reduce API latency.

> Backend Design: Would you favor a monolithic backend or a microservices architecture? Explain your reasoning.
- For a new platform, a modular monolith (e.g., NestJS with clear modules for campaigns, personalization, auth) allows faster development, easier testing, and simpler deployment.
- Monolithic services can reduce inter-service network calls, improving latency for high-traffic operations (important for 1M+ visitors/day).
- Start with a modular monolith to optimize development speed, maintainability, and operational simplicity. As the platform grows, high-demand modules (e.g., personalization engine, media processing) can be gradually extracted into microservices.

> Database Selection: What type(s) of databases would you use? (e.g., Relational, NoSQL, Graph). Explain your choice(s) based on the data requirements (user data, game data, campaign data, etc.).

- MongoDB Ideal for dynamic and semi-structured data like users, campaigns, game progress, and A/B testing logs.
- Flexible schema allows rapid iteration without strict migrations.
- Supports fast queries for personalization and analytics.

- Redis (In-Memory Key-Value Store): Used for caching, session management, rate limiting, and fast retrieval of frequently accessed rules (e.g., A/B test decisions).

## Why Not Relational/Graph Initially:

Relational DBs add schema rigidity; frequent changes to campaign or user data structures could slow development.

Graph DBs are useful only if complex relationships (social graphs, recommendations) become central; not required at launch.

> Caching Strategy: Describe your caching strategy.
- Cache frequently accessed data such as user sessions, personalization rules, A/B test configurations, leaderboard/game stats.

- Reduce latency for high-traffic endpoints and offload read pressure from MongoDB.
- Use stale-while-revalidate or time-based caching for REST API responses that don’t change every request (e.g., campaign banners, promotional content).
- Store images, CSS, JS, and video assets in S3 + CloudFront
- Implement cache invalidation on updates (e.g., when a campaign is modified, Redis cache entries are cleared).

> API Design: Briefly outline how you would structure your APIs to serve the frontend.

- Organize APIs around resources: /users, /campaigns, /games, /notifications, /analytics
- Use HTTP verbs consistently: GET (read), POST (create), PUT/PATCH (update), DELETE (remove).
- OAuth for third-party login
- Support query parameters like `?limit=20&page=2` or `?status=active` for scalable data retrieval
- Version APIs via URL (/api/v1/...) to allow safe backward-compatible changes.
- Cache frequently accessed endpoints (Redis, CDN).

> Deployment Strategy: How would you deploy and manage the websites? (e.g., Docker, Kubernetes, Serverless). Explain your reasoning.

- Package both frontend (Next.js) and backend (NestJS) as Docker containers to ensure consistent environments across development, staging, and production.
- Use Docker Compose for local development and multi-container orchestration (frontend, backend, Redis, MongoDB, workers
- Lightweight compared to Kubernetes; easier to manage if your team is small and has only Docker experience.

> Scalability: How would your architecture handle peak traffic (e.g., during game launches)? What components would you scale, and how?

- Backend design (by SOLID principle) allows easy extend logic.
- Queue tasks via AWS SQS and scale worker containers horizontally to handle bursts (emails, notifications, media processing, A/B logs).

> Monitoring & Alerting
- Use Sentry for both frontend and backend to capture exceptions, stack traces, and user-impacting errors.
- Track system metrics (CPU, memory, request latency, database performance) via Prometheus + Grafana

> Database Costs
- MongoDB: Charges depend on instance size; high traffic or large datasets can increase costs.
We can Start with smaller instances and scale vertically or horizontally as needed. After we can make sharding
- By right-sizing instances, using read replicas, caching frequently accessed data, and archiving old records


# Evaluation Criteria Analysis

## 1. Scalability (20%)
- Horizontal scaling of **frontend/backend Docker containers**.  
- **Redis caching** and **CloudFront CDN** reduce backend load.  
- Asynchronous tasks handled via **SQS + worker containers**, supporting peak traffic events (e.g., game launches).

## 2. Cost-Effectiveness (15%)
- **Docker containers** avoid over-provisioning and ensure reproducibility.  
- **MongoDB + Redis hybrid** with read replicas and caching reduces DB costs.  
- **S3 + CloudFront** efficiently serves static content, lowering server load.

## 3. Maintainability & Complexity (15%)
- **Modular monolith (NestJS)** with clear separation of concerns.  
- Popular frameworks (**Next.js, TailwindCSS**) with strong community support and good dev/hour cost
- modular design simplify testing, debugging, and future refactoring.

## 4. Deployment & CI/CD (15%)
- **Docker + Docker Compose** ensures consistent environments across dev/staging/prod.  
- **GitHub Actions** for linting, testing, building, and deployment.  

## 5. Technical Depth & Justification (20%)
- Architectural choices (Next.js SSR/SSG, Redis caching, MongoDB, SQS queues) justified for performance, personalization, and high traffic.  
- Hybrid caching, modular backend, and queue-based async processing demonstrate strong technical understanding.

## 6. Completeness & Clarity (15%)
- Architecture covers **frontend, backend, workers, DB, caching, CI/CD, monitoring, and deployment**.  

---
---
---

# Legacy Web Application Performance Troubleshooting

## 1. Frontend Analysis
- Review all pages using a **profiler** (Chrome DevTools, Lighthouse, etc.).  
- Identify what is **heavily loading the call stack** — long-running JavaScript, memory leaks, or infinite loops.  
- Detect bottlenecks in rendering or inefficient client-side code.

## 2. Backend Investigation
- If frontend is not the issue, move to the **backend**.  
- Monitor **TTFB (Time To First Byte)** for all endpoints to find slow controllers.  
  - Tools: New Relic, Datadog, or custom PHP logging middleware.  
- Identify endpoints with high latency or high variability.

## 3. Database Analysis
- Inspect queries from problematic controllers.  
- Check for **inefficient queries, missing indexes, or excessive repeated queries**.  
- Optimize queries: add indexes, batch requests, refactor to reduce load.

## 4. Scaling Considerations
- If database load remains high, consider **splitting tables, sharding, or partitioning** to improve performance.

## Tools & Techniques Summary
- **Frontend:** Chrome DevTools profiler, Lighthouse, memory snapshots.  
- **Backend:** TTFB monitoring, slow controller logs, request tracing.  
- **Database:** MySQL EXPLAIN, slow query log, indexing, query optimization.  
- **Optional Scaling:** CDN, caching, read replicas.

## Conclusion
Start with the frontend to catch rendering or JS issues, then systematically move to backend and database analysis. Optimize queries and structure data appropriately to address bottlenecks without a full rewrite.
