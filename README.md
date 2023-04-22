# Hotel Revenue Analysis for Resort & City Hotel

Creating analytical dashboards involves data extraction, transformation, and processing. The first step is to identify the data sources. Once the sources have been identified, the data can be extracted from various schemas using SQL, which involves agreggating data via the use of joins and unions.

Next, the data is imported into Microsoft Power Query, a data transformation and processing tool. Here, various data cleaning and transformation tasks can be performed, such as removing duplicates, filtering data, or performing calculated fields like averages, revenue metrics and KPIs.

After cleaning and transforming the data, it is processed to create analytical dashboards. The design process involves creating user friendly graphnics and figures that is intuitive and summarizes data in a concise manner.

The data workflow is ELT (extract, load, transform). It is first extracted from the dataset and loaded directly into the Microsoft plaform - Power Query and Power BI. The transformation is then performed on the data within this target system. This means that the data is loaded into the target system before it is transformed.


## Import Data

## Querying

### 1. Combining tables ### 
We have 3 tables of transaction info with 2018 - 2020 figures and can use UNION to join the rows into one big tabble. All the column names have to be the same and the data type the same to avoid errors such as `Conversion failed when converting the nvarchar value 'NA' to data type tinyint`, 
` `
To fix issue, change the data type in the 2020 table to nvchar(50) which is the same as the other table: 
``` ALTER TABLE dbo.[2020]
ALTER COLUMN children NVARCHAR(50) 
```

Now run union: 
```
SELECT *
FROM dbo.[2018]

UNION

SELECT *
FROM dbo.[2019]

UNION

SELECT *
FROM dbo.[2020]

```
|hotel|is_canceled|lead_time|arrival_date_year|arrival_date_month|arrival_date_week_number|arrival_date_day_of_month|stays_in_weekend_nights|stays_in_week_nights|adults|children|babies|meal|country|market_segment|distribution_channel|is_repeated_guest|previous_cancellations|previous_bookings_not_canceled|reserved_room_type|assigned_room_type|booking_changes|deposit_type|agent|company|days_in_waiting_list|customer_type|adr|required_car_parking_spaces|total_of_special_requests|reservation_status|reservation_status_date|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|City Hotel|0|0|2018|August|31|1|0|1|2|0|0|BB|PRT|Direct|Direct|0|0|0|A|A|0|No Deposit|NULL|NULL|0|Transient|75|0|0|Check-Out|2018-08-02|
|City Hotel|0|0|2018|August|32|3|1|0|2|0|0|BB|PRT|Complementary|Direct|0|0|0|A|D|0|No Deposit|NULL|NULL|0|Transient|0|0|0|Check-Out|2018-08-04|
|City Hotel|0|0|2018|August|32|3|1|0|2|0|0|BB|PRT|Complementary|Direct|0|0|0|D|D|0|No Deposit|NULL|NULL|0|Transient|0|0|0|Check-Out|2018-08-04|


Create a temporary result set: 

```
WITH hotel
AS (
    SELECT *
    FROM dbo.[2018]
    
    UNION
    
    SELECT *
    FROM dbo.[2019]
    )
SELECT *
FROM hotel

```

### 2. Is the hotel revenue growing year on year? ### 

We have column 'adr' which is average daily revenue and columns for duration of stay. 

Revenue: 
``` 
WITH hotel AS (
        SELECT *
        FROM dbo.[2018]
        
        UNION
        
        SELECT *
        FROM dbo.[2019]
        
        UNION
        
        SELECT *
        FROM dbo.[2020]
        )
SELECT arrival_date_year
    , SUM((stays_in_week_nights + stays_in_weekend_nights) * adr) AS 'revenue'
FROM hotel
GROUP BY arrival_date_year


```


|arrival_date_year|revenue|
|---|---|
|2018|4885517.059999978|
|2019|20188409.400000054|
|2020|14284246.240000024|


### 3. Yearly revenue broken down by hotel and year? ### 

```
WITH hotel
AS (
    SELECT *
    FROM dbo.[2018]
    
    UNION
    
    SELECT *
    FROM dbo.[2019]
    
    UNION
    
    SELECT *
    FROM dbo.[2020]
    )
SELECT arrival_date_year
    , hotel
    , round(SUM((stays_in_week_nights + stays_in_weekend_nights) * adr), 2) AS 'revenue'
FROM hotel
GROUP BY arrival_date_year
    , hotel

```
|arrival_date_year|hotel|revenue|
|---|---|---|
|2018|City Hotel|1764667.57|
|2019|City Hotel|10755979.11|
|2020|City Hotel|8018122.43|
|2018|Resort Hotel|3120849.49|
|2019|Resort Hotel|9432430.29|
|2020|Resort Hotel|6266123.81|




### 4. Combine the marketing segment table and meal cost table for a more accurate indication of revenue.
###

```
WITH hotel
AS (
    SELECT *
    FROM dbo.[2018]
    
    UNION
    
    SELECT *
    FROM dbo.[2019]
    
    UNION
    
    SELECT *
    FROM dbo.[2020]
    )
SELECT * FROM hotel

LEFT JOIN dbo.market_segment
    ON hotel.[market_segment] = [dbo].[market_segment].market_segment

LEFT JOIN dbo.mealcost
    ON hotel.meal = dbo.mealcost.meal


```

### 5. Microsoft Power BI Access ###
You cannot use a personal email to log into power BI unless you are a developer or have a business address. Visit https://developer.microsoft.com/en-us/microsoft-365/dev-program to sign up for a developer access, which is used for login to PowerBI.

Once you have access and installed the Power BI desktop app in Windows: 

* Get Data > SQL Server Database
* Server/Database: (Azure database address/database)
* Data Connectivity Mode: Import (note: DirectQuery mode causes syntax errors with SQL)
* Advanced: SQL Statement 
    * paste the previous SQL statement into the text box

You will get a preview of the query and be able to import it into Power BI. A new Query table will be created, which you get data for the dashboard from.

### 6. Build a dashboard in Power BI ###

* Is revenue growing year on year?
* Should we increase the parking lot size?
* What trends do we see in the data

