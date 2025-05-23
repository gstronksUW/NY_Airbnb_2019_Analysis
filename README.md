# NY_Airbnb_2019_Analysis
This project analyzes the 2019 NY Airbnb data using postgreSQL

Background:
this analysis is focused on the business questions related to a client similar to a travel website that serves potential clients looking to rent an Airbnb in NY City.  The data base consists of data related to each listing across the various neighbourhoods and sub neighbourhoods in New York City.  For the purposes of this analysis, the questions answered will be focused on clients looking to rent in the city or Airbnb looking to promote various properties that are under utilized.  One caveat to the data is that after examining the data it appears that it is autogenerated data so the room rates may not reflect what would be expected on the actual travel websites.  For example, the room rates to as low as 10.00.  


The data:
The data focused on with this analysis is related to the following variables:

Location - neighbourhood_group 
Room type - Entire home/apt, Private room, or shared room
Price - cost per night of the property
Availability - the number of days the property is available during the year

These questions were selected as the most relevant to a potential client for their personal use or for Airbnb to use to market to potential clients. 


Method:
This analysis was conducted by uploading the data to postgresql and analyzing through the use of SQL.  Additional tools used include, excel, Tableau, and openAI (ChatGTP).  
The database was collected from Kaggle.com listed using the link: https://www.kaggle.com/datasets/dgomonov/new-york-city-airbnb-open-data?resource=download 
The data was cleaned and in this process a number of listings with zero price were found and omitted because these were most likely omissions in the data and would skew the overall outputs.  
The number of zero price locations were small enough where it was deeemed acceptable to use the data without these individual listings.  
Next an number of questions were created of which the answers would be of interest to potential clients.



Questions:
1. What is the average cost of each room type accross the city?
2. What are the total number of listings for each type of room?
3. What is the average price for each room type in each neighbourhood and how does that compare to the average price for each room type city wide?
4. What neighborhoods have the most and least average availability


Analysis:

the questions the will be answered are the following

1. What is the average cost of each room type accross the city?

To extract the data for this question, the data had to be filtered and rolled up by neighbourhood and then by room type.  For readability, the data was then pivoted by room type in order to compate the prices across each neighbourhood group.  This will show the relative affordability for each room type.


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


![image](https://github.com/user-attachments/assets/e00e44b0-573b-4d41-8df6-0020231ef2f1)


![image](https://github.com/user-attachments/assets/1768265a-4d53-44cb-ae20-a615d531f20b)



This gives the client an idea of how much each type of rental is averaging in each neighbourhood.  As the data shows, that the rental of an entire home ranges from 167.00 in the Bronx to 662.00 in Manhattan.  For private rooms, the price ranges from 73.00 per night in Staten Island to 386.00 in Manhattan.  Finally, the shared room rental ranges from 58.00 in Staten Island to 139.00 in Manhattan.  The results show that for all rental type, Manhattan will be the most expensive on average while either Staten Island or the Bronx will provide the cheapest options.  This is helpful in guiding the client to the possible neighborhoods that fit the price range for rentals if budget ranks high on the priority list.  



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


![image](https://github.com/user-attachments/assets/94987bd6-0e3c-45da-8a14-5cde3dfec1d4)


![image](https://github.com/user-attachments/assets/722bf047-9a3b-4710-9870-fa39f27ec197)


This result is more geared to the business side as they may want to track the inventory but a client may also want to know what kind of room has the most listings and which neighborhoods are more popular for short term rentals.  In  this case, once again different neighborhoods have the majority of each room type.  For full apartments, Manhattan has the largest inventory by almost 3,500 of the next neighborhood which is Brooklyn.  On the other hand Brooklyn is more popular for private room rentals where the inventory exceeds the next neighborhood (Manhattan) by almost 2,000.  Finally, shared room inventory is much lower overall as the neighbourhood with the highest inventory is Manhattan with only 480 with Brooklyn close behind with 413.   

3.  What is the average price for each room type in each neighbourhood and how does that compare to the average price for each room type city wide?


WITH avg_price AS (
    SELECT
        room_type,
        ROUND(AVG(price)::numeric, 2) AS avg_rental_price_overall  -- Overall average price for each room type
    FROM 
        listings
    GROUP BY 
        room_type
)

SELECT 
    l.neighbourhood_group,
    l.room_type,
    ROUND(AVG(l.price)::numeric, 2) AS avg_price,  -- Average price in each neighbourhood group, rounded to 2 decimal places
    ROUND(AVG(l.price - avg_price.avg_rental_price_overall)::numeric, 2) AS difference_from_avg  -- Difference from the overall average price
FROM 
    listings l
JOIN 
    avg_price ON avg_price.room_type = l.room_type  -- Join to get the overall avg price for room_type
GROUP BY 
    l.neighbourhood_group,
    l.room_type,
    avg_price.avg_rental_price_overall  -- Grouping by overall avg price for correct calculation
ORDER BY 
    l.neighbourhood_group, 
    l.room_type;

![image](https://github.com/user-attachments/assets/6cb762df-1a74-4ee3-b51d-5d5b142d9d65)


![image](https://github.com/user-attachments/assets/07e219a3-da71-4f96-b545-dbe335c7dc71)

A key metric for the rentals is how the individual price compares to the same rental type city wide.  The key metric is labeled as difference from avg and the only neighbourhood where each room type is above the city wide average is Manhattan.  The rest of the neigbhourhoods all have prices under the city wide average for each room type.    


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

![image](https://github.com/user-attachments/assets/f9285dc2-48a7-4885-9155-f45b8d34335e)



![image](https://github.com/user-attachments/assets/c2dc732d-24a4-4a12-9cec-072ca409b193)



The availability tends to differ based on the room type and location.  For example the availability for an entire home/apartment is between 180 and 190 days a year accross all neighbourhoods while the private room is at 180 to 185 days a year across all neighbourhoods except for Staten Island which has an average availability of 236 days per year.  The variance continues to increase with the shared room that ranges from 72 days in Staten Island to 195 days in Queens.  



Summary

There are three major factors that a potential client is going to take in to account when looking to rent from Airbnb whcih include Location, room type, and availability.  The data suggests that while Manhattan is central to alot of the attractions in the city, you will pay a premium for that location regardless of the type of rental.  Looking at the variance from the average price city wide, there are potential savings in price by looking in other neighbourhoods especially in the Bronx where the average price of an entire home is 84.00 under the average price for that room type city wide.  The other neighbourhoods offer similar discounts to a lesser degree in the other room typs as well.  Manhattan is the only neighbourhood with rental price above the city wide average cost for each room type which helps quantify that location premium.  




