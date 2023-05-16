# Cyclistic Case Study SQL Queries

## Distinct values 
```
SELECT DISTINCT rideable_type
FROM
`cyclistic-marketing-project.spring_2022.march-april-may-2022`
```

## The number of rows
```
SELECT COUNT(*) AS number_of_row
FROM
`cyclistic-marketing-project.spring_2022.march-april-may-2022`
```

## Check the null cell for the table
```
DECLARE sql_query STRING;


SET sql_query = (SELECT STRING_AGG(
   CONCAT(
     "SUM(CASE WHEN ", column_name, " IS NULL THEN 1 ELSE 0 END) AS ", column_name, "_null_count"
   ),
   ", "
 )
 FROM `cyclistic-marketing-project.spring_2022.INFORMATION_SCHEMA.COLUMNS`
 WHERE table_name = "march-april-may-2022"
 AND table_catalog = "cyclistic-marketing-project"
 AND table_schema = "spring_2022"
);


SET sql_query = CONCAT("SELECT ", sql_query, " FROM `cyclistic-marketing-project.spring_2022.march-april-may-2022`");


EXECUTE IMMEDIATE sql_query;
```

## Check the nulls for columns
```
SELECT SUM(CASE WHEN ride_length IS NULL THEN 1 ELSE 0 END) AS null_length,
SUM(CASE WHEN ride_id IS NULL THEN 1 ELSE 0 END) AS null_id
from
`cyclistic-marketing-project.spring_2022.march-april-may-2022`
```

## Union tables
```
SELECT ride_id, rideable_type, started_at, ended_at, start_lat, start_lng, end_lat, end_lng, member_casual, ride_length, day_of_week, start_station_name, end_station_name
FROM `cyclistic-marketing-project.autumn_2022.sep-oct-nov-2022`
UNION DISTINCT
SELECT ride_id, rideable_type, started_at, ended_at, start_lat, start_lng, end_lat, end_lng, member_casual, ride_length, day_of_week, start_station_name, end_station_name
FROM `cyclistic-marketing-project.spring_2022.march-april-may-2022`
UNION DISTINCT
SELECT ride_id, rideable_type, started_at, ended_at, start_lat, start_lng, end_lat, end_lng, member_casual, ride_length, day_of_week, start_station_name, end_station_name
FROM `cyclistic-marketing-project.summer_2022.june-july-august-2022`
UNION DISTINCT
SELECT ride_id, rideable_type, started_at, ended_at, start_lat, start_lng, end_lat, end_lng, member_casual, ride_length, day_of_week, start_station_name, end_station_name
FROM `cyclistic-marketing-project.winter_2022_2023.dec-2022-jan-feb-2023_winter`
```

## Create a table
```
CREATE TABLE `cyclistic-marketing-project.union_table.union_seasons`
AS (
 SELECT DISTINCT ride_id, rideable_type, started_at, ended_at, start_lat, start_lng, end_lat, end_lng, member_casual, ride_length, day_of_week, start_station_name, end_station_name
 FROM (
   SELECT * FROM `cyclistic-marketing-project.autumn_2022.sep-oct-nov-2022`
   UNION ALL
   SELECT * FROM `cyclistic-marketing-project.spring_2022.march-april-may-2022`
   UNION ALL
   SELECT * FROM `cyclistic-marketing-project.summer_2022.june-july-august-2022`
   UNION ALL
   SELECT * FROM `cyclistic-marketing-project.winter_2022_2023.dec-2022-jan-feb-2023_winter`
 )
)
```

## Count duplicates
```
SELECT ride_id, COUNT(*) as duplicate_count
FROM cyclistic-marketing-project.union_table.union_seasons
GROUP BY ride_id
HAVING COUNT(*) > 1
```

## Delete duplicates
```
DELETE FROM `cyclistic-marketing-project.union_table.union_seasons`
WHERE ride_id IN (SELECT ride_id
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY ride_id HAVING COUNT(ride_id) > 1);
```

## Delete nulls
```
DELETE FROM `cyclistic-marketing-project.union_table.union_seasons`
WHERE start_station_name IS NULL OR end_station_name IS NULL;
```

## Check for spaces before and after string values
```
SELECT rideable_type,
start_station_name,
end_station_name,
member_casual
FROM `cyclistic-marketing-project.union_table.union_seasons`
WHERE rideable_type LIKE ' %' OR rideable_type LIKE '% '
OR start_station_name LIKE ' %' OR rideable_type LIKE '% '
OR end_station_name LIKE ' %' OR rideable_type LIKE '% ';
```

## Member type percentage
```
SELECT member_casual,
COUNT(member_casual) AS member_casual_count,
ROUND(COUNT(member_casual) / SUM(COUNT(member_casual)) OVER() * 100, 2) AS percentage
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY member_casual;
```

## Preferred bike type per member type percentage
```
SELECT member_casual, rideable_type,
 COUNT(rideable_type) AS rideable_type_count,
 ROUND((COUNT(rideable_type) / SUM(COUNT(rideable_type)) OVER(PARTITION BY member_casual) * 100), 2) AS percentage
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY member_casual, rideable_type;
```

## Number of rides by day of the week per member type
```
SELECT member_casual,
day_of_week,
COUNT(ride_id) AS rides_count
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY member_casual, day_of_week
ORDER BY member_casual;
```

## Average ride duration per member type
```
SELECT member_casual,
ROUND(AVG(DATE_DIFF(ended_at, started_at, minute)),0) AS avg_ride_duration
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY member_casual;
```

## Number of rides by day of the week
```
SELECT day_of_week, COUNT(ride_id) AS rides_count
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY day_of_week
ORDER BY day_of_week;
```

## Average ride duration by day of the week per member type
```
SELECT member_casual,
day_of_week,
ROUND(AVG(DATE_DIFF(ended_at, started_at, minute)),0) AS avg_ride_duration
FROM `cyclistic-marketing-project.union_table.union_seasons`
GROUP BY member_casual, day_of_week
ORDER BY member_casual;
```
