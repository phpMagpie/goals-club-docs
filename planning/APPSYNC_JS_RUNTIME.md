# AppSync JavaScript Runtime Guidelines

**Last Updated:** May 11, 2026

This document covers the constraints and best practices for writing AppSync resolvers using the JavaScript (APPSYNC_JS) runtime with Aurora MySQL / RDS Data Source.

---

## 🚨 MOST COMMON MISTAKES

### 1. Multiple SQL Statements
```javascript
// ❌ WRONG - createMySQLStatement only accepts 1-2 arguments
return createMySQLStatement(sql`...`, sql`...`, sql`...`);

// ✅ CORRECT - Maximum 2 statements per call
return createMySQLStatement(sql`...`, sql`...`);

// For more operations, use a pipeline resolver with multiple functions
```

### 2. Loops
```javascript
// ❌ WRONG - for, while, for...of, for...in NOT supported
for (let i = 0; i < arr.length; i++) { }
while (condition) { }

// ✅ CORRECT - Use array methods
arr.forEach((item) => { });
arr.map((item) => item.value);
```

### 3. Date Objects
```javascript
// ❌ WRONG - new Date() not supported
const date = new Date("2027-01-01");

// ✅ CORRECT - Use string comparison or util.time
const now = util.time.nowISO8601();
const isBeforeCutoff = dateStr < "2027-01-01";
```

### 4. JSON Parsing
```javascript
// ❌ WRONG - JSON.parse may not work reliably
const obj = JSON.parse(jsonString);

// ✅ CORRECT - Manual string parsing with indexOf/substring
const value = extractStringValue(jsonString, 'key');
```

### 5. String() Constructor
```javascript
// ❌ WRONG - String() not supported
const str = String(value);

// ✅ CORRECT - Use concatenation to coerce to string
const str = "" + value;
```

### 6. parseInt() / parseFloat()
```javascript
// ❌ WRONG - parseInt not supported
const num = parseInt(str, 10);

// ✅ CORRECT - Use unary + or Number()
const num = +str;
// or
const num = str * 1;
```

---

## ⚠️ Critical Limitations

The AppSync JS runtime is **NOT** Node.js. It's a subset of JavaScript with significant restrictions designed for fast, sandboxed execution.

### Syntax Restrictions

| ❌ NOT Supported | ✅ Use Instead |
|------------------|----------------|
| `for` loops | `forEach()`, `map()`, `filter()`, `reduce()` |
| `for...of` loops | `forEach()` or `Array.from().forEach()` |
| `for...in` loops | `Object.keys().forEach()` |
| `while` loops | Use `indexOf()` to find delimiters, or recursive functions |
| `++` / `--` operators | `+= 1` / `-= 1` |
| Regex literals `/pattern/` | `new RegExp()` or string methods (`split()`, `indexOf()`, `replace()`) |
| `new Date()` | String comparison or `util.time` functions |
| `Math.cos()` / `Math.sqrt()` / `Math.PI` | Pre-computed constants or SQL-side calculation |
| `.sort()` with callbacks | Not supported — move sorting to frontend or SQL `ORDER BY` |
| `JSON.parse()` | Manual string parsing with `indexOf()`/`substring()` |
| `String(value)` | `"" + value` (concatenation to coerce) |
| `parseInt()` / `parseFloat()` | `+str` or `str * 1` (unary plus or multiply) |
| `async` / `await` | Not needed - resolver model handles async |
| `try` / `catch` | Check `ctx.error` in response handler |
| Template literal tags | Plain template literals work fine |
| Classes | Object literals and functions only |
| Generators / iterators | Array methods |
| Spread in function calls | `fn(...args)` — works in most cases |
| Dynamic imports | Not supported |
| `createMySQLStatement` with 3+ args | Use pipeline resolver with multiple functions |
| `util.time.parseFormattedToEpochMilliSeconds()` | Unreliable — avoid. Use string comparison or SQL date functions |

### What DOES Work

```javascript
// ✅ Arrow functions
const format = (str) => str.toUpperCase();

// ✅ Template literals
const query = `SELECT * FROM users WHERE id = '${id}'`;

// ✅ Array methods
const names = users.map(u => u.name);
const active = users.filter(u => u.status === 'ACTIVE');

// ✅ Object destructuring
const { id, name } = ctx.args;

// ✅ Spread operator (objects and arrays)
const merged = { ...defaults, ...overrides };

// ✅ Ternary operators
const status = count >= target ? 'COMPLETE' : 'IN_PROGRESS';

// ✅ String methods
const date = isoString.substring(0, 10);
const formatted = timestamp.replace('T', ' ');

// ✅ JSON methods
const parsed = JSON.parse(ctx.args.metadata);
const serialized = JSON.stringify(data);

// ✅ Math operations
const rounded = Math.round(value);
const maxVal = Math.max(a, b);

// ✅ util module
import { util } from '@aws-appsync/utils';
const id = util.autoUlid();
const now = util.time.nowISO8601();
```

---

## 🗄️ RDS Data Source Patterns

### Basic Query with `sql` Template

```javascript
import { sql, createMySQLStatement, toJsonObject } from '@aws-appsync/utils/rds';

export function request(ctx) {
  const { id } = ctx.args;
  return createMySQLStatement(
    sql`SELECT * FROM users WHERE id = ${id}`
  );
}

export function response(ctx) {
  if (ctx.error) {
    return util.error(ctx.error.message, ctx.error.type);
  }
  const rows = toJsonObject(ctx.result);
  return rows[0]?.[0] || null;
}
```

### Multiple Statements (Same Request)

You can pass up to **2 SQL statements** to `createMySQLStatement()`:

```javascript
export function request(ctx) {
  return createMySQLStatement(
    sql`SELECT * FROM goals LIMIT ${limit} OFFSET ${offset}`,
    sql`SELECT COUNT(*) as total FROM goals`
  );
}

export function response(ctx) {
  const result = toJsonObject(ctx.result);
  const items = result[0] || [];
  const total = result[1]?.[0]?.total || 0;
  return { items, total };
}
```

**⚠️ Important:** Beyond 2 statements, use **Pipeline Resolvers** instead.

### Using `insert()` and `select()` Helpers

```javascript
import { createMySQLStatement, insert, select, toJsonObject } from '@aws-appsync/utils/rds';

export function request(ctx) {
  const id = util.autoUlid();
  return createMySQLStatement(
    insert({
      table: 'users',
      values: { id, name: ctx.args.name, email: ctx.args.email }
    }),
    select({
      table: 'users',
      columns: ['id', 'name', 'email', 'created_at'],
      where: { id: { eq: id } }
    })
  );
}
```

---

## 📅 Date/Time Formatting

**MySQL DATETIME format:** `YYYY-MM-DD HH:MM:SS`

**⚠️ Problem:** `util.time.nowISO8601()` returns `2026-03-08T14:30:45.123Z` which MySQL rejects.

**✅ Solution:** Strip the timezone and milliseconds:

```javascript
// Convert ISO 8601 to MySQL DATETIME
const nowIso = util.time.nowISO8601();
const mysqlDatetime = nowIso.substring(0, 19).replace('T', ' ');
// Result: "2026-03-08 14:30:45"

// For DATE only fields
const activityDate = input.activityDate.split('T')[0];
// Result: "2026-03-08"
```

**Converting MySQL DATETIME back to ISO 8601:**

```javascript
function toISO8601(mysqlDatetime) {
  if (!mysqlDatetime) return null;
  // "2026-03-08 14:30:45" → "2026-03-08T14:30:45.000Z"
  return mysqlDatetime.replace(' ', 'T') + '.000Z';
}
```

---

## 🔗 Pipeline Resolvers (Multi-Statement Operations)

When you need to execute **3+ SQL statements** or **conditional logic between statements**, use Pipeline Resolvers.

### When to Use Pipeline Resolvers

| Scenario | Solution |
|----------|----------|
| INSERT then SELECT (2 statements) | Single resolver with `createMySQLStatement(insert, select)` |
| INSERT + conditional UPDATE + SELECT | Pipeline resolver with 3 functions |
| Multiple tables need updating atomically | Pipeline resolver |
| Complex business logic between queries | Pipeline resolver |

### File Structure

```
packages/infra/modules/appsync-rds/
├── pipeline-functions/
│   ├── src/
│   │   ├── insertActivity.js      # Function 1
│   │   ├── getGoalPeriodInfo.js   # Function 2
│   │   └── upsertGoalPeriod.js    # Function 3
│   └── dist/                       # Built files
├── pipeline-resolvers/
│   ├── src/
│   │   └── logGoalActivity.js     # Pipeline resolver definition
│   └── dist/                       # Built files
└── resolvers/                      # Standard resolvers
```

### Pipeline Resolver Definition

```javascript
// pipeline-resolvers/src/logGoalActivity.js
//@type=Mutation
//@field=logGoalActivity
//@functions=insertActivity,getGoalPeriodInfo,upsertGoalPeriod

export function request(ctx) {
  return {};
}

export function response(ctx) {
  return ctx.prev.result;
}
```

### Pipeline Function Pattern

Each function has access to:
- `ctx.args` - Original GraphQL arguments
- `ctx.stash` - Shared data between functions (use this to pass data!)
- `ctx.prev.result` - Result from previous function

```javascript
// pipeline-functions/src/insertActivity.js
import { util } from '@aws-appsync/utils';
import { createMySQLStatement, insert, select, toJsonObject } from '@aws-appsync/utils/rds';

export function request(ctx) {
  const id = util.autoUlid();
  
  // Store data for subsequent functions
  ctx.stash.activityId = id;
  ctx.stash.userGoalId = ctx.args.input.userGoalId;
  
  return createMySQLStatement(
    insert({ table: 'activities', values: { id, ... } }),
    select({ table: 'activities', where: { id: { eq: id } } })
  );
}

export function response(ctx) {
  if (ctx.error) {
    return util.error(ctx.error.message, '400 Bad Request');
  }
  
  const rows = toJsonObject(ctx.result);
  const inserted = rows[1]?.[0];
  
  // Store result for final response
  ctx.stash.insertedActivity = inserted;
  
  return inserted;
}
```

### Conditional Execution in Pipeline

```javascript
// Function that may or may not need to run
export function request(ctx) {
  if (!ctx.stash.needsUpdate) {
    // Return a no-op query
    return createMySQLStatement(sql`SELECT 1 as skip`);
  }
  
  return createMySQLStatement(
    sql`UPDATE user_goal_periods SET ...`
  );
}

export function response(ctx) {
  // Always return the final result regardless of whether this function did work
  return ctx.stash.insertedActivity;
}
```

---

## 🔍 Debugging Tips

### Check for Errors

```javascript
export function response(ctx) {
  if (ctx.error) {
    // Log context for debugging (visible in CloudWatch)
    console.log('Error context:', JSON.stringify({
      error: ctx.error,
      args: ctx.args,
      stash: ctx.stash
    }));
    return util.error(ctx.error.message, ctx.error.type);
  }
  // ...
}
```

### Validate Input Early

```javascript
export function request(ctx) {
  const cognitoId = ctx.identity?.sub || ctx.identity?.claims?.sub;
  if (!cognitoId) {
    return util.error('Unauthorized', 'UnauthorizedException');
  }
  
  if (!ctx.args.input?.userGoalId) {
    return util.error('userGoalId is required', 'ValidationError');
  }
  // ...
}
```

### Return Meaningful Errors

```javascript
export function response(ctx) {
  if (ctx.error) {
    // Include helpful context in error message
    const msg = `Failed to create activity: ${ctx.error.message}`;
    return util.error(msg, '400 Bad Request');
  }
  // ...
}
```

---

## 📝 Code Style Guidelines

### Resolver File Header Comments

```javascript
//@type=Query          // or Mutation
//@field=listAllGoals  // GraphQL field name
```

### Camel Case Conversion

GraphQL uses camelCase, MySQL uses snake_case. Convert in response handlers:

```javascript
export function response(ctx) {
  const row = toJsonObject(ctx.result)[0]?.[0];
  if (!row) return null;
  
  return {
    id: row.id,
    userId: row.user_id,           // snake_case → camelCase
    goalId: row.goal_id,
    activityDate: row.activity_date,
    createdAt: toISO8601(row.created_at)
  };
}
```

### Enum Handling

Store enums as UPPERCASE in MySQL, return as UPPERCASE to GraphQL:

```javascript
// In SELECT
sql`SELECT UPPER(status) as status, UPPER(visibility) as visibility FROM goals`

// Or convert in response
return {
  status: row.status?.toUpperCase() || 'ACTIVE',
  visibility: row.visibility?.toUpperCase() || 'PRIVATE'
};
```

### Null Handling

```javascript
// Provide null fallbacks
const values = {
  notes: input.notes || null,
  location_lat: input.locationLat || null,
  image_url: input.imageUrl || null
};
```

---

## 🏗️ Build Process

### Makefile Targets

```makefile
# Build standard resolvers
build-resolvers:
	cd packages/infra/modules/appsync-rds/resolvers && bash build.sh

# Build pipeline functions
build-pipeline-functions:
	cd packages/infra/modules/appsync-rds/pipeline-functions && bash build.sh

# Build pipeline resolvers
build-pipeline-resolvers:
	cd packages/infra/modules/appsync-rds/pipeline-resolvers && bash build.sh

# Build all (runs before deploy)
build: build-shared build-resolvers build-pipeline-functions build-pipeline-resolvers
```

### Resolver Build Script

```bash
#!/bin/bash
# build.sh
mkdir -p dist
for file in src/**/*.js src/*.js; do
  if [ -f "$file" ]; then
    filename=$(basename "$file")
    cp "$file" "dist/$filename"
  fi
done
```

---

## 📚 Reference Links

- [AppSync JavaScript Runtime Documentation](https://docs.aws.amazon.com/appsync/latest/devguide/resolver-reference-js-version.html)
- [APPSYNC_JS Runtime Features](https://docs.aws.amazon.com/appsync/latest/devguide/resolver-util-reference-js.html)
- [RDS Data Source Utilities](https://docs.aws.amazon.com/appsync/latest/devguide/resolver-reference-rds-js.html)
- [Pipeline Resolvers](https://docs.aws.amazon.com/appsync/latest/devguide/pipeline-resolvers-js.html)

---

## 🚫 Common Mistakes to Avoid

### ❌ Using for loops
```javascript
// WRONG
for (let i = 0; i < items.length; i++) {
  results.push(process(items[i]));
}

// CORRECT
const results = items.map(item => process(item));
```

### ❌ Using increment operators
```javascript
// WRONG
count++;

// CORRECT
count += 1;
```

### ❌ Multiple SQL statements beyond 2
```javascript
// WRONG - Will fail or behave unexpectedly
return createMySQLStatement(
  sql`INSERT INTO activities ...`,
  sql`UPDATE user_goals ...`,
  sql`INSERT INTO user_goal_periods ...`,
  sql`SELECT * FROM activities ...`
);

// CORRECT - Use pipeline resolver with separate functions
```

### ❌ ISO 8601 dates to MySQL
```javascript
// WRONG
const created_at = util.time.nowISO8601();
// Results in "2026-03-08T14:30:45.123Z" → MySQL error

// CORRECT
const nowIso = util.time.nowISO8601();
const created_at = nowIso.substring(0, 19).replace('T', ' ');
// Results in "2026-03-08 14:30:45" → MySQL accepts
```

### ❌ Using async/await
```javascript
// WRONG - async/await is not supported
export async function request(ctx) {
  const result = await someAsyncOperation();
}

// CORRECT - AppSync handles async via request/response model
export function request(ctx) {
  return createMySQLStatement(...);
}
```

### ❌ Forgetting to check ctx.error
```javascript
// WRONG - Silent failure
export function response(ctx) {
  const rows = toJsonObject(ctx.result);
  return rows[0][0];
}

// CORRECT - Handle errors explicitly
export function response(ctx) {
  if (ctx.error) {
    return util.error(ctx.error.message, ctx.error.type);
  }
  const rows = toJsonObject(ctx.result);
  return rows[0]?.[0] || null;
}
```

### ❌ Using new Date()
```javascript
// WRONG - NewExpression not supported
const cutoff = new Date("2027-01-01");
const userCreated = new Date(stats.userCreatedAt);
return userCreated < cutoff;

// CORRECT - Use string comparison (ISO dates sort correctly)
const cutoff = "2027-01-01";
const userCreated = stats.userCreatedAt.substring(0, 10);
return userCreated < cutoff;

// OR use util.time for current time
const now = util.time.nowISO8601();
```

### ❌ Using while loops
```javascript
// WRONG - while statements not supported
let i = 0;
while (i < str.length) {
  if (str.charAt(i) === ',') break;
  i += 1;
}

// CORRECT - Use indexOf to find delimiters
const commaIdx = str.indexOf(',');
const result = commaIdx !== -1 ? str.substring(0, commaIdx) : str;
```

### ❌ Using .sort() with callbacks
```javascript
// WRONG - Both arrow functions and function expressions fail in .sort()
items.sort((a, b) => b.score - a.score);
items.sort(function(a, b) { return b.score - a.score; });

// CORRECT - Move sorting to frontend or use SQL ORDER BY
// In resolver: ORDER BY score DESC
// In frontend: data.sort((a, b) => b.score - a.score);
```

### ❌ Using Math.cos / Math.sqrt / Math.PI
```javascript
// WRONG - Trig and advanced Math functions not available
const distance = Math.sqrt(dx * dx + dy * dy);
const lngFactor = Math.cos(lat * Math.PI / 180);

// CORRECT - Use pre-computed constants or compare squared values
const distSq = dx * dx + dy * dy; // Compare squared distances instead
const lngFactor = 0.6; // Fixed factor, accurate enough for UK (50-58°N)
```

### ❌ Using JSON.parse()
```javascript
// WRONG - JSON.parse may not work reliably
const criteria = JSON.parse(badge.criteria);

// CORRECT - Manual string parsing with indexOf/substring
function extractStringValue(jsonStr, key) {
  const searchKey = '"' + key + '":"';
  const startIdx = jsonStr.indexOf(searchKey);
  if (startIdx === -1) return null;
  const valueStart = startIdx + searchKey.length;
  const valueEnd = jsonStr.indexOf('"', valueStart);
  return jsonStr.substring(valueStart, valueEnd);
}

const criteriaType = extractStringValue(badge.criteria, 'type');
```

### ❌ More than 2 SQL statements in createMySQLStatement
```javascript
// WRONG - createMySQLStatement only accepts 1-2 arguments
return createMySQLStatement(
  sql`INSERT INTO table1 ...`,
  sql`INSERT INTO table2 ...`,
  sql`INSERT INTO table3 ...`  // ERROR: Expected 1-2 arguments, but got 3
);

// CORRECT - Limit to 2 statements per call
return createMySQLStatement(
  sql`INSERT INTO table1 ...`,
  sql`INSERT INTO table2 ...`
);

// For more operations, use a pipeline resolver with multiple functions
```


