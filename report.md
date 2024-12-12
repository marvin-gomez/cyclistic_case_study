### How does a bike-share navigate speedy success?

**Introduction:**

Hi everyone, I have been working on the Google Data Analytics Professional Certificate to further my analytical skills. I am excited to showcase my new skills and knowledge through this capstone project.    

In this case study, I will be analyzing a public dataset for a fictional company provided by Coursera. I will be using Microsoft Excel and SQL for data analysis and Tableau for visualizations.

**Scenario:**

Cyclistic, a bike-share company in Chicago that features more than 5,800 bicycles and 600 docking stations. Cyclistic wants to understand how casual riders and annual members use Cyclistic bikes differently with a goal to maximize the number of annual memberships.  

**Objective:**

  •	Understand how annual members and casual riders use Cyclistic bikes differently.

  •	Create digital marketing strategies aimed at converting casual riders into Cyclistic members.

**Data Analysis Process**

In this project, I will follow the six steps of the data analysis process in order to solve this problem.  The six steps include:

1.	Ask
2.	Prepare
3.	Process
4.	Analyze
5.	Share
6.	Act

**Step 1: Ask**

Key Task: Understand how annual members and casual riders use Cyclistic bikes differently and create digital marketing strategies aimed at converting casual riders into Cyclistic members.

Three questions that need to be answered are:
1.	How do annual members and casual riders use Cyclistic bikes differently?
2.	Why would casual riders buy Cyclistic annual memberships?
3.	How can Cyclistic use digital media to influence casual riders to become members?

**Step 2: Prepare**

The data that I will be using is Cyclistic’s historical trip data.  The data used in this analysis is publicly available and provided by Motivate International Inc., ensuring transparency and credibility. It covers the past 12 months of bike trip data, making it current and relevant for identifying trends. The dataset is comprehensive, including various ride metrics such as trip duration, start and end times, and user type, ensuring robust analysis. Privacy measures are in place to prevent bias, as no personally identifiable information is included. Additionally, the data source is appropriately cited, adhering to licensing agreements, which further enhances its reliability.

For this case study, the datasets I have chosen for analysis cover a 12 month period from November 2023 to October 2024.  All trip data is in a comma-delimited (.CSV) format with 13 columns including:

1.	ride_id - Unique identifier for each ride booking
2.	rideable_type - Type of bicycle used
3.	started_at - Date and time of the start of the ride
4.	ended_at - Date and time of the end of the ride
5.	start_station_name - Name of the starting station
6.	start_station_id - ID of the starting station
7.	end_station_name - Name of the end station
8.	end_station_id - ID of the end station
9.	start_lat - Latitude of the start station
10.	start_lng - Longitude of the start station
11.	end_lat - Latitude of the end station
12.	end_lng - Longitude of the end station
13.	member_casual - User type

**Step 3: Process**

In this step, I will document all data cleaning and manipulation processes. I have chosen to use SQL due to its ability to efficiently handle large datasets far faster than traditional spreadsheet tools. SQL excels in filtering, transforming, and cleaning data while ensuring data integrity and integration with other tools such as Tableau and R.

```SQL
--Understand the Schema, review data types and identify critical fields
SELECT *
FROM 202410_trip_data;


--Combine all 12 tables
CREATE TABLE `alldata_trip_data` AS
    SELECT * 
    FROM `202410_trip_data`
    UNION ALL
    SELECT * 
    FROM `202409_trip_data`
    UNION ALL
    SELECT * 
    FROM `202408_trip_data`
    UNION ALL
    SELECT * 
    FROM `202407_trip_data`
    UNION ALL
    SELECT * 
    FROM `202406_trip_data`
    UNION ALL
    SELECT * 
    FROM `202405_trip_data`
    UNION ALL
    SELECT * 
    FROM `202404_trip_data`
    UNION ALL
    SELECT * 
    FROM `202403_trip_data`
    UNION ALL
    SELECT * 
    FROM `202402_trip_data`
    UNION ALL
    SELECT * 
    FROM `202401_trip_data`
    UNION ALL
    SELECT * 
    FROM `202312_trip_data`
    UNION ALL
    SELECT * 
    FROM `202311_trip_data`;


--Find duplicate records
SELECT ride_id, COUNT(ride_id)
FROM alldata_trip_data
GROUP BY ride_id
HAVING COUNT(ride_id) > 1


--Delete duplicates by creating a new table excluding duplicates
CREATE TABLE alldata_trip_data_v2 AS
WITH CTE AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY ride_id ORDER BY started_at) AS row_num
    FROM `alldata_trip_data`
)
SELECT *
FROM CTE
WHERE row_num = 1;


--Identify columns with null values
SELECT 
COUNT(*) - COUNT(ride_id) AS ride_id,
COUNT(*) - COUNT(rideable_type) AS rideable_type,
COUNT(*) - COUNT(started_at) AS started_at,
COUNT(*) - COUNT(ended_at) AS ended_at,
COUNT(*) - COUNT(start_station_name) AS start_station_name,
COUNT(*) - COUNT(start_station_id) AS start_station_id,
COUNT(*) - COUNT(end_station_name) AS end_station_name,
COUNT(*) - COUNT(end_station_id) AS end_station_id,
COUNT(*) - COUNT(start_lat) AS start_lat,
COUNT(*) - COUNT(start_lng) AS start_lng,
COUNT(*) - COUNT(end_lat) AS end_lat,
COUNT(*) - COUNT(end_lng) AS end_lng,
COUNT(*) - COUNT(member_casual) AS member_casual
FROM alldata_trip_data;


--Output rows with null values 
SELECT * 
FROM alldata_trip_data_v2
WHERE start_station_name IS NULL 
  AND start_station_id IS NULL
  AND end_station_name IS NULL
  AND end_station_id IS NULL


--Delete rows with no station id and station name
DELETE FROM alldata_trip_data_v2
WHERE start_station_name IS NULL 
  AND start_station_id IS NULL
  AND end_station_name IS NULL
  AND end_station_id IS NULL


--Delete rows where end_lat and end_lng are null
DELETE FROM alldata_trip_data_v2
WHERE end_lat IS NULL
  AND end_lng IS NULL;


--Identify invalid data in ride lengths with zero or negative value
SELECT * 
FROM alldata_trip_data_v2
WHERE TIMESTAMP_DIFF(ended_at, started_at, SECOND) <= 0;


--Delete rows with ride_lengths <= 0
DELETE FROM alldata_trip_data_v2
WHERE TIMESTAMP_DIFF(ended_at, started_at, SECOND) <= 0;


--Check for logical inconsistencies 
SELECT  *
FROM alldata_trip_data_v2
WHERE start_station_name IS NOT NULL
  AND start_station_name = end_station_name;


--Identify then standardize categorical data rideable_type
SELECT DISTINCT rideable_type
FROM alldata_trip_data_v2;


--identify then standardize categorical data member_casual
SELECT DISTINCT member_casual
FROM alldata_trip_data_v2;


--Add derived column ride_length_minutes
SELECT ride_id, started_at, ended_at, TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS ride_length_minutes
FROM alldata_trip_data_v2;


--Add derived column ride_month
SELECT started_at, 
    CASE 
      WHEN EXTRACT(MONTH FROM started_at) = 1 THEN 'January'
      WHEN EXTRACT(MONTH FROM started_at) = 2 THEN 'February'
      WHEN EXTRACT(MONTH FROM started_at) = 3 THEN 'March'
      WHEN EXTRACT(MONTH FROM started_at) = 4 THEN 'April'
      WHEN EXTRACT(MONTH FROM started_at) = 5 THEN 'May'
      WHEN EXTRACT(MONTH FROM started_at) = 6 THEN 'June'
      WHEN EXTRACT(MONTH FROM started_at) = 7 THEN 'July'
      WHEN EXTRACT(MONTH FROM started_at) = 8 THEN 'August'
      WHEN EXTRACT(MONTH FROM started_at) = 9 THEN 'September'
      WHEN EXTRACT(MONTH FROM started_at) = 10 THEN 'October'
      WHEN EXTRACT(MONTH FROM started_at) = 11 THEN 'November'
      WHEN EXTRACT(MONTH FROM started_at) = 12 THEN 'December'
    END as ride_month
FROM alldata_trip_data_v2;


--Add derived column day_of_week
SELECT started_at, 
  CASE
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 1 THEN 'Sunday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 2 THEN 'Monday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 3 THEN 'Tuesday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 4 THEN 'Wednesday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 5 THEN 'Thursday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 6 THEN 'Friday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 7 THEN 'Saturday'
    END as day_of_week
FROM alldata_trip_data_v2;


--Organize and create new cleaned sheet
CREATE TABLE alldatacleaned_trip_data AS
(
SELECT ride_id, 
  member_casual, 
  rideable_type, 
  TIMESTAMP_DIFF(ended_at, started_at, MINUTE) AS ride_length_minutes, 
  started_at, 
  ended_at, 
  CASE 
     WHEN EXTRACT(MONTH FROM started_at) = 1 THEN 'January'
      WHEN EXTRACT(MONTH FROM started_at) = 2 THEN 'February'
      WHEN EXTRACT(MONTH FROM started_at) = 3 THEN 'March'
      WHEN EXTRACT(MONTH FROM started_at) = 4 THEN 'April'
      WHEN EXTRACT(MONTH FROM started_at) = 5 THEN 'May'
      WHEN EXTRACT(MONTH FROM started_at) = 6 THEN 'June'
      WHEN EXTRACT(MONTH FROM started_at) = 7 THEN 'July'
      WHEN EXTRACT(MONTH FROM started_at) = 8 THEN 'August'
      WHEN EXTRACT(MONTH FROM started_at) = 9 THEN 'September'
      WHEN EXTRACT(MONTH FROM started_at) = 10 THEN 'October'
      WHEN EXTRACT(MONTH FROM started_at) = 11 THEN 'November'
      WHEN EXTRACT(MONTH FROM started_at) = 12 THEN 'December'
    END AS ride_month,
  CASE
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 1 THEN 'Sunday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 2 THEN 'Monday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 3 THEN 'Tuesday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 4 THEN 'Wednesday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 5 THEN 'Thursday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 6 THEN 'Friday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 7 THEN 'Saturday'
    END as day_of_week,
    start_lat,
    start_lng,
    end_lat,
    end_lng,
    start_station_name,
    start_station_id,
    end_station_name,
    end_station_id
FROM alldata_trip_data_v2
);
```





