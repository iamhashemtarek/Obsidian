
## **What is Hangfire?**

**Hangfire** is a powerful and reliable **background job processing framework** for .NET applications. It enables developers to **schedule and execute background tasks** using **persistent storage** without relying on external Windows Services or separate applications.

---

## **Core Architecture**

After installing and configuring Hangfire, it operates through **three main components**:
### 1. **Client**
- Responsible for **creating and enqueueing jobs**.
    
- Typically used from your application’s services or controllers.
    
### 2. **Server**
- **Continuously polls the storage** for new jobs.
    
- Executes jobs using **background worker threads**.
    
- Can scale horizontally—multiple servers can process jobs in parallel.
    
### 3. **Storage**
- Used to **store job metadata, states, queues**, etc.
    
- Could be any of the supported storage backends (e.g., SQL Server, PostgreSQL, Redis).
    
- Although you can use your application's main database, it’s a **best practice to use a separate database** for Hangfire to follow **Separation of Concerns**.

![[Pasted image 20250505123628.png]]
---
## **Job Types in Hangfire**

| Type               | Description                                         | Availability |
| ------------------ | --------------------------------------------------- | ------------ |
| Fire-and-Forget    | Executes a job **immediately** in the background.   | Free         |
| Delayed            | Executes a job **after a specified time delay**.    | Free         |
| Recurring          | Executes jobs on a **scheduled basis (CRON)**.      | Free         |
| Continuations      | Executes a job **after the completion** of another. | Free         |
| Batches            | Groups of jobs **executed together**.               | Paid         |
| Batch Continuation | Run **additional jobs after a batch completes**.    | Paid         |

---

## **Best Practices**

- **Use separate storage** for Hangfire to avoid performance issues and promote modular design.
    
- **Keep background jobs stateless** and make sure their arguments are **serializable**.
    
- Monitor and manage jobs using the built-in **Hangfire Dashboard**.
    
- Secure the dashboard with **authorization** in production environments.
    
- Apply proper **retry strategies** to avoid data loss in case of failures.
    
- Organize jobs into **queues** to control priority and execution flow.
    