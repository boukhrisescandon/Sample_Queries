# SQL Practice Problems

*StrataScratch*



## Bikes Last Used

Find the last time each bike was in use.

```python

import pandas as pd

ans = dc_bikeshare_q1_2012.groupby(by='bike_number', as_index=False)['end_time'].max()
ans = ans.sort_values(by='end_time', ascending=False)
ans

```

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

```sql
--option 2:

with chris as 
(select net_time from marathon_male where person_name='Chris Doe'),
ten as
(select distinct net_time from (select net_time, dense_rank() over (order by net_time asc) as ranking from marathon_male) a where ranking = 10)
select abs(chris.net_time - ten.net_time) as diff from chris as chris cross join ten as ten;

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


## Paid Users in April 2020

How many paid users had any calls in April 2020?

```python

import pandas as pd
import datetime as dt

merge = rc_calls.merge(rc_users, on='user_id')
merge.head()

free = merge[merge['status']=='free']
free['year-month'] = free['date'].dt.strftime('%Y-%m')

ans = free[free['year-month'] == '2020-04']

ans.shape[1]

```

```sql

select count(distinct calls.user_id) as count from rc_calls as calls inner join rc_users as users on calls.user_id = users.user_id where users.status='free' and to_char(calls.date, 'YYYY-MM') = '2020-04';

```


## Users with Two Statuses

Find users who are both a viewer and streamer.

```python

import pandas as pd

streamers = twitch_sessions[twitch_sessions['session_type'] == 'streamer']
viewers = twitch_sessions[twitch_sessions['session_type'] == 'viewer']

merge = streamers.merge(viewers, on='user_id')
ans = merge['user_id'].unique()
ans

```

```sql

with viewer as
(select user_id from twitch_sessions where session_type='viewer'),
streamer as
(select user_id from twitch_sessions where session_type='streamer')
select distinct(viewer.user_id) from viewer inner join streamer on viewer.user_id = streamer.user_id;

```


## Homework Results

Given the homework results of a group of students, calculate the average grade and the completion rate of each student.

```python

# Import your libraries
import pandas as pd
import numpy as np

# Start writing code
allstate_homework.head()

allstate_homework['complete'] = np.where(allstate_homework['grade']>0, 1, 0)
allstate_homework.head()

avg_grade = allstate_homework.groupby(by='student_id', as_index=False).mean()
avg_grade['complete'] = avg_grade['complete']*100

merged = avg_grade.merge(allstate_students, on='student_id')
merged.head()

ans = merged[['student_firstname', 'grade', 'complete']]
ans.head()

ans.columns = ['student_firstname', 'avg_grade', 'completion_rate']
ans.head()

```

```sql

-- here we get the average grade
select student.student_firstname, avg(homework.grade) over (partition by homework.student_id) as avg_grade from allstate_students as student inner join allstate_homework as homework on student.student_id = homework.student_id;

-- indication that the homework was completed
with cte1 as
(select student_id, 
case when grade>0 then 1
else 0
end as complete from allstate_homework),
cte2 as
(select student.student_id, student.student_firstname, avg(homework.grade) over (partition by homework.student_id) as avg_grade from allstate_students as student inner join allstate_homework as homework on student.student_id = homework.student_id)
select distinct cte2.student_firstname, cte2.avg_grade, (avg(cte1.complete) over (partition by cte1.student_id))*100 as completion_rate from allstate_homework as homework inner join cte1 on homework.student_id = cte1.student_id inner join cte2 on cte2.student_id = homework.student_id;

-- average completion rate
--select student_id, avg(select student_id, case when grade>0 then 1 else 0 end as complete from allstate_homework) over (partition by student_id) as completion_rate from allstate_homework;

```


## Count the number of user events performed by MacBook Pro Users

Count the number of user events performed by MacBook Pro users.

```python

import pandas as pd

mac = playbook_events[playbook_events['device']=='macbook pro']
mac.head()

ans = mac.groupby(by='event_name', as_index=False).count()
ans.head()

ans = ans[['event_name', 'user_id']]
ans.column = ['event_name', 'event_count']
ans

```

```sql

select event_name, count(*) from playbook_events where device='macbook pro' group by event_name ;

```


## Department Salaries

Find the number of male and female employees per department and also their corresponding total salaries.

```python

import pandas as pd

females = employee[employee['sex']=='F'].groupby(by='department', as_index=False)['id'].count()
females.columns = ['department', 'females']
females

males = employee[employee['sex']=='M'].groupby(by='department', as_index=False)['id'].count()
males.columns = ['department', 'males']
males

fem_sal = employee[employee['sex']=='F'].groupby(by='department', as_index=False)['salary'].sum()
fem_sal.columns = ['department', 'fem_sal']
fem_sal

mal_sal = employee[employee['sex']=='M'].groupby(by='department', as_index=False)['salary'].sum()
mal_sal.columns = ['department', 'mal_sal']
mal_sal

ans = females.merge(males, on='department')
ans = ans.merge(fem_sal, on='department')
ans = ans.merge(mal_sal, on='department')

```

```sql

select * from employee;

-- count females
-- select department, count(id) as females from employee where sex='F' group by department; 

-- count males
-- select department, count(id) as males from employee where sex='M' group by department;

-- average female salary
-- select department, sum(salary) as fem_sal from employee where sex='F' group by department;

-- average male salary
-- select department, sum(salary) as mal_sal from employee where sex='M' group by department;

-- Now put it all together
with females as
(select department, count(id) as females from employee where sex='F' group by department),
female_sal as
(select department, sum(salary) as fem_sal from employee where sex='F' group by department),
males as
(select department, count(id) as males from employee where sex='M' group by department),
male_sal as
(select department, sum(salary) as mal_sal from employee where sex='M' group by department)
select females.department, females.females, female_sal.fem_sal, males.males, male_sal.mal_sal from females inner join males on females.department = males.department inner join female_sal on females.department=female_sal.department inner join male_sal on male_sal.department=females.department;

```


## Active Users per Platform

For each platform, calculate the number of users.

```python

import pandas as pd

ans = user_sessions.groupby(by='platform', as_index=True)['user_id'].nunique().reset_index()
ans

```

```sql

select platform, count(distinct user_id) as n_users from user_sessions group by platform;

```


## Frequent Customers

Find customers who appear in the orders table more than 3 times.

```python

import pandas as pd

ans = orders.groupby(by='cust_id')['id'].count().reset_index()
ans = ans[ans['id']>3]
ans['cust_id']

```

```sql

select cust_id from orders group by cust_id having count(cust_id)>3;

```


## Customers without Orders

Find customers who have never made an order.

```python

import pandas as pd

ordered = orders['cust_id'].unique()
ordered

ans = customers[~customers['id'].isin(ordered)]['first_name']
ans

```

```sql

select distinct first_name from customers where id not in (select cust_id from orders);

```


## Pending Claims

Count how many claims submitted in December 2021 are still pending. A claim is pending when it has neither an acceptance nor rejection date.

```python

import pandas as pd
import datetime as dt

ans = cvs_claims[cvs_claims['date_accepted'].isna() & cvs_claims['date_rejected'].isna()]
ans = ans[ans['date_submitted'].dt.strftime('%Y-%m') == '2021-12']

ans.shape[0]

```

```sql

select count(distinct claim_id) from cvs_claims where to_char(date_submitted, 'YYYY-MM')='2021-12' and date_accepted is NULL and date_rejected is NULL;

```


## Number of Records by Variety

Find the total number of records that belong to each variety in the dataset. 

```python

import pandas as pd

ans = iris.groupby(by='variety', as_index=False).count()
ans = ans[['variety', 'sepal_length']]

```

```sql

select variety, count(variety) from iris group by variety;

```


## Company with Most Desktop Only Users

Write a query that returns the company with the highest number of users that use desktop only.

```sql

with cte as
(select id, user_id, customer_id from fact_events where client_id='desktop' and user_id not in (select user_id from fact_events where client_id<>'desktop'))
select customer_id from cte group by customer_id;
```


## Unique Users per Client per Month

Write a query that returns the number of unique users per client per month.

```sql

select client_id, to_char(time_id, 'MM') as month, count(distinct user_id) from fact_events group by client_id, month;

```


## Most Profitable Companies

Find the three most profitable companies in the world.

```sql

select company, profits from forbes_global_2010_2014 order by profits desc limit 3;

```


## Find User Purchases

Write a query that will identify returning active users.

```sql

select distinct(a.user_id) from amazon_transactions as a join amazon_transactions b on a.user_id=b.user_id where a.created_at - b.created_at between 0 and 7 and a.id != b.id;

```


## The Most Popular Client_Id among Users using Video and Voice Calls

Select the most popular client_id based on a count of the number of users who have at least 50% of their events from the list: 'video call received', 'video call sent', 'voice call received', 'voice call sent'.

```sql

with list as
(select user_id, client_id, count(event_type) as event_in_list from fact_events where event_type in ('video call received', 'video call sent', 'voice call received', 'voice call sent') group by (user_id, client_id)),
total as
(select user_id, client_id, count(event_type) as event_tot from fact_events group by (user_id, client_id))
select list.client_id from list inner join total on list.user_id = total.user_id where (event_in_list/total.event_tot)>=0.5 limit 1;

```


## Average Salaries

Compare each employee's salary with the average salary of the corresponding department.

```sql

select department, first_name, salary, avg(salary) over (partition by department) from employee;

```


## Find the most profitable company in the financial sector

Find the most profitable company in the financial sector. Output the result along with the continent.

```python

import pandas as pd

fin = forbes_global_2010_2014[forbes_global_2010_2014['sector']=='Financials']

prof = fin['profits'].max()

ans = fin[fin['profits']==prof]
ans[['company', 'continent']]

```
```sql

select company, continent from forbes_global_2010_2014 where sector='Financials' and profits = (select max(profits) from forbes_global_2010_2014 where sector='Financials');

```


## Book Sales

Calculate the total revenue made per book. Output the book ID and total sales per book. In case there is a book that has never been sold, include it in your output with a value of 0.

```sql
with cte1 as (
select books.book_id, unit_price, 
case when quantity is Null then 0 else quantity end as quantity
from amazon_books as books 
left join amazon_books_order_details as orders 
on books.book_id = orders.book_id)
select distinct book_id, (unit_price * sum(quantity) over (partition by book_id)) as total_sales from cte1;
```


## Duplicate Training Lessons

Display a list of users who took the same training lessons more than once on the same day. Output their usernames, training IDs, dates and the number of times they took the same lesson.

```sql

with cte as (
select distinct u.u_name, t.training_id, t.training_date, count(t.u_id) over (partition by t.training_date, t.u_id, t.training_id) as count_on_date from training_details as t inner join users_training as u on t.u_id = u.u_id)
select * from cte where count_on_date > 1;

```


## Top 2 Users with Most Calls

Return the top 2 users in each company that called the most. Output the company_id, user_id, and the user's rank. If there are multiple users in the same rank, keep all of them.

```sql

with cte as (
select u.company_id, c.user_id, dense_rank() over (partition by u.company_id order by count(c.user_id) desc) as rank from rc_calls as c inner join rc_users as u on u.user_id = c.user_id group by u.company_id, c.user_id)
select * from cte where rank = 1 or rank = 2;

```


## Bottom 2 Companies by Mobile Usage

Write a query that returns a list of the bottom 2 companies by mobile usage. 

```sql

with cte as (
select customer_id, count(id) as events, dense_rank() over (order by count(id)) from fact_events where client_id='mobile' group by customer_id)
select customer_id, events from cte where dense_rank in (1, 2);

```


## Fastest Hometowns

Find the hometowns with the top 3 average net times. Output the hometowns and their average net time. In case there are ties in net time, return all unique hometowns.

```sql

with cte as
(select hometown, avg(net_time) as avg_net_time, dense_rank() over (order by avg(net_time)) as ranking from marathon_male group by hometown)
select hometown, avg_net_time from cte where ranking <= 3;

```


## Number of shipments per Month

Write a query that will calculate the number of shipments per month. The unique key for one shipment is a combination of shipment_id and sub_id. Output the year_month in format YYYY-MM and the number of shipments in that month.

```sql

select count(distinct concat(shipment_id, sub_id)), to_char(shipment_date, 'YYYY-MM') as year_month from amazon_shipment group by to_char(shipment_date, 'YYYY-MM');

```


## Responsible for Most Customers

Each Employee is assigned one territory and is responsible for the Customers from this territory. There may be multiple employees assigned to the same territory.
Write a query to get the Employees who are responsible for the maximum number of Customers. Output the Employee ID and the number of Customers.

```sql

with cte as
(select e.empl_id, count(c.cust_id) as n_customers, dense_rank() over (order by count(c.cust_id) desc) as rank from map_employee_territory as e inner join map_customer_territory as c on e.territory_id = c.territory_id group by e.empl_id)
select empl_id, n_customers from cte where rank=1;

```


## Low Fat and Recyclable

What percentage of all products are both low fat and recyclable?

```sql
select (select count(*) from facebook_products where is_low_fat='Y' and is_recyclable='Y') / (select count(*) from facebook_products)::float * 100 as percentage;

--select (count(*) / (select count(*) from facebook_products)::float * 100 )as percentage from facebook_products where is_low_fat='Y' and is_recyclable='Y'
```


## Top Streamers

List the top 10 users who accumulated the most sessions where they had more streaming sessions than viewing. Return the user_id, number of streaming sessions, and number of viewing sessions.

```sql
with streamer as
(select user_id, count(session_id) as streaming from twitch_sessions where session_type='streamer' group by user_id), 
viewer as 
(select user_id, count(session_id) as view from twitch_sessions where session_type = 'viewer' group by user_id)
select streamer.user_id, streamer.streaming, viewer.view from streamer inner join viewer on streamer.user_id = viewer.user_id where streamer.streaming > viewer.view limit 10;
```
