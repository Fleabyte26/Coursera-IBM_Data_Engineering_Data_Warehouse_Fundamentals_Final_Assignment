# Coursera-IBM_Data_Engineering_Data_Warehouse_Fundamentals_Final_Assignment

This lab is poorly maintained, some of the CSV files columns do not match the lab instructions, the download instructions for the CSV are confusing, and a work around is to manually create the tables row by row.

I will put all the screen shots in this git to help students use them as references

Solid Waste Management Data Warehouse Lab – Step-by-Step Tutorial

Goal: Build a data warehouse in PostgreSQL, load CSV data, run aggregation queries, and create a materialized view.

Step 1: Design Dimension Table – MyDimDate

Columns:

dateid
fulldate
day
month
year
weekday
monthname
weekdayname
quarter

Screenshot: 1-MyDimDate.jpg

Step 2: Design Dimension Table – MyDimWaste

Columns:

wasteid
wastename
category

Screenshot: 2-MyDimWaste.jpg

Step 3: Design Dimension Table – MyDimZone

Columns:

zoneid
city
stationname

Screenshot: 3-MyDimZone.jpg

Step 4: Design Fact Table – MyFactTrips

Columns:

tripid
dateid
truckid
stationid
wasteamount

Screenshot: 4-MyFactTrips.jpg

Step 5–8: Create Tables in PostgreSQL (pgAdmin)

Example – DimDate (Step 5):

CREATE TABLE dimdate (
    dateid INT PRIMARY KEY,
    fulldate DATE,
    day INT,
    month INT,
    year INT,
    weekday INT,
    monthname VARCHAR(20),
    weekdayname VARCHAR(20),
    quarter INT
);

Repeat similarly for dimtruck, dimstation, and facttrips (steps 6–8).

Screenshots: 5-MyDimDate.jpg, 6-MyDimWaste.jpg, 7-MyDimZone.jpg, 8-MyFactTrips.jpg

Step 9–12: Load Data into Tables (FinalProject database)

DimDate Example:

INSERT INTO dimdate (dateid, fulldate, day, month, year, weekday, monthname, weekdayname, quarter) VALUES
(1,'2026-02-22',22,2,2026,1,'February','Monday',1),
(2,'2026-02-23',23,2,2026,2,'February','Tuesday',1),
(3,'2026-02-24',24,2,2026,3,'February','Wednesday',1),
(4,'2026-02-25',25,2,2026,4,'February','Thursday',1),
(5,'2026-02-26',26,2,2026,5,'February','Friday',1);

DimTruck Example:

INSERT INTO dimtruck (truckid, truckname, trucktype) VALUES
(1,'Truck A','Small'),
(2,'Truck B','Medium'),
(3,'Truck C','Large'),
(4,'Truck D','Small'),
(5,'Truck E','Medium');

DimStation Example:

INSERT INTO dimstation (stationid, stationname, city) VALUES
(1,'Station X','São Paulo'),
(2,'Station Y','Rio de Janeiro'),
(3,'Station Z','Salvador'),
(4,'Station W','Belo Horizonte'),
(5,'Station V','Curitiba');

FactTrips Example:

INSERT INTO facttrips (tripid, dateid, truckid, stationid, wasteamount) VALUES
(1,1,1,1,500),
(2,2,2,2,750),
(3,3,3,3,600),
(4,4,4,4,450),
(5,5,5,5,800);

Screenshots: 9-DimDate.jpg, 10-DimTruck.jpg, 11-DimStation.jpg, 12-FactTrips.jpg

Step 13: Grouping Sets Query
SELECT 
    stationid,
    trucktype,
    SUM(wasteamount) AS total_waste
FROM facttrips
GROUP BY GROUPING SETS ((stationid, trucktype), ());

Mock Output:

 stationid | trucktype | total_waste
-----------+-----------+------------
         1 | Small     | 500
         2 | Medium    | 750
         3 | Large     | 600
         4 | Small     | 450
         5 | Medium    | 800
    null   | null      | 3100

Screenshot: 13-groupingsets.jpg

Step 14: Rollup Query
SELECT 
    year,
    city,
    stationid,
    SUM(wasteamount) AS total_waste
FROM facttrips f
JOIN dimdate d ON f.dateid = d.dateid
JOIN dimstation s ON f.stationid = s.stationid
GROUP BY ROLLUP (year, city, stationid)
ORDER BY year, city, stationid;

Mock Output:

 year | city           | stationid | total_waste
------+----------------+-----------+-------------
 2026 | Belo Horizonte | 4         | 450
 2026 | Belo Horizonte | null      | 450
 2026 | Curitiba       | 5         | 800
 ...
 2026 | null           | null      | 3100

Screenshot: 14-rollup.jpg

Step 15: Cube Query
SELECT 
    year,
    city,
    stationid,
    AVG(wasteamount) AS avg_waste
FROM facttrips f
JOIN dimdate d ON f.dateid = d.dateid
JOIN dimstation s ON f.stationid = s.stationid
GROUP BY CUBE (year, city, stationid)
ORDER BY year, city, stationid;

Mock Output:

 year | city           | stationid | avg_waste
------+----------------+-----------+-----------
 2026 | Belo Horizonte | 4         | 450
 2026 | Belo Horizonte | null      | 450
 2026 | Curitiba       | 5         | 800
 ...
 2026 | null           | null      | 620

Screenshot: 15-cube.jpg

Step 16: Materialized View
CREATE MATERIALIZED VIEW max_waste_stats AS
SELECT 
    city,
    stationid,
    trucktype,
    MAX(wasteamount) AS max_waste
FROM facttrips f
JOIN dimtruck t ON f.truckid = t.truckid
JOIN dimstation s ON f.stationid = s.stationid
GROUP BY city, stationid, trucktype;

Mock Output (if queried):

 city           | stationid | trucktype | max_waste
----------------+-----------+-----------+-----------
 São Paulo      | 1         | Small     | 500
 Rio de Janeiro | 2         | Medium    | 750
 Salvador       | 3         | Large     | 600
 Belo Horizonte | 4         | Small     | 450
 Curitiba       | 5         | Medium    | 800

Screenshot: 16-mv.jpg
