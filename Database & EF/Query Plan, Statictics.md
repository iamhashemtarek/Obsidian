
### 🧠 Part 1: What Are **Execution Plans** in SQL Server?

An **execution plan** is a **blueprint** SQL Server generates to **retrieve your data in the most efficient way**.

- When you send a query (`SELECT`, `COUNT`, `JOIN`, etc.), SQL Server doesn’t just run it blindly.
    
- Instead, it asks:  
    ❓ “What’s the fastest way to get this data based on indexes, row count, etc.?”
    

### 🔧 The steps:

1. **Query Parser**: Parses your SQL into a tree structure.
    
2. **Algebrizer**: Resolves names, data types, aggregates.
    
3. **Query Optimizer**:
    
    - Explores multiple **execution strategies (plans)**.
        
    - Uses **statistics** to **estimate cost** of each strategy.
        
    - Picks the **least-cost plan**.
        

### 🧩 Types of Plans

|Plan Type|Description|
|---|---|
|**Clustered Index Scan**|Reads entire table via PK. Costly for big tables.|
|**Index Seek**|Uses a narrow index to find specific rows fast.|
|**Key Lookup**|Fetches missing columns from clustered index.|
|**Nested Loops / Hash Join**|Join algorithms based on row estimates.|

You can see them in SSMS by using:

```sql
SET SHOWPLAN_ALL ON;
```

or simply click **Include Actual Execution Plan** in SSMS.

---

### 📊 Part 2: What Are **Statistics**?

**Statistics** are metadata that describe **data distribution** in columns.

They help the optimizer estimate:

|Question|Example|
|---|---|
|How many rows match `WHERE Id = 5`?|1 or 5000?|
|How many rows in a JOIN?|Small or huge?|
|Is the column evenly distributed?|Yes/No?|

### 🔍 What’s Inside a Statistic?

Each statistic contains:

- A **histogram**: distribution of values for **up to 200 steps**
    
- **Density**: measure of uniqueness
    
- **Cardinality estimation**: row count predictions
    

```sql
DBCC SHOW_STATISTICS('Customers', 'IX_Customers_Id')
```

---

### 📈 Why Statistics Matter

Let’s say your query is:

```sql
SELECT COUNT(*) FROM Customers WHERE Country = 'US'
```

If stats say 90% of customers are in the US:

- SQL Server might **scan**
    

If stats say 1% are in the US:

- SQL Server might **seek**, then **lookup**
    

💡 Wrong estimation = bad plan = slow query or timeout

---

## ⚙️ Part 3: How Execution Plans and Statistics Interact

The **optimizer uses statistics to make row count guesses**. These guesses drive decisions like:

- Which **indexes** to use
    
- Whether to use **seek vs scan**
    
- Which **join algorithm** to use
    

#### ❗If the guesses are wrong:

- SQL Server might use a bad plan
    
- Might allocate too much memory (spill to tempdb)
    
- Or underestimate and go serial instead of parallel
    

---

## 🧪 Example of Statistics Affecting the Plan

```sql
-- Suppose 99% of rows have Status = 'Active'
SELECT * FROM Users WHERE Status = 'Inactive';
```

If stats are fresh:

- Optimizer might know it's rare and **use index seek**.
    

If stats are stale:

- Optimizer thinks it's common → **uses table scan** → slow
    

---

## 🏗️ Part 4: How EF Core Interacts With Statistics and Plans

EF Core sends **parameterized queries** to SQL Server:

```csharp
await context.Users.Where(u => u.Id == 5).FirstOrDefaultAsync();
```

Turns into:

```sql
exec sp_executesql N'SELECT TOP 1 * FROM Users WHERE Id = @id', N'@id int', @id=5
```

### 🔄 This interacts with:

|Component|Behavior|
|---|---|
|**Query Plan Cache**|SQL Server may reuse the execution plan if it thinks the shape of the query is the same|
|**Parameter Sniffing**|SQL Server might **use the plan compiled for the first parameter**, even if it's bad for later ones|
|**Statistics**|Used to build plan at compile time|
|**Execution Plan Reuse**|Good for performance — but can be dangerous when parameters vary a lot (see below)|

---

## ⚠️ Parameter Sniffing: A Silent Killer in EF Core

Let’s say:

```csharp
await context.Customers.Where(c => c.Country == "US").CountAsync();
```

If the first call uses `"US"` and most customers are in the US:

- Plan chooses **scan**
    
- Next call asks for `"EG"` (rare) → plan reused → still uses scan → bad plan
    

🩹 Fixes:

- Use `AsNoTracking().ToListAsync()` to avoid reuse (sometimes)
    
- Use `OPTION (RECOMPILE)` in raw SQL
    
- Use `DbCommandInterceptor` to append query hints
    

---

## 🛠️ Tips to Manage This in EF Core

### ✅ 1. Warm up long queries manually on app start

To ensure good plan/statistics are ready

### ✅ 2. Manually update statistics:

```sql
UPDATE STATISTICS dbo.Users;
```

### ✅ 3. Use filtered indexes or computed columns

To improve plan selection

### ✅ 4. Avoid huge `Include()`s or `.Select()` heavy objects when counting

### ✅ 5. Monitor actual vs estimated row counts in execution plans

---

## 📌 Summary Table

|Concept|Purpose|EF Impact|
|---|---|---|
|Execution Plan|Tells SQL Server how to run query|EF's parameterized queries use cached plans|
|Statistics|Help optimizer estimate rows and pick best plan|Must be fresh or EF queries may timeout|
|Parameter Sniffing|Reuse of first execution plan for all calls|Dangerous when first call is not representative|
|Plan Cache|Stores plans to avoid recompilation|Great for performance, bad when queries vary|

---

## ✅ Tools to Analyze

- **SSMS Execution Plan Viewer**
    
- `DBCC FREEPROCCACHE` (clears plan cache – use carefully!)
    
- `DBCC SHOW_STATISTICS`
    
- **Query Store** (SQL Server 2016+): monitors plan performance over time
    
