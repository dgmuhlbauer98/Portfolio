# End2End Project: Excel to Power BI

![excel-to-powerbi-animated-diagram](https://github.com/dgmuhlbauer98/Portfolio/blob/029337d2896663ec0c5786c680a1d504e4cfdd06/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/kaggle_to_powerbi.gif)





# Table of contents 

- [Problem Statement](#problem-statement)
- [Project Work](#project-work)
  - Data Gathering
  - Data Exploration in Excel
  - Load the Data in SQL Server
  - Clean the Data With SQL
  - Test the Data with SQL
  - Visualize the Data in PowerBI
- [Findings](#findings)
- [Recommendations](#recommendations)



# Problem Statement 

- What is the key pain point? 

The Head of Marketing wants to find out who the top YouTubers are in 2024 to decide on which YouTubers would be best to run marketing campaigns throughout the rest of the year.


- What is the ideal solution? 

To create a dashboard that provides insights into the top German YouTubers in 2024 that includes their 
- subscriber count
- total views
- total videos, and
- engagement metrics

This will help the marketing team make informed decisions about which YouTubers to collaborate with for their marketing campaigns.

- What is the desired recommendation?

Identify the accounts for the highest return on investment


# Project Work
## 1. Data Gathering
The data is sourced from Kaggle (an Excel extract), [see here to find it.](https://www.kaggle.com/datasets/bhavyadhingra00020/top-100-social-media-influencers-2024-countrywise?resource=download)

Additionally, the YouTube API was used to identify missing key metrics (total subscribers, total videos, total views):
```Python
import pandas as pd
from googleapiclient.discovery import build

df = pd.read_csv('youtube_data_germany.csv')
df['channel_name'] = df['NAME'].str.split(' @').str[0]

# Connect to the Youtube API Key
API_KEY = 'AIzaSyBrNrFeT4GQBlRh5kSO7fDXt8X-jdUEKAI'
youtube = build('youtube', 'v3', developerKey=API_KEY)

# Write function to get channel statistics
def get_channel_stats(channel_name):
    request = youtube.search().list(
        part='snippet',
        q=channel_name,
        type='channel'
    )
    response = request.execute()
    
    if response['items']:
        channel_id = response['items'][0]['id']['channelId']
        stats_request = youtube.channels().list(
            part='statistics',
            id=channel_id
        )
        stats_response = stats_request.execute()
        stats = stats_response['items'][0]['statistics']
        return {
            'total_subscribers': stats.get('subscriberCount', 'N/A'),
            'total_views': stats.get('viewCount', 'N/A'),
            'total_videos': stats.get('videoCount', 'N/A')
        }
    else:
        return {
            'total_subscribers': 'N/A',
            'total_views': 'N/A',
            'total_videos': 'N/A'
        }

# Apply the function to each row in the dataset to receive total subscribers, total views, and total videos
df[['total_subscribers', 'total_views', 'total_videos']] = df['channel_name'].apply(lambda x: pd.Series(get_channel_stats(x)))
```

## 2. Data Exploration in Excel
Initial Observations:

1. There are at least 4 columns that contain the data we need for this analysis, which signals we have everything we need from the file without needing to contact the client for any more data. 
2. The first column contains the channel ID with what appears to be channel IDS, which are separated by a @ symbol - we need to extract the channel names from this.
3. We have more data than we need, so some of these columns would need to be removed

## 3. Load the Data in SQL Server
```SQL
CREATE DATABASE youtube_db;

USE youtube_db;

-- Next, I imported the Excel as a flat file to the created database

```

## 4. Clean the Data with SQL

The aim is to refine our dataset to ensure it is structured and ready for analysis. 
The cleaned data should meet the following criteria and constraints:
- Only relevant columns should be retained.
- All data types should be appropriate for the contents of each column.
- No column should contain null values, indicating complete data for all records.

Below is a table outlining the constraints on our cleaned dataset:
| Property | Description |
| --- | --- |
| Number of Rows | 100 |
| Number of Columns | 4 |

And here is a tabular representation of the expected schema for the clean data:
| Column Name | Data Type | Nullable |
| --- | --- | --- |
| channel_name | VARCHAR | NO |
| total_subscribers | INTEGER | NO |
| total_views | INTEGER | NO |
| total_videos | INTEGER | NO |

### 4.1 Data Transformation
```sql
/*
# 1. Select the required columns
# 2. Extract the channel name from the 'NAME' column
*/

-- 1.
SELECT
    SUBSTRING(NAME, 1, CHARINDEX('@', NAME) -1) AS channel_name,  -- 2.
    total_subscribers,
    total_views,
    total_videos

FROM
    top_german_youtubers_2024
```
### 4.2 Create View
```sql
/*
# 1. Create a view to store the transformed data
# 2. Cast the extracted channel name as VARCHAR(100)
# 3. Select the required columns from the top_german_youtubers_2024 SQL table 
*/

-- 1.
CREATE VIEW view_german_youtubers_2024 AS

-- 2.
SELECT 
	CAST(SUBSTRING(NAME, 1, CHARINDEX('@', NAME) - 1) AS VARCHAR(100)) AS channel_name,
	total_subscribers,
	total_views,
	total_videos

-- 3.
FROM 
	top_german_youtubers_2024

```

## 5. Test the Data with SQL

Here are the data quality tests conducted:

### 5.1 Row count check
```sql
/*
# Count the total number of records (or rows) are in the SQL view
*/

SELECT 
	COUNT(*) as no_of_rows 
FROM 
	view_german_youtubers_2024;

```
![Row count check](https://github.com/dgmuhlbauer98/Portfolio/blob/49de2552b51e9fa0f41d670036183e0681601f25/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/row%20count%20check.png)



### 5.2 Column count check
```sql
/*
# Count the total number of columns (or fields) are in the SQL view
*/


SELECT 
	COUNT(*) as no_of_columns
FROM
    	INFORMATION_SCHEMA.COLUMNS
WHERE 
	TABLE_NAME = 'view_german_youtubers_2024';
```
![Column count check](https://github.com/dgmuhlbauer98/Portfolio/blob/49de2552b51e9fa0f41d670036183e0681601f25/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/column%20count%20check.png)



### 5.3 Data type check
```sql
/*
# Check the data types of each column from the view by checking the INFORMATION SCHEMA view
*/

-- 1.
SELECT 
	COLUMN_NAME,
	DATA_TYPE
FROM 
	INFORMATION_SCHEMA.COLUMNS
WHERE 
	TABLE_NAME = 'view_german_youtubers_2024';
```
![Data type check](https://github.com/dgmuhlbauer98/Portfolio/blob/49de2552b51e9fa0f41d670036183e0681601f25/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/Data%20Type%20Check.png)


### 5.4 Duplicate count check
```sql
/*
# 1. Check for duplicate rows in the view
# 2. Group by the channel name
# 3. Filter for groups with more than one row
*/

-- 1.
SELECT 
	channel_name,
	COUNT(*) as duplicate_count
FROM 
	view_german_youtubers_2024

-- 2.
GROUP BY
    	channel_name

-- 3.
HAVING
    	COUNT(*) > 1;
```
![Duplicate count check](https://github.com/dgmuhlbauer98/Portfolio/blob/49de2552b51e9fa0f41d670036183e0681601f25/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/duplicate%20count%20check.png)


# 6. Visualize the Data in PowerBI 
## 6.1 Dashboard components required 
Below are the questions that must be answered by the dashboard:

1. Who are the top 10 YouTubers with the most subscribers?
2. Which 3 channels have uploaded the most videos?
3. Which 3 channels have the most views?
4. Which 3 channels have the highest average views per video?
5. Which 3 channels have the highest views per subscriber ratio?
6. Which 3 channels have the highest subscriber engagement rate per video uploaded?



Some of the data visuals that may be appropriate in answering our questions include:

1. Table
2. Treemap
3. Scorecards
4. Horizontal bar chart 

This is outlined in the mockup below.

![Dashboard-Mockup](https://github.com/dgmuhlbauer98/Portfolio/blob/e0c7f37f54f0d70b01d354a070d6fc1806e77ed5/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/dashboard_mockup.png)


## 6.2 Finalized Dashboard
This shows the Top German Youtubers in 2024 so far. 

![Picture of Power BI Dashboard](https://github.com/dgmuhlbauer98/Portfolio/blob/89a460b545d98dff39af106acb0cf687ad2e3859/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/dashboard.png)

## 6.3 DAX Measures
In order to answer the questions above, we had to create DAX measures:
### 6.3.1 Total Subscribers (M)
```sql
Total Subscriber (M) = 
VAR million = 1000000
VAR sumOfSubscribers = SUM(view_german_youtubers_2024[total_subscribers])
VAR totalSubscribers = DIVIDE(sumOfSubscribers, million)

RETURN totalSubscribers

```

### 6.3.2 Total Views (B)
```sql
Total Views (B) = 
VAR billion = 1000000000
VAR sumOfTotalViews = SUM(view_german_youtubers_2024[total_views])
VAR totalViews = DIVIDE(sumOfTotalViews, billion)

RETURN totalViews
```

### 6.3.3 Total Videos
```sql
Total Videos = 
VAR totalVideos = SUM(view_german_youtubers_2024[total_videos])

RETURN totalVideos

```

### 6.3.4 Average Views Per Video (M)
```sql
Avg views per Video (M) = 
VAR sumOfTotalViews = SUM(view_german_youtubers_2024[total_views])
VAR sumofTotalVideos = SUM(view_german_youtubers_2024[total_videos])
VAR avgOfViewsPerVideo = DIVIDE(sumOfTotalViews, sumofTotalVideos, BLANK())
VAR finalAvgViewsPerVideo = DIVIDE(avgOfViewsPerVideo, 1000000, BLANK())

RETURN finalAvgViewsPerVideo

```


### 6.3.5. Subscriber Engagement Rate
```sql
Subscriber Engagement Rate = 
VAR sumOfTotalSubscribers = SUM(view_german_youtubers_2024[total_subscribers])
VAR sumOfTotalVideos = SUM(view_german_youtubers_2024[total_videos])
VAR SubscriberEngRate = DIVIDE(sumOfTotalSubscribers, sumOfTotalVideos, BLANK())

RETURN SubscriberEngRate

```


### 6.3.6 Views per subscriber
```sql
View per Subscriber = 
VAR sumOfTotalViews = SUM(view_german_youtubers_2024[total_views])
VAR sumOfTotalSubscribers = SUM(view_german_youtubers_2024[total_subscribers])
VAR ViewsPerSubscriber = DIVIDE(sumOfTotalViews, sumOfTotalSubscribers, BLANK())

RETURN ViewsPerSubscriber

```




# Findings

- What did we find?

For this analysis, we're going to focus on the questions below to get the information we need for our marketing client - 

Here are the key questions we need to answer for our marketing client: 
1. Who are the top 10 YouTubers with the most subscribers?
2. Which 3 channels have uploaded the most videos?
3. Which 3 channels have the most views?
4. Which 3 channels have the highest average views per video?
5. Which 3 channels have the highest views per subscriber ratio?
6. Which 3 channels have the highest subscriber engagement rate per video uploaded?


### 1. Who are the top 10 YouTubers with the most subscribers?

| Rank | Channel Name         	| Subscribers (M) |
|------|------------------------|-----------------|
| 1    | Kinder Spielzeug Kanal	| 25.80	          |
| 2    | Dhruv Rathee           | 20.10	          |
| 3    | HaerteTest	        | 19.60           |
| 4    | Ice Cream Rolls        | 12.30        	  |
| 5    | Pamela Reif	        | 9.90	          |
| 6    | German Spidey          | 8.62	          |
| 7    | freekickerz            | 8.58            |
| 8    | The Voice Kids         | 8.40            |
| 9    | Rammstein Official     | 8.09            |
| 10   | COLORS                 | 7.49            |


### 2. Which 3 channels have uploaded the most videos?

| Rank | Channel Name    | Videos Uploaded |
|------|-----------------|-----------------|
| 1    | DW Español      | 41,273          |
| 2    | DW News	 | 34,320          |
| 3    | PietSmiet       | 27,716          |



### 3. Which 3 channels have the most views?


| Rank | Channel Name 		| Total Views (B) |
|------|------------------------|-----------------|
| 1    | Kinder Spielzeug Kanal	| 13.21           |
| 2    | German Spidey          | 9.30	          |
| 3    | ArkivaShqip	        | 7.43            |


### 4. Which 3 channels have the highest average views per video?

| Rank | Channel Name 		| Averge Views per Video (M) |
|------|------------------------|----------------------------|
| 1    | Mois	       		| 130.19          	     |
| 2    | Rammstein Official	| 37.40                      |
| 3    | LEO ROJAS - official   | 28.55                      |


### 5. Which 3 channels have the highest views per subscriber ratio?

| Rank | Channel Name       | Views per Subscriber        |
|------|-----------------   |---------------------------- |
| 1    | ArkivaShqip        | 1,651.24                    |
| 2    | PietSmiet          | 1,453.91                    |
| 3    | marsgizmo          | 1,307.03                    |



### 6. Which 3 channels have the highest subscriber engagement rate per video uploaded?

| Rank | Channel Name    	| Subscriber Engagement Rate  |
|------|------------------------|---------------------------- |
| 1    | Mois            	| 424,000                     |
| 2    | LEO ROJAS - official   | 1101,944.44                 |
| 3    | TheFatRat	        | 54,661.02                   |


## Findings Analysis

For this analysis, we'll prioritize analysing the metrics that are important in generating the expected ROI for our marketing client, which are the YouTube channels wuth the most 

- subscribers
- total views
- videos uploaded

## 1. Youtubers with the most subscribers 

Campaign idea = product placement 

a. **Kinder Spielzeug Kanal** 
- Average views per video = 11.08 million
- Product cost = $3
- Potential units sold per video = 11.08 million x 2.5% conversion rate = 277,000 units sold
- Potential revenue per video = 277,000 x $3 = $831,000
- Campaign cost (one-time fee) = $100,000
- **Net profit = $831,000 - $100,000 = $731,000**

b. **Dhruv Rathee**

- Average views per video = 4.41 million
- Product cost = $3
- Potential units sold per video = 4.41 million x 2.5% conversion rate = 110,250 units sold
- Potential revenue per video = 110,250 x $3 = $330,750
- Campaign cost (one-time fee) = $100,000
- **Net profit = $330,750 - $100,000 = $230,750**

c. **HaerteTest**

- Average views per video = 2.15 million
- Product cost = $3
- Potential units sold per video = 2.15 million x 2.5% conversion rate = 53,750 units sold
- Potential revenue per video = 53,750 x $3 = $161,250
- Campaign cost (one-time fee) = $100,000
- **Net profit = $161,250 - $100,000 = $61,250**


Best option from category: Kinder Spielzeug Kanal 

```sql
/* 

# 1. Define variables 
# 2. Create a CTE that rounds the average views per video 
# 3. Select the column you need and create calculated columns from existing ones 
# 4. Filter results by Youtube channels
# 5. Sort results by net profits (from highest to lowest)

*/


-- 1.
DECLARE @conversionRate FLOAT = 0.025;			-- conversion rate @ 2.5%
DECLARE @productCost MONEY = 3.0;			-- Product cost @ $3
DECLARE @campaignCost MONEY = 100000.0;			-- Campaign cost @ $100,000

-- 2.
WITH ChannelData AS (
	SELECT
		channel_name,
		total_views,
		total_videos,
		ROUND((CAST(total_views as FLOAT) / total_videos), -4) AS rounded_avg_views_per_video
	FROM 
		youtube_db.dbo.view_german_youtubers_2024
)

-- 3.

SELECT 
	channel_name,
	rounded_avg_views_per_video,
	(rounded_avg_views_per_video * @conversionRate) AS potential_units_sold_per_video,
	(rounded_avg_views_per_video * @conversionRate * @productCost) AS potential_revenue_per_video,
	(rounded_avg_views_per_video * @conversionRate * @productCost) - @campaignCost AS net_profit
FROM 
	ChannelData

-- 4.
WHERE 
	channel_name IN ('Kinder Spielzeug Kanal', 'HaerteTest', 'Dhruv Rathee')

-- 5.
ORDER BY
	net_profit DESC;

```

![Most subsc](https://github.com/dgmuhlbauer98/Portfolio/blob/8a000fc12467325db672cedc9e331858aa2af602/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/most%20subs.png)

## 2. Youtubers with the most videos uploaded

Campaign idea = sponsored video series  

a. **DW Español**
- Average views per video = 60,000
- Product cost = $3
- Potential units sold per video = 60,000 x 2.5% conversion rate = 1,500 units sold
- Potential revenue per video = 1,500 x $3 = $4,500
- Campaign cost (5-videos @ $1,000 each) = $5,000
- **Net profit = $4,500 - $5,000 = -$500 (potential loss)**

b. **DW News**

- Average views per video = 70,000
- Product cost = $3
- Potential units sold per video = 70,000 x 2.5% conversion rate = 1,750 units sold
- Potential revenue per video = 1,750 x $3 = $5,250
- Campaign cost (5-videos @ $1,000 each) = $5,000
- **Net profit = $5,250 - $5,000 = $250 (profit)**

b. **PietSmiet**

- Average views per video = 130,000
- Product cost = $3
- Potential units sold per video = 130,000 x 2.5% conversion rate = 3,250 units sold
- Potential revenue per video = 3,250 x $3 = $9,750
- Campaign cost (5-videos @ $1,000 each) = $5,000
- **Net profit = $9,750 - $5,000 = $4,750 (profit)**


Best option from category: PietSmiet

```sql
/* 
# 1. Define variables
# 2. Create a CTE that rounds the average views per video
# 3. Select the columns you need and create calculated columns from existing ones
# 4. Filter results by YouTube channels
# 5. Sort results by net profits (from highest to lowest)
*/
-- 1.
DECLARE @conversionRate FLOAT = 0.025;          -- The conversion rate @ 2.5%
DECLARE @productCost FLOAT = 3.0;               -- The product cost @ $3
DECLARE @campaignCostPerVideo FLOAT = 1000.0;   -- The campaign cost per video @ $1,000
DECLARE @numberOfVideos INT = 5;                -- The number of videos (5)


-- 2.
WITH ChannelData AS (
    SELECT
        channel_name,
        total_views,
        total_videos,
        ROUND((CAST(total_views AS FLOAT) / total_videos), -4) AS rounded_avg_views_per_video
    FROM
        youtube_db.dbo.view_german_youtubers_2024
)


-- 3.
SELECT
    channel_name,
    rounded_avg_views_per_video,
    (rounded_avg_views_per_video * @conversionRate) AS potential_units_sold_per_video,
    (rounded_avg_views_per_video * @conversionRate * @productCost) AS potential_revenue_per_video,
    ((rounded_avg_views_per_video * @conversionRate * @productCost) - (@campaignCostPerVideo * @numberOfVideos)) AS net_profit
FROM
    ChannelData


-- 4.
WHERE
    channel_name IN ('DW Español', 'DW News', 'PietSmiet')


-- 5.
ORDER BY
    net_profit DESC;
```

![most videos](https://github.com/dgmuhlbauer98/Portfolio/blob/60deb835c83700b250a0167c6f1f4b9445266cfa/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/most%20videos.png)

### 3.  Youtubers with the most views 

Campaign idea = Influencer marketing 

a. **Kinder Spielzeug Kanal**

- Average views per video = 11.08 million
- Product cost = $3
- Potential units sold per video = 11.08 million x 2.5% conversion rate = 277,000 units sold
- Potential revenue per video = 277,000 x $3 = $831,000
- Campaign cost = $150,000
- **Net profit = $831,000 - $150,000 = $681,000**

b. **German Spidey**

- Average views per video = 6.93 million
- Product cost = $3
- Potential units sold per video = 6.93 million x 2.5% conversion rate = 173,250 units sold
- Potential revenue per video = 173,250 x $3 = $519,750
- Campaign cost = $150,000
- **Net profit = $519,750 - $150,000 = $369,750**

c. **ArkivaShqip**

- Average views per video = 840,000
- Product cost = $3
- Potential units sold per video = 840,000 x 2.5% conversion rate = 21,000 units sold
- Potential revenue per video = 21,000 x $3 = $63,000
- Campaign cost = $150,000
- **Net profit = $63,000 - $150,000 = -$87,000 (potential loss)**

Best option from category: Kinder Spielzeug Kanal



#### SQL query 
```sql
/*
# 1. Define variables
# 2. Create a CTE that rounds the average views per video
# 3. Select the columns you need and create calculated columns from existing ones
# 4. Filter results by YouTube channels
# 5. Sort results by net profits (from highest to lowest)
*/



-- 1.
DECLARE @conversionRate FLOAT = 0.025;       -- The conversion rate @ 2.5%
DECLARE @productCost MONEY = 3.0;            -- The product cost @ $3
DECLARE @campaignCost MONEY = 150000.0;      -- The campaign cost @ $150,000



-- 2.
WITH ChannelData AS (
    SELECT
        channel_name,
        total_views,
        total_videos,
        ROUND(CAST(total_views AS FLOAT) / total_videos, -4) AS avg_views_per_video
    FROM
        youtube_db.dbo.view_german_youtubers_2024
)


-- 3.
SELECT
    channel_name,
    avg_views_per_video,
    (avg_views_per_video * @conversionRate) AS potential_units_sold_per_video,
    (avg_views_per_video * @conversionRate * @productCost) AS potential_revenue_per_video,
    (avg_views_per_video * @conversionRate * @productCost) - @campaignCost AS net_profit
FROM
    ChannelData


-- 4.
WHERE
    channel_name IN ('Kinder Spielzeug Kanal', 'German Spidey', 'ArkivaShqip')


-- 5.
ORDER BY
    net_profit DESC;

```

#### Output

![Most views](https://github.com/dgmuhlbauer98/Portfolio/blob/f6f62d0ca0ffa01e90010a590514a0b82510b65b/SQL_PowerBI_Excel%3A%20E2E%20Project%202024%20Top%20German%20Youtubers/0.%20Images/most%20views.png)



# Recommendations

- What did we learn?

We discovered that 


1. Kinder Spielzeug Kanal, Dhruv Rathee and HaerteTest are the channnels with the most subscribers in Germany
2. DW Español, DW News and PietSmiet are the channels with the most videos uploaded
3. Kinder Spielzeug Kanal, German Spidey and ArkivaShqip are the channels with the most views
4. Channels that are focused on entertainment and news are most destined to generate subscriber and views. 


- What do you recommend based on the insights gathered? 
  
1. Kinder Spielzeug Kanal is the best channel to collaborate with as they have the most subscribers and highest total views.
2. Although the news channels produce a high amount of videos, I don't believe that these are good channels to collaborate with as they have very minimal subscriber engagement ratings.
3. Kinder Spielzeug Kanal is the best channel to collaborate with if we want to maximize reach. Hwoever, Dhruv Rathee and German Spidey mighe be better long-term options due to their subscriber engagement scores and high click rates.
4. The top 3 channels to form collaborations with are Kinder Spielzeug Kanal, Dhruv Rathee, and German Spidey due to their reach and high engagement rate/click rate.


- What ROI do we expect if we take this course of action?

1. If we go with a product placement campaign with Dhruv Rathee, this could  generate the client approximately $230,7500 per video.
2. An influencer marketing contract with German Spidey can see the client generate a net profit of $369,750.
3. If we go with a product placement campaign with Kinder Spielzeug Kanal, this could  generate the client approximately $731,000 per video. If we advance with an influencer marketing campaign deal instead, this would make the client a one-off net profit of $681,000.


- What course of action should we take and why?

Based on the analysis, we believe the best channel to start a ,,long-term partnership with is the Kinder Spielzeug Kanal. Given the volume of subscribers as well as the average views per video, the reach of the channel will enable to marketing campaign to be very successful. 

It would be beneficial to know more about the desired target customers as Dhruv Rathee might be a better fit due to its subscriber engagement rating. However, the susbscriber base must fit the desired target market to yield positive results. German Spidey is interesting due to its clicks per video but again it cannot match with Kinder Spielzeug Kanal view volume.

- What steps do we take to implement the recommended decisions effectively?


1. Reach out to the teams behind each of these channels, starting with Kinder Spielzeug Kanal
2. Negotiate contracts within the budgets allocated to each marketing campaign
3. Kick off the campaigns and track each of their performances against the KPIs
4. Review how the campaigns have gone, gather insights and optimize based on feedback from converted customers and each channel's audiences 



