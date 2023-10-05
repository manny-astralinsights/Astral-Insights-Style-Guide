# Astral-Insights-Style-Guide


#  SQL Style Guide



## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql
with hubspot_interest AS (

   SELECT
        email,
        timestamp_millis(property_beacon_interest) AS expressed_interest_at
    FROM hubspot.contact
    WHERE
        property_beacon_interest is not null

), 

support_interest AS (

    SELECT
        conversation.email,
        conversation.created_at AS expressed_interest_at
   FROM helpscout.conversation
    INNER JOIN helpscout.conversation_tag on conversation.id = conversation_tag.conversation_id
    WHERE
        conversation_tag.tag = 'beacon-interest'

), 

combined_interest AS (

    SELECT * FROM hubspot_interest
    UNION ALL
    SELECT * FROM support_interest

),

first_interest AS (

    SELECT
        email,
        MIN(expressed_interest_at) as expressed_interest_at
   FROM combined_interest
   GROUP BY email

)

SELECT * FROM first_interest
```
## Guidelines

### Use Uppercase SQL

It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
SELECT * FROM users

-- Bad
select * from users

-- Bad
Select * From users
```

### Put each selected column on its own line

When selecting columns, always put each column name on its own line and never on the same line as `select`. For multiple columns, it's easier to read when each column is on its own line. And for single columns, it's easier to add additional columns without any reformatting (which you would have to do if the single column name was on the same line as the `select`).

```sql
-- Good
SELECT
    id
FROM users 

-- Good
SELECT 
    id,
    email
FROM users 

-- Bad
SELECT id
FROM users 

-- Bad
SELECT id, email
FROM users 
```

## select *

When selecting `*` it's fine to include the `*` next to the `select` and also fine to include the `from` on the same line, assuming no additional complexity like `where` conditions:

```sql
-- Good
SELECT * FROM users 

-- Good too
SELECT *
FROM users

-- Bad
select * from users where email = 'name@example.com'
```

## Indenting where conditions

Similarly, conditions should always be spread across multiple lines to maximize readability and make them easier to add to. Operators should be placed at the end of each line:

```sql
-- Good
SELECT *
FROM users
WHERE
    email = 'example@domain.com'

-- Good
SELECT *
FROM users
WHERE 
    email LIKE '%@domain.com' AND 
    created_at >= '2021-10-08'

-- Bad
select *
from users
where email = 'example@domain.com'

-- Bad
select *
from users
where 
    email like '%@domain.com' and created_at >= '2021-10-08'

-- Bad
select *
from users
where 
    email like '%@domain.com' 
    and created_at >= '2021-10-08'
```

### Left align SQL keywords

Some IDEs have the ability to automatically format SQL so that the spaces after the SQL keywords are vertically aligned. This is cumbersome to do by hand (and in my opinion harder to read anyway) so I recommend just left aligning all of the keywords:

```sql
-- Good
SELECT 
    id,
    email
FROM users
WHERE 
    email like '%@gmail.com'

-- Bad
select id, email
  from users
 where email like '%@gmail.com'
```

### Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
SELECT *
FROM users
WHERE 
    email = 'example@domain.com'

-- Bad
select *
from users
where 
    email = "example@domain.com"
```

If your SQL dialect supports double quoted strings and you prefer them, just make sure to be consistent and not switch between single and double quotes.



### Commas should be at the the end of lines

```sql
-- Good
SELECT
    id,
    email
FROM users

-- Bad
select
    id
    , email
from users
```

While the commas-first style does have some practical advantages (it's easier to spot missing commas and results in cleaner diffs), I'm just not a huge fan of how they look so prefer commas-last.

### Avoid spaces inside of parenthesis

```sql
-- Good
SELECT *
FROM users
WHERE 
    id in (1, 2)

-- Bad
select *
from users
where 
    id in ( 1, 2 )
```

### Break long lists of `in` values into multiple indented lines

```sql
-- Good
SELECT *
FROM users
WHERE
    email IN (
        'user-1@example.com',
        'user-2@example.com',
        'user-3@example.com',
        'user-4@example.com'
    )
```

### Table names should be a plural snake case of the noun

```sql
-- Good
SELECT * 
FROM users

-- Good
SELECT * 
FROM visit_logs

-- Bad
select * 
from user

-- Bad
select * 
from visitLog
```

### Column names should be snake_case

```sql
-- Good
SELECT
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
FROM users

-- Bad
select
    id,
    email,
    timestamp_trunc(created_at, month) as SignupMonth
from users
```

### Column name conventions

* Boolean fields should be prefixed with `is_`, `has_`, or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
* Date-only fields should be suffixed with `_date`. For example, `report_date`.
* Date+time fields should be suffixed with `_at`. For example, `created_at`, `posted_at`, etc.

### Column order conventions

Put the primary key first, followed by foreign keys, then by all other columns. If the table has any system columns (`created_at`, `updated_at`, `is_deleted`, etc.), put those last.

```sql
-- Good
SELECT
    id,
    name,
    created_at
FROM users

-- Bad
select
    created_at,
    name,
    id,
from users
```

### Include `inner` for inner joins

Better to be explicit so that the join type is crystal clear:

```sql
-- Good
SELECT
    users.email,
    SUM(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id

-- Bad
select
    users.email,
    sum(charges.amount) as total_revenue
from users
join charges on users.id = charges.user_id
```

### For join conditions, put the table that was referenced first immediately after the `on`

By doing it this way it makes it easier to determine if your join is going to cause the results to fan out:

```sql
-- Good
SELECT
    ...
FROM users
LEFT JOIN charges ON users.id = charges.user_id
-- primary_key = foreign_key --> one-to-many --> fanout
  
select
    ...
from charges
left join users on charges.user_id = users.id
-- foreign_key = primary_key --> many-to-one --> no fanout

-- Bad
select
    ...
from users
left join charges on charges.user_id = users.id
```

### Single join conditions should be on the same line as the join

```sql
-- Good
SELECT
    users.email,
    SUN(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id
GROUP BY email

-- Bad
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges
on users.id = charges.user_id
group by email
```

When you have mutliple join conditions, place each one on their own indented line:

```sql
-- Good
SELECT
    users.email,
    SUM(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON 
    users.id = charges.user_id and
    refunded = false
GROUP BY email
```

### Avoid aliasing table names most of the time

It can be tempting to abbreviate table names like `users` to `u` and `charges` to `c`, but it winds up making the SQL less readable:

```sql
-- Good
SELECT
    users.email,
    SUM(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id

-- Bad
select
    u.email,
    sum(c.amount) as total_revenue
from users u
inner join charges c on u.id = c.user_id
```

Most of the time you'll want to type out the full table name.

There are two exceptions:

If you you need to join to a table more than once in the same query and need to distinguish each version of it, aliases are necessary.

Also, if you're working with long or ambiguous table names, it can be useful to alias them (but still use meaningful names):

```sql
-- Good: Meaningful table aliases
SELECT
  companies.com_name,
  beacons.created_at
FROM stg_mysql_helpscout__helpscout_companies companies
INNER JOIN stg_mysql_helpscout__helpscout_beacons_v2 beacons ON companies.com_id = beacons.com_id

-- OK: No table aliases
SELECT
  stg_mysql_helpscout__helpscout_companies.com_name,
  stg_mysql_helpscout__helpscout_beacons_v2.created_at
FROM stg_mysql_helpscout__helpscout_companies
INNER JOIN stg_mysql_helpscout__helpscout_beacons_v2 on stg_mysql_helpscout__helpscout_companies.com_id = stg_mysql_helpscout__helpscout_beacons_v2.com_id

-- Bad: Unclear table aliases
select
  c.com_name,
  b.created_at
from stg_mysql_helpscout__helpscout_companies c
inner join stg_mysql_helpscout__helpscout_beacons_v2 b on c.com_id = b.com_id
```

### Include the table when there is a join, but omit it otherwise

When there are no join involved, there's no ambiguity around which table the columns came from so you can leave the table name out:

```sql
-- Good
SELECT
    id,
    name
FROM companies

-- Bad
select
    companies.id,
    companies.name
from companies
```

But when there are joins involved, it's better to be explicit so it's clear where the columns originated:

```sql
-- Good
SELECT
    users.email,
    sum(charges.amount) as total_revenue
FROM users
INNER JOIN charges on users.id = charges.user_id

-- Bad
select
    email,
    sum(amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

```

### Always rename aggregates and function-wrapped arguments

```sql
-- Good
SELECT 
    COUNT(*) AS total_users
FROM users

-- Bad
select 
    count(*)
from users

-- Good
SELECT 
    timestamp_millis(property_beacon_interest) AS expressed_interest_at
FROM hubspot.contact
WHERE 
    property_beacon_interest IS NOT NULL

-- Bad
select
    timestamp_millis(property_beacon_interest)
from hubspot.contact
where
    property_beacon_interest is not null
```

### Be explicit in boolean conditions

```sql
-- Good
SELECT * 
FROM customers 
WHERE 
    is_cancelled = TRUE

-- Good
SELECT * 
FROM customers 
WHERE 
    is_cancelled = FALSE

-- Bad
select * 
from customers 
where 
    is_cancelled

-- Bad
select * 
from customers 
where 
    not is_cancelled
```

### Use `as` to alias column names

```sql
-- Good
SELECT
    id,
    email,
    timestamp_trunc(created_at, month) AS signup_month
FROM users

-- Bad
select
    id,
    email,
    timestamp_trunc(created_at, month) signup_month
from users
```

### Group using column names or numbers, but not both

I prefer grouping by name, but grouping by numbers is [also fine](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

```sql
-- Good
SELECT 
    user_id, 
    count(*) as total_charges
FROM charges
GROUP BY user_id

-- Good
SELECT 
    user_id, 
    count(*) as total_charges
FROM charges
GROUP BY 1

-- Bad
select
    timestamp_trunc(created_at, month) as signup_month,
    vertical,
    count(*) as users_count
from users
group by 1, vertical
```

### Take advantage of lateral column aliasing when grouping by name

```sql
-- Good
SELECT
  timestamp_trunc(com_created_at, year) AS signup_year,
  COUNT(*) as total_companies
FROM companies
GROUP BY signup_year

-- Bad
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by timestamp_trunc(com_created_at, year)
```

### Grouping columns should go first

```sql
-- Good
SELECT
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
FROM companies
GROUP BY signup_year

-- Bad
select
  count(*) as total_companies,
  timestamp_trunc(com_created_at, year) as signup_year
from mysql_helpscout.helpscout_companies
group by signup_year
```

### Aligning case/when statements

Each `when` should be on its own line (nothing on the `case` line) and should be indented one level deeper than the `case` line. The `then` can be on the same line or on its own line below it, just aim to be consistent.

```sql
-- Good
SELECT
    CASE
        WHEN event_name = 'viewed_homepage' THEN 'Homepage'
        WHEN event_name = 'viewed_editor' THEN 'Editor'
        ELSE 'Other'
    END AS page_name
FROM events

-- Good too
SELECT
    CASE
        WHEN event_name = 'viewed_homepage'
            THEN 'Homepage'
        WHEN event_name = 'viewed_editor'
            THEN 'Editor'
        ELSE 'Other'            
    END AS page_name
FROM events

-- Bad 
select
    case when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'        
    end as page_name
from events
```

### Use CTEs, not subqueries

Avoid subqueries; CTEs will make your queries easier to read and reason about.

When using CTEs, pad the query with new lines. 

If you use any CTEs, always `select *` from the last CTE at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

Closing CTE parentheses should use the same indentation level as `with` and the CTE names.

```sql
-- Good
with ordered_details AS (

    SELECT
        user_id,
        name,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY date_updated DESC) AS details_rank
    FROM billingdaddy.billing_stored_details

),

first_updates AS (

    SELECT
        user_id, 
        name
    FROM ordered_details
    WHERE 
        details_rank = 1

)

SELECT * FROM first_updates

-- Bad
select 
    user_id, 
    name
from (
    select
        user_id,
        name,
        row_number() over (partition by user_id order by date_updated desc) as details_rank
    from billingdaddy.billing_stored_details
) ranked
where 
    details_rank = 1
```

### Use meaningful CTE names

```sql
-- Good
with ordered_details AS (

-- Bad
with d1 as (
```

### Window functions

Leave it all on its own line:

```sql
-- Good
SELECT
    user_id,
    name,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY date_updated DESC) AS details_rank
FROM billingdaddy.billing_stored_details

```

## Credits

This style guide was inspired in part by:

* [Fishtown Analytics' dbt Style Guide](https://github.com/fishtown-analytics/corp/blob/master/dbt_coding_conventions.md#sql-style-guide)
* [KickStarter's SQL Style Guide](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
* [GitLab's SQL Style Guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)
* [Matt Mazur](https://mattmazur.com/)


