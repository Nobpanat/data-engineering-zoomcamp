# Module 1 Homework: Docker & SQL

## Question 1. Understanding docker first run

Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

What's the version of pip in the image?

24.3.1
24.2.1
23.3.1
23.2.1

```
docker run -it python:3.12.8 bash

root@7ac6be835ec2:/# pip --version
```

**Answer** 24.3.1

## Question 2. Understanding Docker networking and docker-compose

Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "ny_taxi"
    ports:
      - "5433:5432"
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

**Answer** db:5432

## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:

Up to 1 mile
In between 1 (exclusive) and 3 miles (inclusive),
In between 3 (exclusive) and 7 miles (inclusive),
In between 7 (exclusive) and 10 miles (inclusive),
Over 10 miles

```sql
SELECT
	CASE
		WHEN trip_distance <= 1 THEN '1. <=1'
		WHEN trip_distance > 1 AND trip_distance <= 3 THEN '2. < RANGE <=3'
		WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3. < RANGE <=7'
		WHEN trip_distance > 7 AND trip_distance <= 10 THEN '4. < RANGE <=10'
		WHEN trip_distance > 10 THEN '5. > 10'
		ELSE 'OTHERS'
	END AS distance_ranges,
	COUNT(1) AS COUNTS
FROM public.green_taxi
WHERE lpep_pickup_datetime >= '2019-10-01' AND lpep_pickup_datetime < '2019-11-01'
GROUP BY distance_ranges
ORDER BY distance_ranges;
```

**Answers:** 104830; 198995; 109642; 27686; 35201;

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance.

```sql
SELECT trip_distance , CAST(lpep_pickup_datetime AS DATE) as pickup_day
FROM public.green_taxi
ORDER BY trip_distance DESC
LIMIT 1;
```

**Answers:** 2019-10-31

## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18?

Consider only lpep_pickup_datetime when filtering by date.

```sql
SELECT Z."Zone" , SUM(GT."total_amount") AS total_revenue
FROM public.green_taxi AS GT INNER JOIN public.zones AS Z ON (GT."PULocationID" = Z."LocationID")
WHERE CAST(GT."lpep_pickup_datetime" AS DATE) = '2019-10-18'
GROUP BY Z."Zone"
HAVING SUM(GT."total_amount") > 13000
ORDER BY total_revenue DESC
LIMIT 3;
```

**Answers:** East Harlem North; East Harlem South; Morningside Heights;

## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone named "East Harlem North" which was the drop off zone that had the largest tip?

Note: it's tip , not trip

```sql
SELECT z_drop."Zone" as zone_drop_name , gt."tip_amount"
FROM public.zones AS z_pickup INNER JOIN public.green_taxi AS gt ON (z_pickup."LocationID" = gt."PULocationID")
INNER JOIN public.zones AS z_drop ON (z_drop."LocationID" = gt."DOLocationID")
WHERE gt."lpep_pickup_datetime" >= '2019-10-01' AND gt."lpep_pickup_datetime" < '2019-11-01' AND z_pickup."Zone" = 'East Harlem North'
ORDER BY gt."tip_amount" DESC
LIMIT 1;
```

**Answers:** JFK Airport
