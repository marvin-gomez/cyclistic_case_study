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

|Column #|Column Name|Data Tye|Description|
|-----|---------------|----------|------------------|
|1 |ride_id|STRING|	Unique identifier for each ride booking|
|2|	rideable_type	|STRING|	Type of bicycle used |
|3|	started_at	|TIMESTAMP	|Date and time of the start of the ride|
|4|	ended_at	|TIMESTAMP|	Date and time of the end of the ride|
|5|	start_station_name	|STRING|	Name of the starting station|
|6|	start_station_id	|STRING|	ID of the starting station|
|7|	end_station_name|	STRING|	Name of the end station|
|8|	end_station_id	|STRING|	ID of the end station|
|9|	start_lat	|FLOAT	|Latitude of the start station|
|10|	start_lng	|FLOAT|	Longitude of the start station|
|11|	end_lat	|FLOAT	|Latitude of the end station|
|12|	end_lng	|FLOAT|	Longitude of the end station|
|13|	member_casual|	STRING|	User type|

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

**Step 4&5: Analyze & Share**

I will analyze the data to uncover key insights by comparing ride patterns between members and casual users, identifying trends in ride durations, popular bike types, and frequently used stations. Using statistical analysis and visualizations, I will highlight significant differences and correlations that inform user behavior.

My data visualizations can be found on tableau public [here.]( https://public.tableau.com/app/profile/marvin.gomez1953/viz/CylisticData/Story1)

Firstly, both casual users and members predominantly use Cyclistic bikes within the Chicago area, indicating that their geographical usage patterns are largely similar.

![](https://github.com/mgomez024/cyclistic_case_study/blob/main/Data%20visuals/heatmap.jpg)

Here are the top three most frequently used stations, showcasing their popularity as key hubs for both casual users and members.

```sql
--Find the top three station names for both the member and casual riders
WITH station_counts AS 
  (
  SELECT member_casual, start_station_name, COUNT(ride_id) as ride_count
  FROM alldatacleaned_trip_data
  WHERE start_station_name IS NOT NULL
  GROUP BY member_casual, start_station_name
  ),
rank_stations AS 
  (
  SELECT member_casual, start_station_name, ride_count,RANK() OVER(PARTITION BY member_casual ORDER BY ride_count DESC) AS station_rank
  FROM station_counts
  )
SELECT member_casual, start_station_name, ride_count
FROM rank_stations
WHERE station_rank <=3
ORDER BY member_casual DESC, station_rank;
```

|member_casual|start_station_name|ride_count|
|---------------------|--------------------------|---------------|
|member |Kingsbury St & Kinzie St | 28997|
|member |Clinton St & Washington Blvd |28069 |
|member | Clinton St & Madison St | 24705|
|casual | Streeter Dr & Grand Ave | 50135|
|casual |DuSable Lake Shore Dr & Monroe St| 33533|
|casual |Michigan Ave & Oak St |24842 |

Members take significantly more rides than casual users, reflecting their frequent and consistent use of the service. They primarily prefer classic bikes, highlighting a focus on practical and reliable trips. Casual users, with fewer rides overall, show a balanced preference for classic bikes, electric bikes, and scooters, favoring electric options for leisure and occasional use.

```sql
--Count the different ride types and percentage of distribution for both the member and casual riders
WITH total_counts AS (
    SELECT COUNT(ride_id) AS total_ride_count
    FROM coursera-sql2024.cyclistic_data.alldatacleaned_trip_data
)
SELECT 
    member_casual,
    rideable_type,
    COUNT(ride_id) AS ride_count,
    COUNT(ride_id) / (SELECT total_ride_count FROM total_counts) * 100 AS ride_percentage
FROM coursera-sql2024.cyclistic_data.alldatacleaned_trip_data
GROUP BY member_casual,
    rideable_type
ORDER BY member_casual DESC, rideable_type;
```

|member_casual| rideable_type| ride_count| ride_percentage |
|---------------------|--------------------|----------------|--------|
|member| classic_bike| 1797556| 33.76% |
|member| electric_bike| 1582742| 29.73.% |
|member| electric_scooter| 44677| 0.84%|
|casual| classic_bike| 975274| 18.32% |
|casual| electric_bike| 869542| 16.33% |
|casual| electric_scooter| 54487| 1.02 %|

![](https://github.com/marvin-gomez/cyclistic_case_study/blob/a1843e5bd56a0455ac3abb4b4a955ca9987c791b/Data%20visuals/pie_ridetype.jpg)

Members take more rides overall, reflecting frequent, short trips, while casual riders average longer durations per ride, suggesting a preference for extended leisure or exploratory use. 

```sql
--Find the utilization count for members and casual riders across different ride lengths 
SELECT member_casual,
  COUNT(ride_length_minutes) AS total_rides,
  COUNT(CASE WHEN ride_length_minutes <= 30 THEN ride_length_minutes END) AS halfanhour_orless, 
  COUNT(CASE WHEN ride_length_minutes > 30 AND ride_length_minutes <= 60 THEN ride_length_minutes END) AS between_1hour_and_30min, 
  COUNT(CASE WHEN ride_length_minutes > 60 AND ride_length_minutes <= 120 THEN ride_length_minutes END) AS between_1hour_and_2hr,
  COUNT(CASE WHEN ride_length_minutes > 120 THEN ride_length_minutes END) AS greater_than_2hr,
FROM alldatacleaned_trip_data
GROUP BY member_casual
ORDER BY member_casual DESC;
```

|total_rides| halfanhour_orless| between_1hour_and_30min| between_1hour_and_2hr| greater_than_2hr |
|---------|--------|----------|---------------|-----------|
|member| 3424975| 3239353| 164823| 14933| 5866|
|casual| 1899303| 1565828| 218060| 87836| 27579|

Users ride more frequently on weekends and during the summer months, driven by favorable weather and leisure activities. Casual riders see a significant increase on weekends, likely due to recreational trips and tourism. Members also show higher usage during these times, though their riding patterns remain more consistent, balancing leisure and utility.

![](https://github.com/mgomez024/cyclistic_case_study/blob/main/Data%20visuals/bar_monthly.jpg)

![](https://github.com/mgomez024/cyclistic_case_study/blob/main/Data%20visuals/bar_daily.jpg)

Users ride more frequently on weekends and during the summer months, driven by favorable weather and leisure activities. Casual riders see a significant increase on weekends, likely due to recreational trips and tourism. Members also show higher usage during these times, though their riding patterns remain more consistent, balancing leisure and utility.

**Step 6: Act**

Based on the findings from my analysis, I would like to share my insights and provide recommendations Cyclistic’s marketing strategies to convert casual riders to annual members:
1.	Promote Membership Benefits - Launch targeted campaigns emphasizing the cost savings, convenience, and exclusive perks of annual memberships, particularly for frequent casual riders.
2.	Focus on Peak Usage Times - Align marketing and operational resources with peak times, such as weekends and summer months. Offer seasonal promotions or weekend-specific membership deals to capture the interest of casual users during high activity periods.
3.	Enhance User Experience for Casual Riders - Simplify the process of upgrading to an annual membership through the app or website. Provide real-time comparisons showing the cost-effectiveness of memberships after multiple casual rides to encourage conversion.




