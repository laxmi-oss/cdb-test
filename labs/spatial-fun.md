
# Oracle Spatial  


## Steps ##

**Perform location-based queries**

1. Find the five customers closest to the warehouse whose warehouse ID is 'Southlake,TX'
   
     ````
    <copy>
     SELECT  c.customer_id,c.cust_last_name,c.GENDER
     FROM warehouses w,
     customers c
     WHERE w.WAREHOUSE_NAME = 'Southlake, TX'
     AND sdo_nn (c.cust_geo_location, w.wh_geo_location, 'sdo_num_res=5') = 'TRUE';  
   
      </copy>
      ````

     ![](./images/spatial_m1.PNG " ")


       **Notes on Query 1:**

       
      - The SDO-NN operator returns the SDO-NUM-RES value of the customers from the CUSTOMERS table who are closest to warehouse 3. The first argument to SDO-NN (c.cust-geo-location in the example above) is the column to search. The second argument to SDO-NN (w.wh-geo-location in the example above) is the location you want to find the neighbors nearest to. No assumptions should be made about the order of the returned results. For example, the first row returned is not guaranteed to be the customer closest to warehouse 3. If two or more customers are an equal distance from the warehouse, then either of the customers may be returned on subsequent calls to SDO_NN.
      - When using the SDO-NUM-RES parameter, no other constraints are used in the WHERE clause. SDO_NUM_RES takes only proximity into account. For example, if you added a criterion to the WHERE clause because you wanted the five closest female customers, and four of the five closest customers are male, the query above would return one row. This behavior is specific to the SDO-NUM-RES parameter, and its results may not be what you are looking for. You will learn how to find the five closest female customers in the discussion of query 3.




2. **Query 2:** Find the five customers closest to warehouse named 'Seattle, WA' and put the results in order of distance 


     ````
     <copy>
    
     SELECT c.customer_id,c.cust_last_name,c.GENDER,
     round( sdo_nn_distance (1), 2) distance_in_miles
     FROM warehouses w, customers c
     WHERE w.WAREHOUSE_NAME = 'Seattle, WA'
     AND sdo_nn
     (c.cust_geo_location, w.wh_geo_location, 'sdo_num_res=5  unit=mile', 1) = 'TRUE'
     ORDER BY distance_in_miles;

       </copy>
       ````



   **Notes on Query 2:**

    -	The SDO-NN-DISTANCE operator is an ancillary operator to the SDO_NN operator; it can only be used within the SDO-NN operator. 
    The argument for this operator is a number that matches the number specified as the last argument of SDO-NN; in this example it is 1. There is no hidden meaning to this argument, it is simply a tag. If SDO-NN-DISTANCE() is specified, you can order the results by distance and guarantee that the first row returned is the closest. If the data you are querying is stored as longitude and latitude, the default unit for SDO-NN-DISTANCE is meters.
    -	The SDO-NN operator also has a UNIT parameter that determines the unit of measure returned by  SDO-NN-DISTANCE.
    - The ORDER BY DISTANCE clause ensures that the distances are returned in order, 
      with the shortest   distance first.



3. **Query 3:** Find the five female customers closest to warehouse named 'Sussex, NJ', put the results in order of distance, and give the distance in miles

    ````
    <copy>
   SELECT c.customer_id,c.cust_last_name,c.GENDER,
   round( sdo_nn_distance(1), 2) distance_in_miles
   FROM warehouses w, customers c
   WHERE w.WAREHOUSE_NAME = 'Sussex, NJ'
   AND sdo_nn (c.cust_geo_location, w.wh_geo_location,
   'sdo_batch_size =5 unit=mile', 1) = 'TRUE'
   AND c.GENDER = 'F'
   AND rownum< 6
   ORDER BY distance_in_miles; 

    </copy>
     ````

   **Notes on Query 3:**

   - SDO_BATCH_SIZE is a tunable parameter that may affect your query's performance. SDO_NN internally calculates that number of distances at a time. The initial batch of rows returned may not satisfy the constraints in the WHERE clause, so the number of rows specified by SDO_BATCH_SIZE is continuously returned until all the constraints in the WHERE clause are satisfied. You should choose a SDO_BATCH_SIZE that initially returns the number of rows likely to satisfy the constraints in your WHERE clause.
   - The UNIT parameter used within the SDO_NN operator specifies the unit of measure of the SDO_NN_DISTANCE parameter. The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters.
   - c.gender = 'F' and rownum< 6 are the additional constraints in the WHERE clause. The rownum< 6 clause is necessary to limit the number of results returned to fewer than 6.
   - The ORDER BY DISTANCE_IN_MILES clause ensures that the distances are returned in order, with the shortest distance first and the distances measured in miles.


4. **Query 4:** Find all the customers within 100 miles of warehouse named 'Sussex, NJ'
   
      ````
      <copy>
     SELECT c.customer_id,c.cust_last_name,c.GENDER
     FROM        warehouses w,   customers c
     WHERE   w.WAREHOUSE_NAME = 'Sussex, NJ'
     AND  sdo_within_distance (c.cust_geo_location,w.wh_geo_location,
    'distance = 100 unit=MILE') = 'TRUE';
     
         </copy>
      ````


 **Notes on Query 4:** 
   -	The SDO-WITHIN-DISTANCE operator returns the customers from the customers table 
      that are within 100   miles of warehouse 3. 
     The first argument to SDO-WITHIN-DISTANCE (c.cust-geo-location in the example above) is the column to search. 
     The second argument to SDO-WITHIN-DISTANCE (w.wh-geo-location in the example above) is the location you want to determine the distances from. No assumptions should be made about the order of the returned results. For example, the first row returned is not guaranteed to be the customer closest to warehouse 3.
   -	The DISTANCE parameter used within the SDO-WITHIN-DISTANCE operator specifies the distance value; 
     in this example it is 100.
   -	The UNIT parameter used within the SDO-WITHIN-DISTANCE operator specifies the unit of measure 
      of the  DISTANCE parameter. 
     The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters; in this example, it is miles.



5. **Query 5:** Find all the customers within 100 miles of warehouse named 'Sussex, NJ', put the results in order of distance, and give the distance in miles.    
   
    ````
    <copy>
   SELECT  c.customer_id, c.cust_last_name,  c.GENDER,
   round( sdo_geom.sdo_distance (c.cust_geo_location,  w.wh_geo_location, .005, 'unit=MILE'), 2) distance_in_miles 
   FROM warehouses w, customers c
   WHERE sdo_within_distance (c.cust_geo_location,
   w.wh_geo_location,
   'distance = 100 unit=MILE') = 'TRUE'
   ORDER BY distance_in_miles;
    </copy>
     ````

     
 **Notes on Query 5:**

  - 	The SDO_GEOM.SDO-DISTANCE function computes the exact distance between the customer's location and warehouse 3. 
  -   The first argument to SDO-GEOM.SDO-DISTANCE (c.cust-geo-location in the example above) contains the customer's location  whose  distance from warehouse 3 is to be computed. 
  -   The second argument to SDO-WITHIN-DISTANCE (w.wh-geo-location in the example above) is the location of warehouse 3, whose distance from the customer's location is to be computed.
  -	The third argument to SDO-GEOM.SDO-DISTANCE (0.005) is the tolerance value. The tolerance is a round-off error value used by Oracle Spatial. The tolerance is in meters for longitude and latitude data. In this example, the tolerance is 5 mm.
  -	The UNIT parameter used within the SDO-GEOM.SDO-DISTANCE parameter specifies the unit of measure of the distance computed by the SDO-GEOM.SDO-DISTANCE function. The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters. In this example it is miles.
  -	The ORDER BY DISTANCE-IN-MILES clause ensures that the distances are returned in order, with the shortest distance first and the distances measured in miles.

See an issue?  Please open up a request [here](https://github.com/oracle/learning-library/issues).