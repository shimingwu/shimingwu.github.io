---
title: Distributed Database System (9)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-26 19:53:43
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## Query Processing

Translates query into a query plan that minimizes some cost function
- Minimize total cost
  - CPU cost, I/O cost, & communication cost
- Minimize response time
  - Time elapsed for query execution

### Steps
1. Query rewriting
   - Query decomposition
   - Data localization
2. Global query optimization
   - Finds an optimal execution plan for query
3. Distributed query execution
   - Executes query plan to compute query result

   