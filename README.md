CREATE TABLE "Nexa_Sat".nexa_sat (
Customer_ID VARCHAR(50),
gender VARCHAR(10),
Partner VARCHAR(3),
Dependents VARCHAR(3),
Senior_Citizen INT,
Call_Duration FLOAT,
Data_Usage FLOAT,
Plan_Type VARCHAR(20),
Plan_Level VARCHAR(20),
Monthly_Bill_Amount FLOAT,
Tenure_Months INT,
Multiple_Lines VARCHAR(3),
Tech_Support VARCHAR(3),
Churn INT);
---import data

SELECT *
FROM "Nexa_Sat".nexa_sat

SET search_path TO "Nexa_Sat"

SELECT current_schema()

select *
from nexa_sat

---checking for duplicate
SELECT
customer_id, gender, partner, dependents,
senior_citizen, call_duration, data_usage,
plan_type, plan_level, monthly_bill_amount,
tenure_months, multiple_lines, tech_support,
churn
FROM
nexa_sat
GROUP BY
customer_id, gender, partner, dependents,
senior_citizen, call_duration, data_usage,
plan_type, plan_level, monthly_bill_amount,
tenure_months, multiple_lines, tech_support,
churn
HAVING COUNT(*) > 1

---checking for null values (blank cells)
SELECT *
FROM nexa_sat
WHERE customer_id IS NULL
OR gender IS NULL
OR partner IS NULL
OR dependents IS NULL
OR senior_citizen IS NULL
OR call_duration IS NULL
OR data_usage IS NULL
OR plan_type IS NULL
OR plan_level IS NULL
OR monthly_bill_amount IS NULL
OR tenure_months IS NULL
OR multiple_lines IS NULL
OR tech_support IS NULL
OR churn IS NULL

---identifying total users
select count (customer_id) as total_user
from nexa_sat

---identify toal users by level
select plan_type, count(customer_id) as total_user
from nexa_sat
group by plan_type

--total revenue
select sum(monthly_bill_amount::numeric) as total_revenue
from nexa_sat

----total revenue by level
select plan_level, sum(monthly_bill_amount::numeric) as total_revenue
from nexa_sat
group by plan_level

---Churn count
select count(customer_id), churn
from nexa_sat
where churn = 0 or churn = 1
group by 2

---churn=0 count by plan_level and plan_type
SELECT
plan_level,
plan_type,
COUNT(*) AS total_customers,
SUM(churn) AS churn_count
FROM
nexa_sat
GROUP BY 1, 2
ORDER BY 1

--tenure by level
select plan_level, max(tenure_months::numeric), round(avg(tenure_months),2)
from nexa_sat
group by 1

-----market segment( creating table for exixting_users)
create table existing_users as
select *
from nexa_sat
where churn = 0

----view
select *
from existing_users

--identifying the Average Revenue Per User
select round(avg(monthly_bill_amount::numeric),2) as ARPU
from existing_users

---caluculating customer lifetime value (CLV) and add column
alter table existing_users
add column clv float

update existing_users
set clv = monthly_bill_amount * tenure_months

----view the clv column
select *
from existing_users

---creating clv score column
alter table existing_users
add column clv_score numeric (10,2)

---assigning weight to get clv score
update existing_users
set clv_score = (0.4 * monthly_bill_amount) + (0.3 * tenure_months) + (0.1 * call_duration) + (0.1 * data_usage) + (0.1 * case when plan_level = 'premium' then 1 else 0 end)

---view the clv_score column
select *
from existing_users

---group into segment based on score
alter table existing_users
add column clv_segment varchar

update existing_users
set clv_segment =
case
when clv_score > 85 then 'high_risk'
when clv_score >= 50 then 'moderate_risk'
when clv_score >= 25 then 'low_risk'
else 'churn risk' end

----view
select customer_id, clv_segment
from existing_users

-----clv_segment count
select clv_segment, count(*) as segment_count
from existing_users
group by 1
order by 2

----or
select clv_segment, count(customer_id)
from existing_users
group by 1
order by 2

---average bills spend and average tenure spent by segment
select clv_segment, round(avg(monthly_bill_amount::int),2) as avg_month_bill, round(avg(tenure_months::int),2) as avg_tenure
from existing_users
group by 1

-----revenue per segment
select clv_segment, count(customer_id), round(sum(clv::numeric),2)
from existing_users
group by 1
