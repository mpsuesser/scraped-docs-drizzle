---
url: https://orm.drizzle.team/docs/cockroach/set-operations
title: "Set Operations"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Set Operations

SQL set operations combine the results of multiple query blocks into a single result. The SQL standard defines the following three set operations: `UNION`, `INTERSECT`, `EXCEPT`, `UNION ALL`, `INTERSECT ALL`, `EXCEPT ALL`.

### Union

Combine all results from two query blocks into a single result, omitting any duplicates.

Get all names from customers and users tables without duplicates.

import-pattern

builder-pattern

schema.ts

```typescript
import { union } from 'drizzle-orm/cockroach-core'
import { users, customers } from './schema'

const allNamesForUserQuery = db.select({ name: users.name }).from(users);

const result = await union(
    allNamesForUserQuery,
    db.select({ name: customers.name }).from(customers)
).limit(10);
```
```sql
(select "name" from "sellers")
union
(select "name" from "customers")
limit $1 -- params: [10]
```

### Union All

Combine all results from two query blocks into a single result, with duplicates.

Let’s consider a scenario where you have two tables, one representing online sales and the other representing in-store sales. In this case, you want to combine the data from both tables into a single result set. Since there might be duplicate transactions, you want to keep all the records and not eliminate duplicates.

import-pattern

builder-pattern

schema.ts

```typescript
import { unionAll } from 'drizzle-orm/cockroach-core'
import { onlineSales, inStoreSales } from './schema'

const onlineTransactions = db.select({ transaction: onlineSales.transactionId }).from(onlineSales);
const inStoreTransactions = db.select({ transaction: inStoreSales.transactionId }).from(inStoreSales);

const result = await unionAll(onlineTransactions, inStoreTransactions);
```
```sql
(select "transaction_id" from "online_sales")
union all
(select "transaction_id" from "in_store_sales")
```

### Intersect

Combine only those rows which the results of two query blocks have in common, omitting any duplicates.

Suppose you have two tables that store information about students’ course enrollments. You want to find the courses that are common between two different departments, but you want distinct course names, and you’re not interested in counting multiple enrollments of the same course by the same student.

In this scenario, you want to find courses that are common between the two departments but don’t want to count the same course multiple times even if multiple students from the same department are enrolled in it.

import-pattern

builder-pattern

schema.ts

```typescript
import { intersect } from 'drizzle-orm/cockroach-core'
import { depA, depB } from './schema'

const departmentACourses = db.select({ courseName: depA.courseName }).from(depA);
const departmentBCourses = db.select({ courseName: depB.courseName }).from(depB);

const result = await intersect(departmentACourses, departmentBCourses);
```
```sql
(select "course_name" from "department_a_courses")
intersect
(select "course_name" from "department_b_courses")
```

### Intersect All

Combine only those rows which the results of two query blocks have in common, with duplicates.

Let’s consider a scenario where you have two tables containing data about customer orders, and you want to identify products that are ordered by both regular customers and VIP customers. In this case, you want to keep track of the quantity of each product, even if it’s ordered multiple times by different customers.

In this scenario, you want to find products that are ordered by both regular customers and VIP customers, but you want to retain the quantity information, even if the same product is ordered multiple times by different customers.

import-pattern

builder-pattern

schema.ts

```typescript
import { intersectAll } from 'drizzle-orm/cockroach-core'
import { regularCustomerOrders, vipCustomerOrders } from './schema'

const regularOrders = db.select({ 
    productId: regularCustomerOrders.productId,
    quantityOrdered: regularCustomerOrders.quantityOrdered }
).from(regularCustomerOrders);

const vipOrders = db.select({ 
    productId: vipCustomerOrders.productId,
    quantityOrdered: vipCustomerOrders.quantityOrdered }
).from(vipCustomerOrders);

const result = await intersectAll(regularOrders, vipOrders);
```
```sql
(select "product_id", "quantity_ordered" from "regular_customer_orders")
intersect all
(select "product_id", "quantity_ordered" from "vip_customer_orders")
```

### Except

For two query blocks A and B, return all results from A which are not also present in B, omitting any duplicates.

Suppose you have two tables that store information about employees’ project assignments. You want to find the projects that are unique to one department and not shared with another department, excluding duplicates.

In this scenario, you want to identify the projects that are exclusive to one department and not shared with the other department. You don’t want to count the same project multiple times, even if multiple employees from the same department are assigned to it.

import-pattern

builder-pattern

schema.ts

```typescript
import { except } from 'drizzle-orm/cockroach-core'
import { depA, depB } from './schema'

const departmentACourses = db.select({ courseName: depA.projectsName }).from(depA);
const departmentBCourses = db.select({ courseName: depB.projectsName }).from(depB);

const result = await except(departmentACourses, departmentBCourses);
```
```sql
(select "projects_name" from "department_a_projects")
except
(select "projects_name" from "department_b_projects")
```

### Except All

For two query blocks A and B, return all results from A which are not also present in B, with duplicates.

Let’s consider a scenario where you have two tables containing data about customer orders, and you want to identify products that are exclusively ordered by regular customers (without VIP customers). In this case, you want to keep track of the quantity of each product, even if it’s ordered multiple times by different regular customers.

In this scenario, you want to find products that are exclusively ordered by regular customers and not ordered by VIP customers. You want to retain the quantity information, even if the same product is ordered multiple times by different regular customers.

import-pattern

builder-pattern

schema.ts

```typescript
import { exceptAll } from 'drizzle-orm/cockroach-core'
import { regularCustomerOrders, vipCustomerOrders } from './schema'

const regularOrders = db.select({ 
    productId: regularCustomerOrders.productId,
    quantityOrdered: regularCustomerOrders.quantityOrdered }
).from(regularCustomerOrders);

const vipOrders = db.select({ 
    productId: vipCustomerOrders.productId,
    quantityOrdered: vipCustomerOrders.quantityOrdered }
).from(vipCustomerOrders);

const result = await exceptAll(regularOrders, vipOrders);
```
```sql
(select "product_id", "quantity_ordered" from "regular_customer_orders")
except all
(select "product_id", "quantity_ordered" from "vip_customer_orders")
```
