# ps3-starter.ipynb

DuckDB Query Reviewer
A lightweight Python utility for analytics engineers to detect common SQL pitfalls in DuckDB SELECT queries before they reach a dashboard or a board slide.
🎯 Purpose
In complex analytics, "simple" joins often introduce subtle but expensive errors. This tool identifies two primary risks:
1. Join Fan-out: When a one-to-many join multiplies the row count, leading to inflated aggregates (e.g., double-counting revenue).
2. The Null Drop: When filters on nullable columns silently exclude data due to SQL's Three-Valued Logic (TRUE, FALSE, UNKNOWN).
🚀 How it Works
The reviewer uses a combination of schema inspection and execution-time heuristics:
- Fan-out Detection: It compares the row count of the base table (the first table in the FROM clause) against the final result set. If the result is larger, it flags a potential fan-out.
- Nullability Check: It uses PRAGMA table_info to find nullable columns. If those columns appear in a WHERE clause without explicit IS NULL or IS NOT NULL handling, it warns of potential data loss.
- Aggregate Risk: It flags any query that contains both a JOIN and an aggregate function (SUM, COUNT, etc.), as this is the most common pattern for double-counting.
🛠 Usage
Installation
Ensure you have duckdb and pandas installed:
pip install duckdb pandas
Implementation
import duckdb
from review_query import review_query

con = duckdb.connect('my_database.db')
sql = "SELECT SUM(amount) FROM customers JOIN orders ON customers.id = orders.customer_id"

warnings = review_query(con, sql)
for w in warnings:
    print(f"[{w['severity']}] {w['type']}: {w['message']}")
⚠️ Limitations
- Regex Parsing: The tool uses regular expressions. It may miss filters located in subqueries or JOIN ... ON conditions.
- Execution Required: To detect fan-out, the tool must execute the query. This can be resource-intensive for extremely large datasets.
- Heuristic-based: It flags the risk of fan-out aggregates. A query using a safe 1:1 join will still trigger a warning, requiring a human to verify the relationship.
