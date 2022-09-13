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


