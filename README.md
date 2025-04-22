# NY_Airbnb_2019_Analysis
This project analyzes the 2019 NY Airbnb data using postgreSQL


this analysis is focused on the business questions related to a client similar to a travel website that serves potential clients looking to rent an Airbnb in NY City.  

the questions the will be answered are the following

1.  What is the difference from the median price of each listing.


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



2.  What are the total number of listings in each neighbourhood by room type and the median price.

Select
	Neighbourhood_group,
	room_type,
	count(distinct(id)),
	PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS Median
	
from 
	listings
group by 
	listings.neighbourhood_group,
	listings.room_type
;


3.  What neighborhoods have the most and least average availability





4.  For each listings, what is the difference between the price and the average room_type price for each listing?


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










