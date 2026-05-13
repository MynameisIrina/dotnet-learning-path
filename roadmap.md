# Backend Learning Roadmap

## Month 1 — Going Deeper Into What You Already Use

### Week 1 — EF Core Deep Dive

**Topics:**  
- change tracking  
- N+1 problem  
- IQueryable vs IEnumerable  
- AsNoTracking  
- eager/lazy/explicit loading  

**Assignment:**  
Implement 4 versions of the same query:
- naive
- Include
- Select projection
- AsSplitQuery

Include:
- SQL logs
- explanation of trade-offs

**Completion Criteria:**
- [ ] I can explain the N+1 problem in my own words and demonstrate it in code
- [ ] I understand how change tracking works (snapshot comparison)
- [ ] I know the difference between IQueryable and IEnumerable and can demonstrate it in generated SQL
- [ ] I created a `learning-journal.md`
- [ ] I submitted the assignment for code review

**Interview Topics:**  
- N+1
- change tracking
- AsNoTracking
- IQueryable vs IEnumerable
- split queries

---

### Week 2 — PostgreSQL: Indexes, Transactions, ACID

**Topics:**  
- EXPLAIN ANALYZE  
- indexes (B-tree, partial, composite)  
- ACID in depth  
- isolation levels  
- deadlocks  

**Assignment:**  
TBD after Week 1

---

### Week 3 — RabbitMQ for Real

**Topics:**  
- delivery guarantees  
- at-least-once delivery  
- idempotency  
- dead letter queues  
- retry policies  

**Assignment:**  
TBD

---

### Week 4 — Mock Interview #1 + Gap Analysis

**Format:**  
I ask questions as the interviewer, you answer.

Includes:
- answer review
- roadmap adjustments for Month 2

---

## Month 2 — Architecture, Testing, Resiliency

### Week 5 — Clean Architecture + DDD Basics

### Week 6 — Unit and Integration Testing

### Week 7 — Transactions, Outbox Pattern, Idempotency

### Week 8 — Mock Interview #2

---

## Month 3 — Production Engineering

### Week 9 — Docker + Docker Compose for Real

### Week 10 — CI/CD, GitHub Actions, VPS Deployment

### Week 11 — Observability

**Topics:**  
- logging  
- metrics  
- health checks  

### Week 12 — Mock Interview #3 + System Design Basics

---

# Pet Project

**Repository:**  
[link once created]

**Idea:**  
TBD during Weeks 2–3

**Stack:**  
- ASP.NET Core  
- PostgreSQL  
- EF Core  
- RabbitMQ  
- Redis  
- Docker  
- GitHub Actions  
- nginx  

---
