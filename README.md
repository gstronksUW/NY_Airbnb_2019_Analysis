# NY_Airbnb_2019_Analysis
This project analyzes the 2019 NY Airbnb data using postgreSQL

Background:
this analysis is focused on the business questions related to a client similar to a travel website that serves potential clients looking to rent an Airbnb in NY City.  The data base consists of data related to each listing across the various neighbourhoods and sub neighbourhoods in New York City.  For the purposes of this analysis, the questions answered will be focused on clients looking to rent in the city or Airbnb looking to promote various properties that are under utilized.  


The data:
The data focused on with this analysis is related to the following variables:

Location - neighbourhood_group 
room type - Entire home/apt, Private room, or shared room
price - cost per night of the property
Availability - the number of days the property is available during the year

These were selected as the most relevant to a potential client for their personal use or for Airbnb to use to market to potential clients.


Method:
This analysis was conducted by uploading the data to postgresql and analyzing through the use of SQL.  Additional tools used include, excel, Tableau, and openAI.  
The database was collected from Kaggle.com listed using the link: https://www.kaggle.com/datasets/dgomonov/new-york-city-airbnb-open-data?resource=download 
The data was cleaned and in this process a number of listings with zero price were found and omitted because these were most likely omissions in the data and would skew the overall outputs.  
The number of zero price locations were small enough where it was deeemed acceptable to use the data without these individual listings.



Questions:
1. What is the average cost of each room type accross the city?
2. What are the total number of listings for each type of room?
3. What is the difference from the median price of each listing.
4. What neighborhoods have the most and least average availability
5. For each listings, what is the difference between the price and the average room_type price for each listing?
6. What is the most expensive neiborhood to rent each type of room? least expensive?


Analysis:

the questions the will be answered are the following

1. What is the average cost of each room type accross the city?

select
	neighbourhood_group,
	round(avg(distinct case when room_type='Entire home/apt' then price end),2) as Entire_home_ct,
	round(avg(distinct case when room_type='Private room' then price end),2) as Private_room_ct,
	round(avg(distinct case when room_type='Shared room' then price end),2) as Shared_room_ct
from 
	listings
Group by 
	neighbourhood_group
;


![image](https://github.com/user-attachments/assets/acf3013b-8fac-40d8-89c5-efcf7e00a9d7)


2. What are the total number of listings for each type of room?

select
	neighbourhood_group,
 	count(distinct case when room_type='Entire home/apt' then id end) as Entire_home_ct,
	count(distinct case when room_type='Private room' then id end) as Private_room_ct,
	count(distinct case when room_type='Shared room' then id end) as Shared_room_ct
from 
	listings
Group by 
	neighbourhood_group
;

![image](https://github.com/user-attachments/assets/86cb8d17-0462-423f-a722-c7238e7f9a0b)



3.  What is the difference from the median price of each listing.


WITH med_price AS (
    SELECT 
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS Median
    FROM 
        listings
)
-- Distance from median
SELECT 
	host_id, 
	neighbourhood,
	price,
	neighbourhood_group,
    (l.price - m.Median) AS difference_from_median
FROM 
    listings l
JOIN 
    med_price m ON true
Where
	price <> 0
	
Order by 
	price 
	;



4.  What neighborhoods have the most and least average availability

select 
	neighbourhood_group,
	--room_type,
	--round(avg(listings.availability_365),2) as Avg_availability,
	round(avg(distinct case when room_type='Entire home/apt' then listings.availability_365 end),2) as Entire_home_availability,
	round(avg(distinct case when room_type='Private room' then listings.availability_365 end),2) as Private_room_availability,
	round(avg(distinct case when room_type='Shared room' then listings.availability_365 end),2) as Shared_room_availability
from 
	listings
group by
	neighbourhood_group
;




6.  For each listings, what is the difference between the price and the average room_type price for each listing?


With avg_price_loc as (

select
	room_type,
	round(avg(price),2) avg_price
from 
	listings
group by
	room_type 
)

select 
	Neighbourhood_group,
	id,
	name,
	listings.room_type,
	Price,
	avg_price,
	price-avg_price as price_difference
	
from 
	listings
left join 
avg_price_loc on avg_price_loc.room_type = listings.room_type

order by 
	neighbourhood_group,
	room_type,
	price_difference
;
	


5.  What is the most expensive neiborhood to rent each type of room? least expensive?

SELECT 
	listings.neighbourhood_group,
	room_type,
	ROUND(AVG(price), 2) AS avg_price
FROM 
	listings
GROUP BY 
	neighbourhood_group,
	room_type
ORDER BY 
    neighbourhood_group DESC;


Summary/Insights:







