# **Google Capstone Project: Cyclistic Case Study**
## Introduction
This project is part of the Google Professional Data Analytics Course. I will be performing tasks similar to Junior Data Anlayst to draw insights from a fictional bike services provider company called Cyclistic. I will be following data analysis methodology adopted from the program that follows the steps of ASK, PREPARE, PROCESS, ANALYZE AND ACT.
## Scenario
You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.  
## ASK  
### Business Task  
Formulate marketing strategies to convert casual riders into members.  
### Key Questions  
1. How do annual members differ from casual riders?
2. How can casual riders be convinced to change their membership to annual?

The directory of the team has assigned me to answer the first question; How do annual members differ from casual riders?  
## Prepare  
The source of data is [diivvy_tripdata](https://divvy-tripdata.s3.amazonaws.com/index.html) which contains historical trip data of bike usage in the city of Chicago. The data has been made available by Motivate International Inc. under this [licence](https://divvybikes.com/data-license-agreement). I will be using 12 months of bike usage data relating to the year 2023. For confidentiality reasons, all personally identifiably information has been removed. Each file contains 13 columns; ride_id, rideable_type, started_at, ended_at, start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual.  

## Process  
I began my downloading 12 csv files contaning data from January 2023 to December 2023. Following that, I uploaded all the files on Big Query to iniate cleaning and processing of data. Due to file size limit of the platform, I split some files in Excel and uploaded them in Bigquery as separate tables. I used `UNION ALL` to integrate all the months into a single table.  
```
CREATE TABLE if not exists bike-share-415816.okay.combined_data
AS
(SELECT * FROM bike-share-415816.okay.January
UNION ALL
SELECT * FROM bike-share-415816.okay.February
UNION ALL 
SELECT * FROM bike-share-415816.okay.March
UNION ALL
SELECT * FROM bike-share-415816.okay.April
UNION ALL
SELECT * FROM bike-share-415816.okay.May
UNION ALL
SELECT * FROM bike-share-415816.okay.May2
UNION ALL
SELECT * FROM bike-share-415816.okay.June
UNION ALL
SELECT * FROM bike-share-415816.okay.June2
UNION ALL
SELECT * FROM bike-share-415816.okay.July
UNION ALL
SELECT * FROM bike-share-415816.okay.July2
UNION ALL
SELECT * FROM bike-share-415816.okay.August
UNION ALL
SELECT * FROM bike-share-415816.okay.August2
UNION ALL
SELECT * FROM bike-share-415816.okay.September
UNION ALL
SELECT * FROM bike-share-415816.okay.September2
UNION ALL
SELECT * FROM bike-share-415816.okay.October
UNION ALL
SELECT * FROM bike-share-415816.okay.October2
UNION ALL
SELECT * FROM bike-share-415816.okay.November
UNION ALL
SELECT * FROM bike-share-415816.okay.December);
```
Before carrying out analysis it is important to check for any anomalies in the data. In order to do that I first checked for any null data.  
Checking `NULL` data.  
```
SELECT COUNT(*) - COUNT(ride_id) ride_id,
COUNT(*) - COUNT(rideable_type) rideable_type,
COUNT(*) - COUNT(started_at) started_at,
COUNT(*) - COUNT(ended_at) ended_at,
COUNT(*) - COUNT(start_station_name) start_station_name,
COUNT(*) - COUNT(start_station_id) start_station_id,
COUNT(*) - COUNT(end_station_name) end_station_name,
COUNT(*) - COUNT(end_station_id) end_station_id,
COUNT(*) - COUNT(start_lat) start_lat,
COUNT(*) - COUNT(end_lat) end_lat,
COUNT(*) - COUNT(end_lng) end_lng,
COUNT(*) - COUNT(member_casual) member_casual
FROM bike-share-415816.okay.combined_data
```
![Screenshot 2024-03-29 154759](https://github.com/prabinprojects/photos/assets/163358902/f686d1ca-3dbb-4285-a012-7f33613ebfb5)  

I then checked for duplicate rows by using the following query.  
```
SELECT *
FROM bike-share-415816.okay.combined_data
WHERE ride_id (SELECT ride_id
               FROM bike-share-415816.okay.combined_data
               GROUP BY ride_id
               HAVING COUNT(ride_id)>1
```
I found that there were no duplicate entries in the table.  
I wrote queries to create table for quarters to analyse quarterly bike usage behaviour.  
```
CREATE TABLE bike-share-415816.okay.Q1 as
(SELECT * FROM bike-share-415816.okay.January
UNION ALL
SELECT * FROM bike-share-415816.okay.February
UNION ALL
SELECT * FROM bike-share-415816.okay.March
)
```  
The free trial veresion of Big Query does not support ammend functions such as add or delete therefore I transferred non null values into another table and named it `combined_data1`.
```
CREATE TABLE bike-share-415816.okay.combined_data1
AS
(SELECT * FROM bike-share-415816.okay.combined_data
WHERE 
start_station_name IS NOT NULL and
end_station_name IS NOT NULL and
end_station_id IS NOT NULL and
end_lng IS NOT null and
start_station_id IS NOT NULL);
```
I followed the same procedure for all the quarters for accurate quartely analysis.  
I then extracted `HOUR`, `DAY`, `MONTH` from `combined_data1` and integrated them into new table called `acc_data`.  
```
CREATE TABLE bike-share-415816.okay.acc_data
AS 
(
SELECT ride_id, rideable_type,start_station_name,end_station_name,start_lat,start_lng,end_lat,end_lng,member_casual as member_type,started_at,ended_at,
CASE
WHEN EXTRACT (DAYOFWEEK  from started_at) = 1 THEN 'SUN'
WHEN EXTRACT (DAYOFWEEK  from started_at) = 2 THEN 'MON'
WHEN EXTRACT (DAYOFWEEK  from started_at) = 3 THEN 'TUE'
WHEN EXTRACT (DAYOFWEEK  from started_at) = 4 THEN 'WED'
WHEN EXTRACT (DAYOFWEEK  from started_at) = 5 THEN 'THU'
WHEN EXTRACT (DAYOFWEEK  from started_at) = 6 THEN 'FRI'
ELSE 'SAT'
END AS day_of_week,
CASE
  WHEN EXTRACT (month from started_at) = 01 THEN "JAN"
  WHEN EXTRACT (month from started_at) = 02 THEN "FEB"
  WHEN EXTRACT (month from started_at) = 03 THEN "MAR"
  WHEN EXTRACT (month from started_at) = 04 THEN "APR"
  WHEN EXTRACT (month from started_at) = 05 THEN "MAY"
  WHEN EXTRACT (month from started_at) = 06 THEN "JUN"
  WHEN EXTRACT (month from started_at) = 07 THEN "JUL"
  WHEN EXTRACT (month from started_at) = 08 THEN "AUG"
  WHEN EXTRACT (month from started_at) = 09 THEN "OCT"
  WHEN EXTRACT (month from started_at) = 11 THEN "NOV"
  WHEN EXTRACT (month from started_at) = 12 THEN "DEC"
ELSE "UNKOWN"
END AS month,
timestamp_diff(ended_at, started_at, minute) AS ride_length_m,
format_timestamp("%I:%M %p", started_at) AS time
FROM bike-share-415816.okay.combined_data1
WHERE timestamp_diff(ended_at, started_at, minute) > 1 and timestamp_diff(ended_at, started_at, hour) < 24);
```





