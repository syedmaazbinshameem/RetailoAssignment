# RetailoAssignment

-- Question 1 - Creating a Database Called Retailo

CREATE DATABASE IF NOT EXISTS Retailo;
USE Retailo;

-- Creating a Table using COlumns from Bookings tab

CREATE TABLE IF NOT EXISTS Bookings
(
	Level_1_Company varchar(255),
	Level_2_Area varchar(255),
	Opportunity_Owner varchar(255),
	Opportunity_Number int,
	Production_License_Delivered date DEFAULT NULL,
	Stage varchar(255),
	Type varchar(255),
	Billing_Type varchar(255),
	Financial_Services_Vertical int,
	Forbes_Global_2000 int,
	Account_ID varchar(255),
	Activity varchar(255),
	Bookings int,
	Fiscal_Period_Booked varchar(255),
	Fiscal_Year_Booked year,
	Created_Date date,
	First_Signed_Opportunity_Date date,
	Close_Date date,
	Invoice_Date date,
    PRIMARY KEY (Opportunity_Number)
);

-- Creating a Table using columns from Lockups tab

CREATE TABLE IF NOT EXISTS Lockups
(
	Opportunity_Owner varchar(255),
    Level_3_House varchar(255)
);

-- Using Area wise Booking values, created two columns (2014 and 2015)

USE retailo;
SELECT 
	Level_2_Area AS Area,
	SUM(IF(Fiscal_Year_Booked = 2014,Bookings,0)) AS '2014',
    SUM(IF(Fiscal_Year_Booked = 2015,Bookings,0)) AS '2015',
	((SUM(IF(Fiscal_Year_Booked = 2015,Bookings,0))/SUM(IF(Fiscal_Year_Booked = 2014,Bookings,0)))-1) AS 'YoY Growth'
From Bookings
GROUP BY Level_2_Area

-- Creating a Total Row at the end of the query result

UNION ALL
SELECT
	'Total',
	SUM(IF(Fiscal_Year_Booked = 2014,Bookings,0)) AS '2014',
    SUM(IF(Fiscal_Year_Booked = 2015,Bookings,0)) AS '2015',
    ((SUM(IF(Fiscal_Year_Booked = 2015,Bookings,0))/SUM(IF(Fiscal_Year_Booked = 2014,Bookings,0)))-1) AS 'YOY Growth'
FROM Bookings
ORDER BY
	CASE Area
		WHEN 'Braavos' THEN 1
        WHEN 'Dorne' THEN 2
        WHEN "King's Landing" THEN 3
        WHEN 'Meereen' THEN 4
        WHEN 'Old Valaria' THEN 5
        WHEN 'Qarth' THEN 6
        WHEN 'Volantis' THEN 7
        WHEN 'Westeros' THEN 8
        WHEN 'Winterfell' THEN 9
        WHEN 'Total' THEN 10
	
  
-- Question 2
-- Create a new column 'House' in the Bookings table
  
  ALTER TABLE Bookings
ADD COLUMN House varchar(50)

-- Inserting values into the House Column based on values from the Lockups table

UPDATE Bookings
SET House = Lockups.Level_3_House
WHERE Opportunity_Owner = Bookings.Opportunity_Owner;

-- Question 3
USE Retailo;

-- Calculating Totals for Each Quarter Q1-2015 to Q2-2016
SET @Total_Q1_2015 = (SELECT SUM(IF(Fiscal_Period_Booked = 'Q1-2015',Bookings,0)) FROM Bookings);
SET @Total_Q2_2015 = (SELECT SUM(IF(Fiscal_Period_Booked = 'Q2-2015',Bookings,0)) FROM Bookings);
SET @Total_Q3_2015 = (SELECT SUM(IF(Fiscal_Period_Booked = 'Q3-2015',Bookings,0)) FROM Bookings);
SET @Total_Q4_2015 = (SELECT SUM(IF(Fiscal_Period_Booked = 'Q4-2015',Bookings,0)) FROM Bookings);
SET @Total_Q1_2016 = (SELECT SUM(IF(Fiscal_Period_Booked = 'Q1-2016',Bookings,0)) FROM Bookings);
SET @Total_Q2_2016 = (SELECT SUM(IF(Fiscal_Period_Booked = 'Q2-2016',Bookings,0)) FROM Bookings);

-- Calculating Growth Rate for Q3-2015 and Q4-2015
SET @Growth_Rate_Q3_2015 = @Total_Q3_2015/@Total_Q2_2015;
SET @Growth_Rate_Q4_2015 = @Total_Q4_2015/@Total_Q3_2015;

-- Projecting Total Quarterly Bookings for Q3-2016 and Q4-2016 using Growth Rates for Q3-2015 and Q4-2015 Respectively
SET @Total_Q3_2016 = @Total_Q2_2016 * @Growth_Rate_Q3_2015;
SET @Total_Q4_2016 = @Total_Q3_2016 * @Growth_Rate_Q4_2015;

SELECT
	Level_2_Area AS Area,
    SUM(IF(Fiscal_Period_Booked = 'Q1-2015',Bookings,0)) AS 'Actual Q1-2015',
    SUM(IF(Fiscal_Period_Booked = 'Q2-2015',Bookings,0)) AS 'Actual Q2-2015',
    SUM(IF(Fiscal_Period_Booked = 'Q3-2015',Bookings,0)) AS 'Actual Q3-2015',
    SUM(IF(Fiscal_Period_Booked = 'Q4-2015',Bookings,0)) AS 'Actual Q4-2015',
    SUM(IF(Fiscal_Period_Booked = 'Q1-2016',Bookings,0)) AS 'Actual Q1-2016',
    SUM(IF(Fiscal_Period_Booked = 'Q2-2016',Bookings,0)) AS 'Actual Q2-2016',
    ROUND(((SUM(IF(Fiscal_Period_Booked = 'Q3-2015',Bookings,0))/@Total_Q3_2015) * @Total_Q3_2016),0) AS 'Forecast Q3-2016',
    ROUND(((SUM(IF(Fiscal_Period_Booked = 'Q4-2015',Bookings,0))/@Total_Q4_2015) * @Total_Q4_2016),0) AS 'Forecast Q4-2016'
FROM Bookings
GROUP BY Area

-- Adding a Total Row in the end
UNION
SELECT 
	'Total',
	ROUND(@Total_Q1_2015, 0),
    ROUND(@Total_Q2_2015, 0),
    ROUND(@Total_Q3_2015, 0),
    ROUND(@Total_Q4_2015, 0),
    ROUND(@Total_Q1_2016, 0),
    ROUND(@Total_Q2_2016, 0),
    ROUND(((SUM(IF(Fiscal_Period_Booked = 'Q3-2015',Bookings,0))/@Total_Q3_2015) * @Total_Q3_2016),0) AS 'Forecast Q3-2016',
    ROUND(((SUM(IF(Fiscal_Period_Booked = 'Q4-2015',Bookings,0))/@Total_Q4_2015) * @Total_Q4_2016),0) AS 'Forecast Q4-2016'
FROM Bookings


-- Question 4 - Queries the desired columns for specific Opportunity_Numbers, filtered through the WHERE clause

USE Retailo;
SELECT 
	Opportunity_Number,
    House,
    Bookings,
    Created_Date,
    First_Signed_Opportunity_Date,
    Close_Date,
    Billing_Type,
    Financial_Services_Vertical,
    Forbes_Global_2000
FROM Bookings
WHERE Opportunity_Number IN (38932,
288242,
121777,
155839,
984306,
610938,
684767,
157716,
287017,
815224,
869715,
366472,
101165,
11910,
253232,
285617,
741207,
173068,
635268,
638954,
545932,
533827,
437682,
674876,
527115,
26277,
523397,
205622,
43789,
394872,
62787,
325157,
312804,
582051,
294920,
438025,
113766,
919049,
88080,
399641,
774668,
237119,
975117,
662671,
442995,
842905,
160072,
100248,
427453,
877943,
401025,
91095
)


-- Below are the individual results obtained

-- Calculates Subscription of House Greyjoy
SELECT House, ROUND(SUM(Bookings)/1000000,0) AS 'Subscription Bookings ($ Mn)'
FROM Bookings
WHERE Billing_Type = 'Subscription' AND House = 'House Greyjoy'


-- Calculates Bookings of House Baratheon before 2/1/2016
SELECT House, COUNT(Bookings) AS 'House Baratheon Bookings before 2/1/2016'
FROM Bookings
WHERE Created_Date < '2016-2-1' AND House = 'House Baratheon'


-- Calculates House with Most distinct opportunities
SELECT
	House,
    COUNT(DISTINCT Opportunity_Number) AS 'Distinct Opportunities'
FROM Bookings
GROUP BY House
ORDER BY 'Distinct Opportunities'
LIMIT 1


-- Calculates House with least distinct opportunities
SELECT
	House,
    COUNT(DISTINCT Opportunity_Number) AS 'Distinct Opportunities'
FROM Bookings
GROUP BY House
ORDER BY (COUNT(DISTINCT Opportunity_Number))
LIMIT 1


-- Calculates largest unique opportunity size and its House
SELECT 
	House,
    MAX(Bookings) AS 'Largest Unique Opportunity Size'
FROM Bookings
GROUP BY House
ORDER BY MAX(Bookings) DESC
LIMIT 1


-- Question 5

-- Calculates House with highest bookings for H1-2016 by combining and results for first 2 quarters.
SELECT 
	House,
    (SUM(IF(Fiscal_Period_Booked = 'Q1-2016',Bookings,0)))+(SUM(IF(Fiscal_Period_Booked = 'Q2-2016',Bookings,0))) AS 'H1-2016' 
FROM Bookings
GROUP BY House
ORDER BY (SUM(IF(Fiscal_Period_Booked = 'Q1-2016',Bookings,0)))+(SUM(IF(Fiscal_Period_Booked = 'Q2-2016',Bookings,0))) DESC
LIMIT 1



-- Assuming Account_ID is the ID of Sales Representative, we are finding out the House with the highest bookings with the least number of sales reps.

SELECT
	House,
    COUNT(DISTINCT Account_ID),
    SUM(Bookings) AS 'Total Bookings',
    SUM(Bookings)/COUNT(DISTINCT Account_ID) AS 'Bookings per Account ID'
FROM Bookings
GROUP BY House
ORDER BY SUM(Bookings)/COUNT(DISTINCT Account_ID) DESC
LIMIT 1
