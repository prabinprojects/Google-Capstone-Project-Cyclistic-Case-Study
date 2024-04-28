# **Google Capstone Project: Cyclistic Case Study**
## Introduction
This project is a part of the Google Professional Data Analytics Course. In this report, I will be performing tasks similar to Junior Data Anlayst to draw insights from a fictional company called Cyclistic that provides bike services in the city of Chicago. The methodology of data analysis carried out for this project is adopted from the program that follows the steps of ASK, PREPARE, PROCESS, ANALYZE AND ACT.
## Scenario
I am a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, the team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, my team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve derived recommendations, so they must be backed up with compelling data insights and professional data visualizations.  
## ASK  
### Business Task  
Formulate marketing strategies to convert casual riders into members.  
### Key Questions  
1. How do annual members differ from casual riders?
2. How can casual riders be convinced to change their membership to annual?

The directory of the team has assigned me to answer the first question; How do annual members differ from casual riders?  
## Prepare  
Historic data of bike trips of Cysclistic's customers is downloaded from [diivvy_tripdata](https://divvy-tripdata.s3.amazonaws.com/index.html) . The data has been made available by Motivate International Inc. under this [licence](https://divvybikes.com/data-license-agreement). I used 12 months worth of bike usage data relating to the year 2023. For confidentiality reasons, all personally identifiably information has been removed. Each file contains 13 columns; ride_id, rideable_type, started_at, ended_at, start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, member_casual.  

## Process  
I began by downloading 12 csv files contaning data from January 2023 to December 2023. Following that, I uploaded all the files on Big Query to iniate cleaning and processing of data. Due to file size limit, the files were split in Excel and uploaded in Bigquery as separate tables. Firstly, I used `UNION ALL` to integrate all the months into a single table.  
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
I created separate tables for each quarter to optimise analysis.  
```
CREATE TABLE bike-share-415816.okay.Q1 as
(SELECT * FROM bike-share-415816.okay.January
UNION ALL
SELECT * FROM bike-share-415816.okay.February
UNION ALL
SELECT * FROM bike-share-415816.okay.March
)
```  
Free trial version of Big Query does not support ammend functions such as add or delete therefore I transferred non null values into another table and named it `combined_data1`.
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
I then created new columns by extracting `HOUR`, `DAY`, `MONTH` from `combined_data1` and integrated them into new table called `acc_data`.  
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
### Overall distribution of members
```
SELECT COUNT(*) as count, member_type
FROM bike-share-415816.okay.acc_data
GROUP BY member_type
ORDER BY member_type
```
![Screenshot 2024-04-27 152500](https://github.com/prabinprojects/photos/assets/163358902/a452e2be-60e9-4d70-89fc-59043c1b58e9)  
  
### Distribution of members across bike types
```
SELECT COUNT(*) as trips, rideable_type
FROM bike-share-415816.okay.acc_data
GROUP BY rideable_type
ORDER BY trips
```
![Screenshot 2024-04-27 152933](https://github.com/prabinprojects/photos/assets/163358902/2e793ca0-3873-40b3-993a-6757cece2801)  

### Trips by `Month`  
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

### Trips vs Day of week 
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
#### Average ride length by `Month`  
  
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

#### Average ride length by `DAY`  
  
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

  
#### Average ride length by `Hour`  
  
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

## Most popular stations
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

## Key Findings  

#### Statistics of riders across `BIKE TYPES`  
Bike Type   |    Riders Distribution
:----------:|:----------------------:
![Screenshot 2024-04-28 173921](https://github.com/prabinprojects/photos/assets/163358902/723b748f-f65f-4c5f-bb50-84c8fca1bbb7)|![Screenshot 2024-04-28 173604](https://github.com/prabinprojects/photos/assets/163358902/1784b40c-ef7f-4764-a45f-3d7f800b8cb4)

Majority of the customers prefer classic bike over other types. Interestingly, docked bikes are only used by casual riders indicating characteristics of spontaniety rather than a routinely bheaviour. More than 60% of customers have annual membership.
#### `Rides per Quarter` 

![Screenshot 2024-04-22 142723](https://github.com/prabinprojects/photos/assets/163358902/28416d2e-ada0-4981-8114-ecb0ae8babbf)  

Q4 and Q3 boasts the highest number of rides in both member types. Naturally, Q2 and Q1 fall behind in numbers with affecting factors likely to be cold weather. It is evident that customer consumption is seen highest in summer months rather than winter months and this behaviour is identical in both casual and members.  

#### `Monthly Rides`  

![Screenshot 2024-04-22 150233](https://github.com/prabinprojects/photos/assets/163358902/a2ec4aa1-8e0e-476f-96a4-ded295f11cb3)  

When compared statistics of monthly rides by membership type,  a similar trend in customer behaviour is seen in both customers. The number of rides grow rapidly during transition from spring to summer and fall sharply during the winter months. Members in general exhibit higher numbers than casual riders and this trend in maintained throughout the calendar year.  

#### Ride distribution by `DAY OF WEEK`  
  
Casual              |               Member
:------------------:|:-------------------:
![Screenshot 2024-04-28 170210](https://github.com/prabinprojects/photos/assets/163358902/ba987648-12bd-4c57-aa5b-82a3603b27eb)|![Screenshot 2024-04-28 170227](https://github.com/prabinprojects/photos/assets/163358902/6fff4a97-4570-4d79-835f-73cc9524d0b5)  

As seen in the set of bar graphs above, casual riders seem to be most active during weekends while the numbers shrink in the weekdays. Conversely, members tend to use the bikes consistently in high numbers throughout the week followed by sharp decline in the weekends. Peak rides in casual riders can be seen on Saturday with Sunday as the second highest performing day. Whereas Tuesday, Wednesday and Thursday seem to be more popular choices for members and they all share identical ride statistics. The data suggests that members might predominantly be using the bikes for commuting to work while casual riders do it for recreational purposes.  

#### Hourly Trips pattern  

![Screenshot 2024-04-22 151713](https://github.com/prabinprojects/photos/assets/163358902/55a8c512-4cb0-4c3d-930b-f4969be24103)  

As seen in the graph above, the usage of bikes by casual riders rise gradually from 5 AM untill its peak at 5 PM followed by a sharp fall. Although both member types share similar peak and trend line, a sharp spike is observed from 6 to 8 AM in annual members which strongly suggests that rides are being used to commute to work. It is possible that casual riders might be the using bikes only on their way back from work or simply for recreational purposes.
#### Average ride length by `MONTH`  

![Screenshot 2024-04-22 162606](https://github.com/prabinprojects/photos/assets/163358902/0fc3c8eb-450c-411d-b4fc-7dd5434b6a8c)  

When average ride of is compared it is discovered that the length of trips is highest in the spring and summer months and fall after for both type of customers. Although, more trips are taken by annual members, casual members seem to use it for a longer period of time. According to the data average ride length by casual riders is nearly double the length of trips by members.  

#### Average ride length by `DAY OF WEEK`  

![Screenshot 2024-04-22 171656](https://github.com/prabinprojects/photos/assets/163358902/8658dfdb-fcea-42dd-87d2-dea97c43dd29)  

The graph above shows similar trend is exhibited by both type of riders where peak average ride length is reached on the weekend. The increase in average ride length is steeper in casual riders than annual members.  

#### Average ride length by 
![Screenshot 2024-04-22 160151](https://github.com/prabinprojects/photos/assets/163358902/6d35fa42-d89e-4f16-8aff-457af74acdf7)  

Average ride length is seen highest during the mid-day for casual riders and lowest in the early hours. Meanwhile, average ride length stays of annual members stay consistent between 10 to 12 minutes.  

#### `TOP DESTINATIONS`  

Casual  |  Member
:---------:|:----------:
![Screenshot 2024-04-22 184800](https://github.com/prabinprojects/photos/assets/163358902/42c43698-e8b9-4ee5-9e2f-fe21d83827d8)|![Screenshot 2024-04-22 184705](https://github.com/prabinprojects/photos/assets/163358902/a75c4169-f4c4-46eb-a99e-a8af1440af42)  

The maps above reveal most members started their rides in the vicinity of universities, hospitals, local parks, restaurants, train stations, office blocks and water bodies. Whereas casual riders frequented rides starting from beaches, piers, museums, and large parks. This directly correlates to the aforementioned findings leading to a confirmation that casual riders use the bikes for recreational purposes while members may use it for commute or to support their lifestyle.  

## ACT  

### Key takeaways 
- Most weekday rides are by members and most weekend rides are by casual riders.
- Members are using the bikes to commute to work in addition to recreational purposes where as casual riders are predominantly using bikes for recreational purposes.
- Average ride length of casual riders is almost double than of members'.
- High traffic of member' rides is seen in the vicinity of office blocks, local parks, restaurants, universities and hospitals.
- High traffic of casual rides can be observed near the beach, museums, piers and large parks.  

### Recommendations  
Heare are my top recommendations based on the insights derived from my analysis.  
- A flexible pricing plan can be pushed forward specifically targeting casual riders that allow them to use the bikes on their preferred times.
- Targeted marketing campaigns should be implemented before the start of Spring to attract prospective customers since the amount of riders peak from Spring to Summer months.
- High traffic of riders is observed near water bodies, parks and museums suggesting the bikes are being used for recreational purposes. Marketing posters or banners can be set up in these locations to attract new customers.
- Implementation of an inhouse app is highly recommended. The data gathered from the app can be instrumental in conversion of casual riders into annual members. For example, it can hel p with identifying casual riders that exhibit similar characteristics like annual members. Marketing campaigns can be integrated into the app , this allows for more exposure leading to an increased chance of turnover.

### Dashboard  
![Screenshot 2024-04-28 172912](https://github.com/prabinprojects/photos/assets/163358902/9d8319b2-f99e-467c-9f97-bae77034156a)
















