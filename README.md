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
## Analysis  
First, I wrote queries to find distribution of bike preferrences across member types  
Demographic of member types
```
SELECT COUNT(*) as count, member_type
FROM bike-share-415816.okay.acc_data
GROUP BY member_type
ORDER BY member_type
```
![Screenshot 2024-04-27 152500](https://github.com/prabinprojects/photos/assets/163358902/a452e2be-60e9-4d70-89fc-59043c1b58e9)  
  
### Distribution of bike types across members  
```
SELECT COUNT(*) as trips, rideable_type
FROM bike-share-415816.okay.acc_data
GROUP BY rideable_type
ORDER BY trips
```
![Screenshot 2024-04-27 152933](https://github.com/prabinprojects/photos/assets/163358902/2e793ca0-3873-40b3-993a-6757cece2801)  

Number of trips by `Month`  
`Casual`
```
SELECT COUNT(*) AS TRIPS, month
from bike-share-415816.okay.acc_data
where member_type="casual"
group by month, member_type 
order by TRIPS
```
![Screenshot 2024-04-27 153824](https://github.com/prabinprojects/photos/assets/163358902/1c3155b3-1c05-45d0-bb47-de988c8ead9c)  

`Member` 
```
SELECT COUNT(*) AS TRIPS, month
from bike-share-415816.okay.acc_data
where member_type="member"
group by month, member_type 
order by TRIPS
```
![Screenshot 2024-04-27 153939](https://github.com/prabinprojects/photos/assets/163358902/ebb24ae6-dda3-40af-8936-61f98a039819)  

### Distribution of rides by day of the week  
`Casual`  
```
SELECT day_of_week, count(day_of_week) as DAY
FROM bike-share-415816.okay.acc_data
WHERE member_type = "casual"
GROUP BY day_of_week
ORDER BY 
CASE
WHEN day_of_week = "SUN" then 1
WHEN day_of_week = "MON" then 2
WHEN day_of_week = "TUE" then 3
WHEN day_of_week = "WED" then 4
WHEN day_of_week = "THU" then 5
WHEN day_of_week = "FRI" then 6
WHEN day_of_week = "SAT" then 7
END ASC
```
![Screenshot 2024-04-27 154502](https://github.com/prabinprojects/photos/assets/163358902/5e3f139d-d9a0-48f2-a4b1-98ad3022877c)  

`Member`
```
SELECT day_of_week, count(day_of_week) as DAY
FROM bike-share-415816.okay.acc_data
WHERE member_type = "member"
GROUP BY day_of_week
ORDER BY 
CASE
WHEN day_of_week = "SUN" then 1
WHEN day_of_week = "MON" then 2
WHEN day_of_week = "TUE" then 3
WHEN day_of_week = "WED" then 4
WHEN day_of_week = "THU" then 5
WHEN day_of_week = "FRI" then 6
WHEN day_of_week = "SAT" then 7
END ASC
```
![Screenshot 2024-04-27 154714](https://github.com/prabinprojects/photos/assets/163358902/31a0c408-7f0a-41e6-ad8e-0f5c47f92fff)  

### Hourly trips pattern  
`Casual`  
```
SELECT EXTRACT (hour FROM started_at) as TIME_OF_TRIP, count(*) as TRIPS
FROM bike-share-415816.okay.acc_data
GROUP BY time_of_trip, member_type
HAVING member_type = "casual"
ORDER BY TRIPS DESC
```
![Screenshot 2024-04-27 160219](https://github.com/prabinprojects/photos/assets/163358902/f810ebca-a919-443f-82c0-54207f740cc4)
  
`Member`

```
SELECT extract (hour from started_at) as TIME_OF_TRIP, count(*) as TRIPS
FROM bike-share-415816.okay.acc_data
GROUP BY time_of_trip, member_type
HAVING member_type = "member"
ORDER BY TRIPS DESC
```
![Screenshot 2024-04-27 155639](https://github.com/prabinprojects/photos/assets/163358902/48d33afc-2e8f-4a85-8b04-848abef8cfca)  

### Average ride length
Average ride length by `Month`  
  
`Casual`
```
SELECT ROUND(AVG(ride_length_m),2) AS AVG_RIDE, month
FROM bike-share-415816.okay.acc_data
WHERE member_type = 'casual'
GROUP BY month, member_type
ORDER BY AVG_RIDE
```
![Screenshot 2024-04-27 161630](https://github.com/prabinprojects/photos/assets/163358902/50d2b6f4-30b5-4b8f-96f3-d18ecf129e6b)  

`Member`  
```
SELECT ROUND(AVG(ride_length_m),2) AS AVG_RIDE, month
FROM bike-share-415816.okay.acc_data
WHERE member_type = 'member'
GROUP BY month, member_type
ORDER BY AVG_RIDE
```
![Screenshot 2024-04-27 161809](https://github.com/prabinprojects/photos/assets/163358902/c76e6118-79a7-4003-9289-fe9f2addaa40)  

Average ride length by `DAY`  
  
`Casual`
```
SELECT ROUND(AVG(ride_length_m),2) AS AVG_RIDE_LEN, day_of_week
FROM bike-share-415816.okay.acc_data
where member_type = 'casual'
GROUP BY day_of_week, member_type
ORDER BY AVG_RIDE_LEN
```  
![Screenshot 2024-04-27 162305](https://github.com/prabinprojects/photos/assets/163358902/7827ff32-0b6b-41a1-a5f1-2878b533bbe1)  

`Member`  
```
SELECT ROUND(AVG(ride_length_m),2) AS AVG_RIDE_LEN, day_of_week
FROM bike-share-415816.okay.acc_data
where member_type = 'member'
GROUP BY day_of_week, member_type
ORDER BY AVG_RIDE_LEN
```
![Screenshot 2024-04-27 162124](https://github.com/prabinprojects/photos/assets/163358902/78f84f70-241c-452b-bb30-cfa287084a8e)

  
Average ride length by `Hour`  
  
`Casual` 
```
SELECT EXTRACT (hour FROM started_at) AS TIME_OF_TRIP, ROUND(AVG(ride_length_m),2) AS AVG_RIDE_LEN
FROM bike-share-415816.okay.acc_data
GROUP BY time_of_trip, member_type
HAVING member_type = "casual"
ORDER BY AVG_RIDE_LEN DESC
```
![Screenshot 2024-04-27 162941](https://github.com/prabinprojects/photos/assets/163358902/1730e746-27bb-41fe-825e-fded261cf4f6)  


`Member`  
```
SELECT EXTRACT (hour FROM started_at) AS TIME_OF_TRIP, ROUND(AVG(ride_length_m),2) AS AVG_RIDE_LEN
FROM bike-share-415816.okay.acc_data
GROUP BY time_of_trip, member_type
HAVING member_type = "member"
ORDER BY AVG_RIDE_LEN DESC
```
![Screenshot 2024-04-27 163315](https://github.com/prabinprojects/photos/assets/163358902/e3afc478-c4f0-4196-ad95-86720968fca0)

# Top stations by member type
`Casual`  
```
SELECT 
DISTINCT start_station_name, count(*) as TRIPS
FROM bike-share-415816.okay.acc_data
WHERE member_type = "casual"
GROUP BY start_station_name, member_type
ORDER BY TRIPS
DESC LIMIT 10
```
![Screenshot 2024-04-27 164122](https://github.com/prabinprojects/photos/assets/163358902/491d663c-2ab4-4cfc-85cf-143595c82aeb)  

`Member`  
```
SELECT 
DISTINCT start_station_name, count(*) as TRIPS
FROM bike-share-415816.okay.acc_data
WHERE member_type = "member"
GROUP BY start_station_name, member_type
ORDER BY TRIPS
DESC LIMIT 10
```
![Screenshot 2024-04-27 164308](https://github.com/prabinprojects/photos/assets/163358902/02f67f88-6c0c-478e-bb81-03981f45d3f8)  


















