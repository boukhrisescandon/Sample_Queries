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

```python

import pandas as pd
reviews = airbnb_reviews[airbnb_reviews['from_type']=='guest']
reviews = reviews[['from_user', 'review_score']]
reviews.columns = ['guest_id', 'review_score']
reviews.head()

guests = reviews.merge(airbnb_guests, on = 'guest_id')

guests = guests[['gender', 'review_score']]

ans = guests.groupby(by = 'gender', as_index = False).mean()
ans.head()

ans = ans[ans['review_score'] == ans['review_score'].max()]
ans.head()


```

```sql

select guest.gender, avg(review.review_score) from airbnb_reviews as review inner join airbnb_guests as guest on review.from_user = guest.guest_id where review.from_type = 'guest' group by guest.gender limit 1;

```


## Records with '?'

Find records with the value '?' in the stars column.

```python

import pandas as pd

yelp_reviews = yelp_reviews[yelp_reviews['stars'] == '?']
yelp_reviews.head()

```

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

```python

import pandas as pd

emails = employee[['emails']]
emails['counter'] = 1
emails.head()

counts = emails.groupby(by = ['email'], as_index=False).count()
counts.head()

ans = counts[counts['counter'] > 1]
ans = ans[['email']]
ans.head()

```

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


## Recommendation System

You are given a list of Facebook friends and the list of Facebook pages that users follow. Your task is to create a new recommendation system for Facbook. For each Facebook user, find pages that this user doesn't follow but at least one of their friends does. 

```sql

select distinct f.user_id, p.page_id from users_friends as f join users_pages as p on f.friend_id=p.user_id where (f.user_id, p.page_id) not in (select user_id, page_id from users_pages) order by f.user_id;

```


## Salaries Differences

Write a query that calculates the difference between the highest salaries found in the marketing and engineering departments. 

```sql

select abs((select max(e.salary) from db_employee as e inner join bd_dept as d on e.department_id = d.id where d.department='engineering') - (select max(e.salary from db_employee as e inner join db_dept as d on e.department_id = d.id where d.department = 'marketing')) as salary_difference;
           
```


## Percentage of Total Spend

Calculate the percentage of the total spend a customer spent on each order.

```sql

with name_cte as 
(select c.id, c.first_name, o.order_details, o.total_order_cost from orders as o inner join customers as c on o.cust_id = c.id order by c.first_name),
tot_cte as 
(select cust_id, sum(total_order_cost) as total_cost from orders group by cust_id)
select name_cte.first_name, name_cte.order_details, (name_cte.total_order_cost/tot_cte.total_cost) as percentage_total_cost from name_cte join tot_cte on name_cte.id = tot_cte.cust_id;

```


## Finding Updated Records

We have a table with employees and their salaries, some records are old and contain outdated salary information. Find the current salary of each employee assuming that salaries increase each year. 

```sql

select id, first_name, last_name, department_id, max(salary) as max from ms_employee_salary group by (id, first_name, last_name, department_id) order by id;

```


## Cities with the Most Expensive Homes

Write a query that identifies cities with higher than average home prices when compared to the national average. 

```python

import pandas as pd
import numpy as np

national_average = np.mean(zillow_transactions['mkt_price'])

cities = zillow_transactions[['city','mkt_price']]

city_averages = cities.groupby(by=['city'], as_index=False).mean()

expensive_cities = city_averages[city_averages['mkt_price'] > national_average]
ans = expensive_cities('city')
ans.head()

```

```sql

--first option using common table expressions
with cte_city as 
(select city, avg(mkt_price) as city_average from zillow_transactions group by city),
cte_nat as
(select avg(mkt_price) as national_average from zillow_transactions)
select cte_city.city from cte_city where cte_city.city_average > (select national_average from cte_nat);

--second option using having
select city from zillow_transactions group by city having avg(mkt_price) > (select avg(mkt_price) rom zillow_transactions);

```


## 3 Bed Minimum

Find the average number of beds in each neighborhood that has at least 3 beds in total.

```python
import pandas as pd 

neighbor = airbnb_search_details[['neighbourhood', 'beds']]
avg_beds = neighbor.groupby(['neighbourhood'], as_index=False).mean()
sum_beds = neighbor.groupby(['neighbourhood'], as_index=False).sum()
big_beds = sum_beds[sum_beds['beds']>=3]

ans = avg_beds.merge(big_beds, on='neighbourhood')
ans = ans[['neighbourhood', 'beds_x']]
ans.columns = ['neighbourhood', 'n_beds_avg']
ans.sort_values(by = 'n_beds_avg', ascending=False)

```

```sql

select neighbourhood, avg(beds) as n_beds_avg from airbnb_search_details group by neighbourhood having sum(beds)>=3 order by n_beds_avg desc;

```


## Most Profitable Companies 

Find the three most profitable companies in the entire world.

```sql

select company, profits from forbes_global_2010_2014 order by profits desc limit 3;

```


## Total AdWords Earnings

Find the total AdWords earnings for each business type. 

```python

import pandas as pd

buz = google_adwords_earnings[['business_type', 'adwords_earnings']]
buz_avg = buz.groupby(by = ['business_type'], as_index=False).sum()
buz_avg.head()

```


## Find details of workers excluding those with the first name 'Vipul' or 'Satish'

Find details of workers excluding those with the first name 'Vipul' or 'Satish'.

```python

import pandas as pd

ans = worker[(worker['first_name'] != 'Vipul')]
ans = ans[(ans['first_name'] != 'Satish')]
ans

```

## Number of Shipments per Month

Write a query that will calculate the number of shipments per month.

```python

import pandas as pd
import datetime as dt

orders = amazon_shipment[['shipment_date']]
orders['counter'] = 1
orders['year_month'] = orders['shipment_date'].dt.strftime('%Y-%m')
orders.head()

orders = orders[['year_month', 'counter']]

agg = orders.groupby(by='year_month', as_index=False).count()
agg.head()

```

```sql

select to_char(shipment_date, 'YYYY-MM') as date, count(shipment_id) as cnt from amazon_shipment group by to_char(shipment_date, 'YYYY-MM');

```


## Customer Average Orders

How many customers placed an order and what is the average order amount?

```python

import pandas as pd
import numpy as np

avg = postmates_orders['amount'].mean()
avg

cnt = postmates_orders['customer_id'].unique().shape[0]
cnt

result = postmates_orders.agg({'customer_id':'nunique', 'amount':'mean'}).reset_index()
result

```

```sql

select (select count(distinct customer_id) as count from postmates_orders), (select avg(amount) as avg from postmates_orders) from postmates_orders limit 1;

```


## User Scroll up Events

Find all users who have performed at least one scroll_up event.

```python

import pandas as pd

users = facebook_web_log[facebook_web_log['action'] == 'scroll_up']
users.head()

ans = pd.DataFrame(users['user_id'].unique())
ans.columsn = ['user_id']

ans.head()

```

```sql

select distinct(user_id) from facebook_web_log where action = 'scroll_up';

```









