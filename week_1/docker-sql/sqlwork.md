

- How many taxi trips were totally made on January 15?
> SELECT COUNT(index)  
> FROM green_taxi_trips  
> WHERE lpep_pickup_datetime BETWEEN '2019-01-15 00:00:00' AND '2019-01-15 23:59:59'

- Which was the day with the largest trip distance Use the pick up time for your calculations.
> SELECT CAST(lpep_pickup_datetime as DATE)  
> FROM green_taxi_trips  
> ORDER BY trip_distance DESC   
> LIMIT 1

- In 2019-01-01 how many trips had 2 and 3 passengers?
> SELECT passenger_count, COUNT(index)  
> FROM green_taxi_trips  
> WHERE CAST(lpep_pickup_datetime as date) = '2019-01-01' AND > (passenger_count = '2' OR passenger_count = '3')  
> GROUP BY passenger_count  
- For the passengers picked up in the Astoria Zone which was the drop off zone that had the largest tip? We want the name of the zone, not the id.

> SELECT doz.Zone, green.tip_amount  
> FROM public.green_taxi AS green  
> LEFT JOIN public.zones AS pz ON green."PULocationID"=pz."LocationID"  
> LEFT JOIN public.zones AS dz ON green."DOLocationID"=dz."LocationID"  
> WHERE pz."Zone" LIKE '%Astoria%'  
> GROUP BY dz."Zone"  
> ORDER BY largest_tip DESC  
> LIMIT 1  
