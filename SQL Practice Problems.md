# SQL Practice Problems
### StrataScratch


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