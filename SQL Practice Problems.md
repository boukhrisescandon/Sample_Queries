# SQL Practice Problems

*StrataScratch*



## Bikes Last Used

Find the last time each bike was in use.

```sql

select bike_number, max(end_time) as last_used from dc_bikeshare_q1_2012
group by bike_number
order by last_used desc;

```


## Number of Acquisitions

Find the number of acquisitions that occurred in each quarter of each year.

```sql

select distinct acquired_quarter, count(acquired_quarter) as cnt_acq from crunchbase_acquisitions
group by acquired_quarter
order by cnt_acq desc;

```


## Number of Acquisitions

Find the number of acquisitions that occurred in each quarter of each year.

```sql

select distinct acquired_quarter, count(acquired_quarter) as cnt_acq from cruchbase_acquisitions
group by acquired_quarter order by cnt_acq desc;

```


## Gender with Generous Reviews

Write a query to find which gender gives a higher average review score when writing reviews as guests.

```sql

select guest.gender, avg(review.review_score) from airbnb_reviews as review inner join airbnb_guests as guest on review.from_user = guest.guest_id where review.from_type = 'guest' group by guest.gender limit 1;

```


## Records with '?'

Find records with the value '?' in the stars column.

```sql

select * from yelp_reviews where stars = '?';

```


## Patrons who Revewed Books

Find the number of patrons who renewed books at least once but less than 10 times in April 2015.

```sql

select count(total_renewals) from library_usage where total_renewals between 1 and 10 and circulation_active_month='April' and circulation_active_year=2015;

```


## Duplicate Emails

Find all emails with duplicates

```sql

select email from (select email, count(email) as cnt from employee group by email order by cnt desc) a limit 1;

-- alternative solution
select email from employee group by email having count(id)>=2;

```


## Find departments with less than 5 employees

Find departments with less than 5 employees. 

```sql

select department, count(worker_id) as num_of_workers from worker group by department having count(worker_id)<5;

```


## Time from the 10th Runner

In a marathon, time is counted from the moment of the formal start of the race while net time is counted from the moment a runner crosses a starting line. How much net time separates Chris Doe from the 10th best net time (in ascending order)? Avoid gaps in ranking calculations.

```sql

with cte1 as 
(select net_time from marathon_male where person_name = 'Chris Doe'),
cte2 as
(select * from (select net_time, dense_rank() over (order by net_time) as d_rank from marathon_male) a where d_rank=10)
select distinct abs(cte1.net_time - cte2.net_time) from cte1, cte2;

```


## Responsible for Most Customers

Write a query to get the Employees who are responsible for the maximum number of Customers.

```sql

select e.empl_id, count(c.cust_id) as n_customers from map_employee_territory as e inner join map_customer_territory as c on e.territory_id = c.territory_id group by e.empl_id order by count(c.cust_id) desc limit 4;

```


