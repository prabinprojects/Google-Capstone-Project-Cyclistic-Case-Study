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




