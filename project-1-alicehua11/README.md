# Summary:
- This is a README that contains **Part 1** and **Part 2** of the **Project 1: Query Project**
- **Part 3** is contained in the **Project_1.jpynb** 

# Project 1: Query Project

- In the Query Project, you will get practice with SQL while learning about
  Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven
  questions using public datasets housed in GCP. To give you experience with
  different ways to use those datasets, you will use the web UI (BiqQuery) and
  the command-line tools, and work with them in Jupyter Notebooks.

#### Problem Statement

- You're a data scientist at Lyft Bay Wheels (https://www.lyft.com/bikes/bay-wheels), formerly known as Ford GoBike, the
  company running Bay Area Bikeshare. You are trying to increase ridership, and
  you want to offer deals through the mobile app to do so. 
  
- What deals do you offer though? Currently, your company has several options which can change over time.  Please visit the website to see the current offers and other marketing information. Frequent offers include: 
  * Single Ride 
  * Monthly Membership
  * Annual Membership
  * Bike Share for All
  * Access Pass
  * Corporate Membership
  * etc.

#### Questions: Answered in Part 3
1. What are the 5 most popular trips that you would call "commuter trips"? 
  
2. What are your recommendations for offers (justify based on your findings)?
---

## Part 1 - Querying Data with BigQuery
- What's the size of this dataset? (i.e., how many trips)

```sql
SELECT COUNT(*)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Answer:
There are 983,648 trips total for the trips from 2013 to 2016.

- What is the earliest start date and time and latest end date and time for a trip?

```sql
SELECT min(start_date) as earliest , max(end_date) as latest
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Answer: The earliest start date and time is August 29, 2013 at  09:08:00 UTC. The latest start date and time is August 31, 2016 at 23:48:00 UTC.

- How many bikes are there?

```sql
SELECT COUNT(DISTINCT bike_number)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Answer: There are 700 bikes in total.

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.  These questions MUST be different than any of the questions and queries you ran above.

- Question 1: Which days of the week have the longest trips and how long? 
  * Answer: Weekend has the longest trips. The top 3 longest durations per trip is 4797, 594 and 515 hours.
  
|trip_id | dow_str | dow_weekday | start_hour_str | duration_hours_rounded |
| -------|---------| ------------| -------------- | -----------------------|
|568474 | Saturday | Weekend     | Evening        | 4797 |
|825850 | Sunday   | Weekend     | Evening        | 594  |
|750192 | Saturday | Weekend     | Morning        | 515  |
|841176 | Friday   | Weekday     | Mid Morning    | 315  |
|111309 | Saturday | Weekend     | Mid Day        | 201  |
|522337 | Thursday | Weekday     | Morning        | 200  |
|323594 | Friday   | Weekday     | Early Afternoon| 199  |

  * SQL query:
```sql
SELECT trip_id, dow_str, dow_weekday, start_hour_str, duration_hours_rounded
FROM (
    SELECT id AS trip_id, total_duration, dow_str, dow_weekday, start_hour_str
    FROM 
        (SELECT trip_id AS id, max(duration_sec) as total_duration
        FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
        GROUP BY trip_id
        ORDER BY total_duration DESC
        LIMIT 7) longest_sec
INNER JOIN `teak-radio-287600.bike_trip_data.dayofweek` dayofweek 
ON longest_sec.id = dayofweek.id2) days
INNER JOIN (
    SELECT trip_id AS id, duration_sec, 
    CAST(ROUND(duration_sec / 60.0) AS INT64) AS duration_minutes,
    CAST(ROUND(duration_sec / 3600.0) AS INT64) AS duration_hours_rounded,
    ROUND(duration_sec / 3600.0, 1) AS duration_hours_tenths
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`) time
ON days.trip_id = time.id
ORDER BY duration_hours_rounded DESC
```

- Question 2: How many subscriber vs. customer are there and how long on average do they tend to ride?
  * Answer: There are 136,809 Customers and  846,839 Subscribers. Customers ride on average 62 mins per trip whereas Subscribers ride on average 10 min per trip.
  * SQL query:
```sql
SELECT subscriber_type, AVG(duration_minutes) as avg_duration_min
FROM `teak-radio-287600.bike_trip_data.trip_dates_time_transformed`
WHERE subscriber_type = "Customer" OR
subscriber_type = "Subscriber"
GROUP BY subscriber_type
```

- Question 3: What are the 10 most popular commuter locations?
  * Answer: Assuming that the commuter trips are trips that customers + subscribers both regularly take in Morning to Mid Morning (6am - 10am) and return in the Afternoon (5pm - 7pm) hours and that they would return to the same station they started with. Also, commuter trips are those made at the same stations from start to end over 50 times, this way we can filter out the noise because trips made less than 50 times are not repeated enough to be regular commuter trips. Here are tthe most popular locations for commuter trips:
1. Harry Bridges Plaza (Ferry Building)
2. Ambarcadero at Sansome
3. Davis at Jackson
4. San Francisco Caltrain (Townsend at 4th)
5. 2nd at Townsend
6. Steuart at Market
7. Powell at Post (Union Square)
8. Powell Street BART
9. San Francisco Caltrain 2 (330 Townsend)
10. Market at 10th

  * SQL query: 
```sql
SELECT start_station_name, end_station_name, COUNT(*) AS number_of_trips
FROM `teak-radio-287600.bike_trip_data.commuter_trip` 
WHERE start_station_name = end_station_name
GROUP BY start_station_name, end_station_name
HAVING number_of_trips > 50
ORDER BY number_of_trips DESC
```

## Part 2 - Querying data from the BigQuery CLI 

- Use BQ from the Linux command line:

  * General query structure

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

### Queries

1. Rerun the first 3 queries from Part 1 using bq command line tool (Paste your bq
   queries and results here, using properly formatted markdown):

  * What's the size of this dataset? (i.e., how many trips)
```
bq query --use_legacy_sql=false
    'SELECT count(*)
    FROM 
        `bigquery-public-data.san_francisco.bikeshare_trips`'
```
Answer: 
```
+-------------+
| total_trips |
+-------------+
|      983648 |
+-------------+
```

  * What is the earliest start time and latest end time for a trip?
```
 bq query --use_legacy_sql=false 
 'SELECT min(start_date) AS earliest , max(end_date) AS latest 
 FROM 
     `bigquery-public-data.san_francisco.bikeshare_trips`'
```
Answer:
```
+---------------------+---------------------+
|      earliest       |       latest        |
+---------------------+---------------------+
| 2013-08-29 09:08:00 | 2016-08-31 23:48:00 |
+---------------------+---------------------+
```

  * How many bikes are there?
```
bq query --use_legacy_sql=false 
'SELECT COUNT(DISTINCT bike_number) AS total_bikes  
FROM 
    `bigquery-public-data.san_francisco.bikeshare_trips
```
Answer:
```
+-------------+
| total_bikes |
+-------------+
|         700 |
+-------------+
```

2. New Query (Run using bq and paste your SQL query and answer the question in a sentence, using properly formatted markdown):

  * How many trips are in the morning vs in the afternoon?
```
bq query --use_legacy_sql=false 
'SELECT COUNT(CASE WHEN start_hour_str = "Morning" OR start_hour_str = "Mid Morning" THEN 1 END) AS morning_trips,
        COUNT(CASE WHEN start_hour_str = "Afternoon" OR start_hour_str = "Early Afternoon" THEN 1 END) AS afternoon_trips
        FROM `teak-radio-287600.bike_trip_data.trips_with_dates`'
```

Answer:
```
+---------------+-----------------+
| morning_trips | afternoon_trips |
+---------------+-----------------+
|        359414 |          426175 |
+---------------+-----------------+
```

### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

### Answers

Answer at least 4 of the questions you identified above You can use either
BigQuery or the bq command line tool.  Paste your questions, queries and
answers below.

- Question 1: How many stations are there for the busiest cities (defined by having high numbers of trips)?
  * Answer: There are 5 cities with the busiest cities (defined by having high number of trips in desceding order) and their numbers of stations in descending order in the table below. It looks like San Francisco is the busiest with 37 stations followed by San Jose, Mountain View and the not as busy cities have less stations such as Palo Alto and Redwood City.

| city | num_stations |
| -----|--------------|
| San Francisco | 37 |
| San Jose | 18 |
| Redwood City | 7 |
| Mountain View | 7 |
| Palo Alto | 5 |

  * SQL query: 
```sql
SELECT city, COUNT(*) as num_stations,
FROM (
  SELECT station_id, landmark AS city, COUNT(*) AS num_trips,
  FROM `teak-radio-287600.bike_trip_data.trips_with_stations` 
  GROUP BY landmark, station_id
  ORDER BY num_trips DESC) 
GROUP BY city
ORDER BY num_stations DESC
```

- Question 2: Based on Part 1 answer, if customers ride 62 minutes on averager, we might have a business opportunity for customers (non-subscribers) to purchase membership if they ride more than 30 min on average. Find out how many customers (non-subscribers) who ride more than 30 min each year from 2013 to 2016?
  * Answer: The number of customers (non-subscribers) who ride more than 30 min per year from 2013 to 2016 is listed in the table below. They can be converted to subscribers to save cost. In term of pricing, they are paying more out of pocket as compared to having a memberships since options such as single ride and access pass give only 30 min of riding for $2 and $3 for each additional 15 min. If they are subscribers, their allowed time would increase to 45 min, they would save $3*15min per ride that is longer than 30 min. 
  
| year | num_cus |
| -----|---------|
| 2013 | 6658 |
| 2014 | 14826 |
| 2015 | 12679 |
| 2016 | 7223 |


  * SQL query:
```sql
SELECT year, COUNT(*) AS num_cus
FROM (
  SELECT subscriber_type, duration_minutes, year 
  FROM (
    SELECT start_date, subscriber_type, duration_minutes,
      EXTRACT(YEAR FROM start_date) AS year
    FROM `teak-radio-287600.bike_trip_data.trip_dates_time_transformed`)
  WHERE duration_minutes > 30 AND subscriber_type = 'Customer')
GROUP BY year
ORDER BY year 
```

- Question 3: What are some of the most popular trips that most subscribers embarked on?
  * Answer: The most popular trips for subscribers are from Morning - Afternoon on weekdays from these stations:
  
  1. Harry Bridges Plaza (Ferry Building) to 2nd at Townsend
  2. San Francisco Caltrain (Townsend at 4th) to Temporary Transbay Terminal (howeard at Beale)
  3. Embarcadero at Sansome to Steuart at Market
  4. 2nd at South Park to Market at Sansome
  5. San Francisco Caltrain (Townsend at 4th) to Embarcadero at Folsom
  
  These seem to be regular commuting trips for work for these subscribers.

  * SQL query:
```sql
SELECT start_station_name, end_station_name, start_hour_str, dow_str, dow_weekday, COUNT(*) AS number_of_trips
FROM `teak-radio-287600.bike_trip_data.subscriber_table`
GROUP BY start_station_name, end_station_name, start_hour_str, dow_str, dow_weekday
HAVING number_of_trips > 100
ORDER BY number_of_trips DESC
```
  
- Question 4: With the most trips in the busiest cities, how many bikes do they have per city in total over the years?
  * Answer: In the order of busiest cities, the total number of bikes increased over time from 2013 - 2016 for San Francisco and San Jose but not for Moutain View ad Palo Alto. Though interestingly, Redwood City with the lowest number of trips had an increase in total bikes in 2016. 
  
| city | year | total_bike_per_yr
|------|------|------------------|
| San Francisco| 2013 | 638 |
| San Francisco| 2014 | 667 |
| San Francisco| 2015| 665 |
| San Francisco| 2016| 735 |
| San Jose| 2013| 249 |
| San Jose| 2014| 264 |
| San Jose| 2015| 264 |
| San Jose| 2016| 302 |
| Mountain View| 2013| 117 |
| Mountain View| 2014| 117 |
| Mountain View| 2015| 117 |
| Mountain View| 2016| 117 |
| Palo Alto| 2013| 75 |
| Palo Alto| 2014| 75 |
| Palo Alto| 2015| 75 |
| Palo Alto| 2016| 75 |
| Redwood City| 2013| 102 |
| Redwood City| 2014| 117 |
| Redwood City| 2015| 117 |
| Redwood City| 2016| 161 |


  * SQL query:
```sql
SELECT city, year, SUM(bike_total) as total_bike_per_yr
FROM (
  SELECT station_id, city, year, MAX(total_bikes) as bike_total
  FROM `teak-radio-287600.bike_trip_data.bikes_per_city_year` 
  GROUP BY station_id, city, year
  )
GROUP BY year, city 
ORDER BY ( CASE city
              WHEN 'San Francisco' THEN 1
              WHEN 'San Jose' THEN 2
              WHEN 'Mountain View' THEN 3
              WHEN 'Palo Alto' THEN 4
              WHEN 'Redwood City' THEN 5
              ELSE 0
            END)
```

Continue to **Part 3** in the Jupyter Notebook.
---


