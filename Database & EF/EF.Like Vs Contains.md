

---

## 🔍 `EF.Functions.Like` vs `Contains` in Entity Framework

|Expression|SQL Translation|Index Usage|Wildcard Position|Performance|Use Case|
|---|---|---|---|---|---|
|`EF.Functions.Like(x.Name, "%term")`|`LIKE '%term'`|❌ **Cannot use index**|Suffix match|🚫 **Slow** for large data|Ends with `term`|
|`EF.Functions.Like(x.Name, "%term%")`|`LIKE '%term%'`|❌ **Cannot use index**|Substring match|🚫 **Slow**|Contains `term`|
|`x.Name.Contains("term")`|`LIKE '%term%'`|❌ **Same as above**|Substring match|🚫 **Slow**|Contains `term`|
|`EF.Functions.Like(x.Name, "term%")`|`LIKE 'term%'`|✅ **Uses index**|Prefix match|✅ **Fast**|Starts with `term`|

---

## 💡 Key Differences

1. **Index Usage**:
    
    - Only `LIKE 'term%'` (prefix match) can use a database index efficiently.
        
    - `%term` or `%term%` cannot use the index and require full table scan.
        
2. **Performance**:
    
    - `Contains` and `EF.Functions.Like('%term%')` are **equivalent** in behavior and performance — both translate to `LIKE '%term%'`.
        
3. **Behavior**:
    
    - `EF.Functions.Like` gives you control over wildcards (`%`, `_`) explicitly.
        
    - `Contains` is more readable, but less flexible (only supports `'%term%'` pattern).
        

---

## ✅ When to Use What?

|Scenario|Recommended|
|---|---|
|**Prefix match** (e.g., names starting with "Ali")|`EF.Functions.Like(x.Name, "Ali%")` ✅|
|**Flexible pattern with custom wildcards**|`EF.Functions.Like(...)` ✅|
|**Simple "contains" logic**|`x.Name.Contains("Ali")` (simpler syntax) 👍|
|**Performance-sensitive + large data**|Avoid `%term` or `%term%` ❌|

---

