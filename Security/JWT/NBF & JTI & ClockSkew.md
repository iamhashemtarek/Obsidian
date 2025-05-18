## 🔐 1. `nbf` (Not Before)

**Definition**:

- `nbf` stands for **"Not Before"**.
    
- It tells **when the token becomes valid**.
    
- The token **should not be accepted** for processing **before this time**.
    

**Format**:

- A Unix timestamp (number of seconds since Jan 1, 1970 UTC).
    

**Use case**:

- If a token is issued now but should only be valid starting a minute later, you’d set `nbf` to `now + 60`.
    

**Example payload**:

```json
{
  "sub": "1234567890",
  "nbf": 1716171163
}
```

**Security benefit**:

- Prevents the token from being used **too early** (e.g., if the user schedules a token for future use).
    

---

## 🆔 2. `jti` (JWT ID)

**Definition**:

- `jti` stands for **"JWT ID"**.
    
- It’s a **unique identifier** for the token.
    

**Purpose**:

- Used to **identify and track** individual tokens.
    
- Helps to **prevent replay attacks** or allows you to implement **token blacklisting**.
    

**Example**:

```json
{
  "sub": "1234567890",
  "jti": "aa41cbb1-4893-46f3-97f4-0ae890fc69cf"
}
```

**Use case**:

- You can store `jti` values in a database to **invalidate** or **revoke** specific tokens.
    
- Example: When a user logs out, you store their token’s `jti` in a blocklist.
    

---

## ⏰ 3. `clockSkew`

**Definition**:

- Not part of the JWT itself — it’s a setting in your **JWT validation logic**.
    
- It defines an **allowed time drift** between the issuer and the consumer of the token (in case their clocks are not perfectly synchronized).
    

**Default in .NET**:

- 5 minutes (`ClockSkew = TimeSpan.FromMinutes(5)`)
    

**Purpose**:

- Prevents token rejection due to **small time differences** between systems.
    
- Applies to `exp`, `nbf`, and `iat` claims.
    

**Example in .NET**:

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ClockSkew = TimeSpan.FromMinutes(2), // Accept 2 minutes difference
        };
    });
```

---

## ✅ Summary Table

|Claim / Setting|Meaning|Type|Purpose|
|---|---|---|---|
|`nbf`|Not before|Unix time|Token not valid before this time|
|`jti`|JWT ID|String|Unique token ID for tracking/revoking|
|`clockSkew`|Time tolerance in validation|TimeSpan (server setting)|Allowable time difference in validation|

---

## ✅ Example Use in Real API

Imagine your API accepts JWTs and you want:

- Tokens valid only **after login time** → set `nbf`
    
- Ability to revoke a token → use `jti`
    
- Different servers might be a few seconds out of sync → configure `clockSkew`
    
