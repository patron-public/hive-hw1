# hive-hw1
//IMPORT data from EXTERNAL tables

CREATE TABLE hw.airports 
STORED AS ORCFILE
AS 
SELECT * FROM hw.ext_airports;
--------------
CREATE TABLE hw.carriers 
STORED AS ORCFILE
AS 
SELECT * FROM hw.ext_carriers;
--------------
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.max.dynamic.partitions.pernode=1000;
set hive.exec.max.dynamic.partitions=100000;
set hive.exec.max.created.files=100000;

CREATE TABLE hw.flights (DayofMonth INT,DayOfWeek INT,DepTime INT,CRSDepTime INT,ArrTime INT,CRSArrTime INT,UniqueCarrier VARCHAR(7),FlightNum INT,TailNum VARCHAR(7),ActualElapsedTime INT,CRSElapsedTime INT,AirTime INT,ArrDelay INT,DepDelay INT,Origin VARCHAR(4),Dest VARCHAR(4),Distance INT,TaxiIn INT,TaxiOut INT,Cancelled INT,CancellationCode VARCHAR(1),Diverted INT,CarrierDelay INT,WeatherDelay INT,NASDelay INT,SecurityDelay INT,LateAircraftDelay INT)
PARTITIONED BY (Year INT, Month INT)
STORED AS ORCFILE;



insert overwrite table hw.flights partition (Year, Month)
select 
DayofMonth, DayOfWeek, DepTime, CRSDepTime, ArrTime, CRSArrTime, UniqueCarrier, FlightNum, TailNum, ActualElapsedTime,CRSElapsedTime,AirTime,ArrDelay,DepDelay,Origin,Dest,Distance,TaxiIn,TaxiOut,Cancelled,CancellationCode,Diverted,CarrierDelay,WeatherDelay,NASDelay,SecurityDelay,LateAircraftDelay, Year, Month
from hw.ext_flights;

#Count total number of flights per carrier in 2007 (Make screenshot #1.1 and 1.2)
SELECT UniqueCarrier carrier, COUNT(*) total_flights FROM hw.flights WHERE Year=2007 GROUP BY UniqueCarrier ORDER BY carrier;

USE hw;
SELECT c.carrier_name carrier, f.cnt total_flights FROM hw.carriers c 
RIGHT JOIN (SELECT UniqueCarrier, COUNT(*) as cnt FROM hw.flights GROUP BY UniqueCarrier) f
ON(f.UniqueCarrier=c.carrier_id)
ORDER BY carrier;

carrier	total_flights
--------------------	
AirTran Airways Corporation	263159
Alaska Airlines Inc.	160185
Aloha Airlines Inc.	46360
American Airlines Inc.	633857
American Eagle Airlines Inc.	540494
Atlantic Southeast Airlines	286234
Comair Inc.	233787
Continental Air Lines Inc.	323151
Delta Air Lines Inc.	475889
Expressjet Airlines Inc.	434773
Frontier Airlines Inc.	97760
Hawaiian Airlines Inc.	56175
JetBlue Airways	191450
Mesa Airlines Inc.	294362
Northwest Airlines Inc.	414526
Pinnacle Airlines Inc.	258851
Skywest Airlines Inc.	597882
Southwest Airlines Co.	1168871
US Airways Inc. (Merged with America West 9/05. Reporting for both starting 10/07.)	485447
United Air Lines Inc.	490002

#The total number of flights served in Jun 2007 by NYC (all airports, use join with Airportsdata)
SELECT a.airport_name airport, SUM(f.cnt) sum
FROM 

(SELECT Origin as airport, COUNT(*) as cnt FROM hw.flights WHERE (month = 6) GROUP BY Origin 
UNION
SELECT Dest as airport, COUNT(*) as cnt FROM hw.flights WHERE (month = 6) GROUP BY Dest) f

JOIN
(SELECT airport_id, airport_name FROM hw.airports WHERE city = "New York") a
ON (a.airport_id = f.airport)

GROUP BY a.airport_name;

airport	sum
-----------
John F Kennedy Intl	20979
LaGuardia	20301


#Find five most busy airports in US during Jun 01 - Aug 31. (Make screenshot #3.1 and 3.2)
SELECT a.airport_name airport, SUM(f.cnt) sum
FROM 

(SELECT Origin as airport, COUNT(*) as cnt FROM hw.flights WHERE (month > 5 AND month < 9) GROUP BY Origin 
UNION
SELECT Dest as airport, COUNT(*) as cnt FROM hw.flights WHERE (month > 5 AND month < 9) GROUP BY Dest) f

JOIN
(SELECT airport_id, airport_name FROM hw.airports) a
ON (a.airport_id = f.airport)

GROUP BY a.airport_name ORDER BY sum DESC LIMIT 5;

#Find the carrier who served the biggest number of flights (Make screenshot #4.1 and 4.2)
SELECT c.carrier_name as career, f.cnt as count FROM hw.carriers c 
JOIN (SELECT UniqueCarrier, COUNT(*) as cnt FROM hw.flights GROUP BY UniqueCarrier) f
ON(f.UniqueCarrier=c.carrier_id)
ORDER BY count DESC LIMIT 1;

career	count
----
Southwest Airlines Co.	1168871


