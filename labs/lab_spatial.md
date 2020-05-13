
# Oracle Spatial  

**Create tables and spatial metadata:**
 

## Steps ##

1. Login to the PDB:
   
   ````
    <copy>
   ps -ef|grep|pmon
   . oraenv
   sqlplus ' / as sysdba'
   alter session set container=SPAGRAPDB;
    </copy>
    ````

2. Check to see who you are connected as. At any point in the lab you can run this script to see who or where you are connected.  
    ````
    <copy>
    select
      'DB Name: '  ||Sys_Context('Userenv', 'DB_Name')||
      ' / CDB?: '     ||case
        when Sys_Context('Userenv', 'CDB_Name') is not null then 'YES'
          else  'NO'
          end||
      ' / Auth-ID: '   ||Sys_Context('Userenv', 'Authenticated_Identity')||
      ' / Sessn-User: '||Sys_Context('Userenv', 'Session_User')||
      ' / Container: ' ||Nvl(Sys_Context('Userenv', 'Con_Name'), 'n/a')
      "Who am I?"
      from Dual
      /
      </copy>
    ````

    
3. Create a  user **app_test**

    ````
    <copy>
    create user app_test identified by app container=current;
    </copy>
    ````
    
4. Grant dba privilage to **app_test** user.

    ````
    <copy>
    grant dba to app_test;
    </copy>
    ````
   
5. Connect Container **SPAGRAPDB** as user **app_test**

    ````
    <copy>
    sqlplus app_test/app@SPAGRAPDB
    </copy>
    ````
   
6. Create  table **CUSTOMERS**  and **WAREHOUSES** 

    ````
    <copy>
    CREATE TABLE CUSTOMERS
    ( 
    CUSTOMER_ID NUMBER(6, 0),
    CUST_FIRST_NAME VARCHAR2(20 CHAR),
    CUST_LAST_NAME VARCHAR2(20 CHAR), 
    GENDER VARCHAR2(1 CHAR), 
    CUST_GEO_LOCATION SDO_GEOMETRY,
    ACCOUNT_MGR_ID NUMBER(6, 0)
    );
  
    CREATE TABLE WAREHOUSES
    (
    WAREHOUSE_ID    NUMBER(3,0), 
    WAREHOUSE_NAME        VARCHAR2(35 CHAR), 
    LOCATION_ID   NUMBER(4,0), 
    WH_GEO_LOCATION       SDO_GEOMETRY
    );
      </copy>

    ````

  
7.    Next we add Spatial metadata for the CUSTOMERS and WAREHOUSES tables
      to the **USER-SDO-GEOM-METADATA** view. Each **SDO-GEOMETRY** column is registered
      with a row in   **USER-SDO-GEOM-METADATA**.

    ````
    <copy>
     EXECUTE SDO_UTIL.INSERT_SDO_GEOM_METADATA (sys_context('userenv','current_user'), -
    'CUSTOMERS', 'CUST_GEO_LOCATION', -  SDO_DIM_ARRAY(SDO_DIM_ELEMENT('X',-180, 180, 0.05), - SDO_DIM_ELEMENT('Y', -90, 90, 0.05)),-  4326);

     EXECUTE SDO_UTIL.INSERT_SDO_GEOM_METADATA (sys_context('userenv','current_user'), -
     'WAREHOUSES', 'WH_GEO_LOCATION', - SDO_DIM_ARRAY(SDO_DIM_ELEMENT('X',-180, 180, 0.05), - SDO_DIM_ELEMENT('Y', -90, 90, 0.05)),-  4326);
      </copy>
    
       ````

     
     **Here is a description of the items that were entered:**

     -	TABLE-NAME: Name of the table which contains the spatial data.
     -	COLUMN-NAME: Name of the SDO-GEOMETRY column which stores the spatial data.
     -	 MDSYS.SDO-DIM-ARRAY: Constructor which holds the MDSYS.SDO-DIM-ELEMENT object,which in turn stores the extents of the spatial data  in each dimension (-180.0, 180.0), and a tolerance value (0.05). The tolerance is a round-off error value used by Oracle Spatial, and is in meters for longitude and latitude data. In this example, the tolerance is 5 mm.
     -	4326: Spatial reference system id (SRID): a foreign key to an Oracle dictionary table  (MDSYS.CS-SRS) tha  contains all the     supported coordinate systems. It is important to associate your customer's location to a coordinate system. In this example, 4326    corresponds to "Longitude / Latitude (WGS 84).".
 
**Load data**


  First we load CUSTOMERS by copying from the table oeuser.CUSTOMERS

   **Note that we are using two spatial functions in this -**
   -  we use sdo_cs.transform() to convert to our desired coordinate system SRID of 4326.
   -  we use sdo-geom.validate-geometry() to insert only valid geometries. 




 1. Check to see who you are connected as. At any point in the lab you can run this script to see who or where you are connected.  

    ````
    <copy>
    select
      'DB Name: '  ||Sys_Context('Userenv', 'DB_Name')||
      ' / CDB?: '     ||case
        when Sys_Context('Userenv', 'CDB_Name') is not null then 'YES'
          else  'NO'
          end||
      ' / Auth-ID: '   ||Sys_Context('Userenv', 'Authenticated_Identity')||
      ' / Sessn-User: '||Sys_Context('Userenv', 'Session_User')||
      ' / Container: ' ||Nvl(Sys_Context('Userenv', 'Con_Name'), 'n/a')
      "Who am I?"
      from Dual
      /
      </copy>
    ````

    
 2. Insert Data into **CUSTOMERS** Table

    ````
    <copy>
    INSERT INTO CUSTOMERS
    SELECT CUSTOMER_ID, CUST_FIRST_NAME, CUST_LAST_NAME , sdo_cs.transform(CUST_GEO_LOCATION,4326), ACCOUNT_MGR_ID
    FROM oeuser.customers WHERE sdo_geom.validate_geometry(CUST_GEO_LOCATION,0.05)='TRUE';
    
    commit;
    </copy>
    ````
    
 3. Manually load **warehouses** using the **SDO-GEOMETRY** constructor.

    ````
    <copy>
    INSERT INTO WAREHOUSES values (1,'Southlake, TX',1400,SDO_GEOMETRY(2001, 4326, MDSYS.SDO_POINT_TYPE(-103.00195, 36.500374, NULL), NULL, NULL));

    INSERT INTO WAREHOUSES values (2,'San Francisco, CA',1500,SDO_GEOMETRY(2001, 4326, MDSYS.SDO_POINT_TYPE(-124.21014, 41.998016, NULL), NULL, NULL));

    INSERT INTO WAREHOUSES values (3,'Sussex, NJ',1600,SDO_GEOMETRY(2001, 4326, MDSYS.SDO_POINT_TYPE(-74.695305, 41.35733, NULL), NULL, NULL));

    INSERT INTO WAREHOUSES values (4,'Seattle, WA',1700, SDO_GEOMETRY(2001, 4326, MDSYS.SDO_POINT_TYPE(-123.61526, 46.257458, NULL), NULL, NULL));

    COMMIT;
    </copy>
    ````
   



   **The elements of the constructor are:**

   -	2001: SDO-GTYPE attribute and it is set to 2001 when storing a two-dimensional single point such as a customer's location.
   -	4326: This is the spatial reference system ID (SRID): a foreign key to an Oracle dictionary table  (MDSYS.CS-SRS) that contains all the supported coordinate systems. It is important to associate your customer's location to a coordinate system. In this example, 4326 corresponds to "Longitude / Latitude (WGS 84)."
   -	MDSYS.SDO-POINT-TYPE: This is where you store your longitude and latitude values within the SDO_GEOMETRY constructor. 
     Note that you can store a third value also, but for these tutorials, all the customer data is two-dimensional.
   -	NULL, NULL: The last two null values are for storing linestrings, polygons, and geometry collections. 
     For more information on all the fields of the SDO_GEOMETRY object, please refer to the Oracle Spatial Developer's Guide. For this tutorial with point data,  these last two fields should be set to NULL.


 **Create spatial indexes**


   1. **Create Indexes**
   
  ````
    <copy>
   CREATE INDEX customers_sidx ON customers(CUST_GEO_LOCATION) 
   indextype is mdsys.spatial_index; 

   CREATE INDEX warehouses_sidx ON warehouses(WH_GEO_LOCATION)
    indextype is mdsys.spatial_index;  
   
   </copy>
    ````

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




2. **Query 2:** Find the five customers closest to warehouse 3 and put the results in order of distance
   
     ````
    <copy>
   SELECT c.customer_id, c.cust_last_name, c.GENDER,  
   round( sdo_nn_distance (1), 2) distance_in_miles
  FROM warehouses w, customers c WHERE w.warehouse_id = 3 AND 
  sdo_nn (c.cust_geo_location, w.wh_geo_location, 'sdo_num_res=5  unit=mile', 1) = 'TRUE' 
  ORDER BY distance_in_miles; 
       </copy>
       ````



   **Notes on Query 2:**

    -	The SDO-NN-DISTANCE operator is an ancillary operator to the SDO_NN operator; it can only be used within the SDO-NN operator. 
    The argument for this operator is a number that matches the number specified as the last argument of SDO-NN; in this example it is 1. There is no hidden meaning to this argument, it is simply a tag. If SDO-NN-DISTANCE() is specified, you can order the results by distance and guarantee that the first row returned is the closest. If the data you are querying is stored as longitude and latitude, the default unit for SDO-NN-DISTANCE is meters.
    -	The SDO-NN operator also has a UNIT parameter that determines the unit of measure returned by  SDO-NN-DISTANCE.
    - The ORDER BY DISTANCE clause ensures that the distances are returned in order, 
      with the shortest   distance first.



3. **Query 3:** Find the five female customers closest to warehouse 3, put the results in order of distance, and give the distance in miles

    ````
    <copy>
  SELECT
  c.customer_id,
  c.cust_last_name,
  c.GENDER,
  round( sdo_nn_distance(1), 2) distance_in_miles
  FROM warehouses w,
  customers c
  WHERE w.warehouse_id = 3
  AND sdo_nn (c.cust_geo_location, w.wh_geo_location,
  'sdo_batch_size =5 unit=mile', 1) = 'TRUE'
  AND c.GENDER = 'F'
  AND rownum < 6
  ORDER BY distance_in_miles;
  </copy>
    ````

   **Notes on Query 3:**

    -  SDO-BATCH-SIZE is a tunable parameter that may affect your query's performance. SDO-NN internally calculates that number  of   distances at a time. The initial batch of rows returned may not satisfy the constraints in the WHERE clause, so the number   of rows  specified by SDO-BATCH-SIZE is continuously returned until all the constraints in the WHERE clause are satisfied. 
    - You should choose a SDO-BATCH-SIZE that initially returns the number of rows likely to satisfy the constraints in your WHERE clause.
    - The UNIT parameter used within the SDO_NN operator specifies the unit of measure of the SDO-NN-DISTANCE parameter. 
    - The default  unit  is the unit of measure associated with the data. For longitude and latitude data, the default is meters.
    - c.gender = 'F' and rownum < 6 are the additional constraints in the WHERE clause. The rownum < 6 clause is necessary to limit the number of results returned to fewer than 6.
    - The ORDER BY DISTANCE-IN-MILES clause ensures that the distances are returned in order, with the shortest distance first and the distances measured in miles.



4. **Query 4:** Find all the customers within 100 miles of warehouse 3
   
      ````
      <copy>
   SELECT 	c.customer_id, c.cust_last_name,c.GENDER
   FROM 	warehouses w, customers c
   WHERE 	w.warehouse_id = 3 AND sdo_within_distance (c.cust_geo_location,w.wh_geo_location,
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



5. **Query 5:** Find all the customers within 100 miles of warehouse 3, put the results in order of distance, and give the distance in miles    
   
    ````
    <copy>
   SELECT  c.customer_id, c.cust_last_name,c.GENDER,
   round(sdo_geom.sdo_distance (c.cust_geo_location,w.wh_geo_location,.005, 'unit=MILE'), 2) distance_in_miles FROM warehouses w, customers c
   WHERE w.warehouse_id = 3 AND sdo_within_distance (c.cust_geo_location,
   w.wh_geo_location,'distance = 100 unit=MILE') = 'TRUE' ORDER BY distance_in_miles;
 
    </copy>
     ````

     
 **Notes on Query 5:**

- 	The SDO_GEOM.SDO-DISTANCE function computes the exact distance between the customer's location and warehouse 3. 
-   The first argument to SDO-GEOM.SDO-DISTANCE (c.cust-geo-location in the example above) contains the customer's location  whose  distance from warehouse 3 is to be computed. 
-   The second argument to SDO-WITHIN-DISTANCE (w.wh-geo-location in the example above) is the location of warehouse 3, whose distance from the customer's location is to be computed.
-	The third argument to SDO-GEOM.SDO-DISTANCE (0.005) is the tolerance value. The tolerance is a round-off error value used by Oracle Spatial. The tolerance is in meters for longitude and latitude data. In this example, the tolerance is 5 mm.
-	The UNIT parameter used within the SDO-GEOM.SDO-DISTANCE parameter specifies the unit of measure of the distance computed by the SDO-GEOM.SDO-DISTANCE function. The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters. In this example it is miles.
-	The ORDER BY DISTANCE-IN-MILES clause ensures that the distances are returned in order, with the shortest distance first and the distances measured in miles.

**Perform location-based queries**



1. Find the five customers closest to warehouse named 'Seattle, WA' and put the results in order of distance
   
     ````
    <copy>
     SELECT c.customer_id,c.cust_last_name,c.GENDER,round( sdo_nn_distance (1), 2)  distance_in_miles  FROM warehouses w, customers c 
     WHERE w.WAREHOUSE_NAME = 'Seattle, WA'   AND sdo_nn
     (c.cust_geo_location,w.wh_geo_location,'sdo_num_res=5 unit=mile',1) = 'TRUE' ORDER BY distance_in_miles;


       </copy>
       ````

   ![](./images/spatial_m2.PNG " ")

   **Notes:**

    -	The SDO-NN-DISTANCE operator is an ancillary operator to the SDO_NN operator; it can only be used within the SDO-NN operator. 
    The argument for this operator is a number that matches the number specified as the last argument of SDO-NN; in this example it is 1. There is no hidden meaning to this argument, it is simply a tag. If SDO-NN-DISTANCE() is specified, you can order the results by distance and guarantee that the first row returned is the closest. If the data you are querying is stored as longitude and latitude, the default unit for SDO-NN-DISTANCE is meters.
    -	The SDO-NN operator also has a UNIT parameter that determines the unit of measure returned by  SDO-NN-DISTANCE.
    - The ORDER BY DISTANCE clause ensures that the distances are returned in order, 
      with the shortest   distance first.

**Perform location-based queries**



1. Find the five female customers closest to warehouse named 'Sussex, NJ', put the results in order of distance, and give the distance in miles

    ````
    <copy>
  SELECT c.customer_id,c.cust_last_name,c.GENDER,round( sdo_nn_distance(1), 2) distance_in_miles FROM warehouses w, customers c
   WHERE w.WAREHOUSE_NAME = 'Sussex, NJ'  AND sdo_nn (c.cust_geo_location, w.wh_geo_location,'sdo_batch_size =5 unit=mile', 1) = 'TRUE'
  AND c.GENDER = 'F'AND rownum< 6  ORDER BY distance_in_miles; 

  </copy>
    ````

     ![](./images/spatial_m3.PNG " ")


   **Notes**

- SDO-BATCH-SIZE is a tunable parameter that may affect your query's performance. SDO-NN internally calculates that number of distances at a time. The initial batch of rows returned may not satisfy the constraints in the WHERE clause, so the number of rows specified by SDO-BATCH-SIZE is continuously returned until all the constraints in the WHERE clause are satisfied. You should choose a SDO-BATCH-SIZE that initially returns the number of rows likely to satisfy the constraints in your WHERE clause.
- The UNIT parameter used within the SDO-NN operator specifies the unit of measure of the SDO-NN-DISTANCE parameter. The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters.
- c.gender = 'F' and rownum< 6 are the additional constraints in the WHERE clause. The rownum< 6 clause is necessary to limit the number of results returned to fewer than 6.
- The ORDER BY DISTANCE-IN-MILES clause ensures that the distances are returned in order, with the shortest distance first and the distances measured in miles.



**Perform location-based queries**

1. Find all the customers within 100 miles of warehouse named 'Sussex, NJ'
   
     ````
    <copy>
    SELECT c.customer_id,c.cust_last_name,c.GENDER
    FROM warehouses w,   customers c
    WHERE w.WAREHOUSE_NAME = 'Sussex, NJ' 
    AND sdo_within_distance (c.cust_geo_location,w.wh_geo_location,
   'distance = 100 unit=MILE') = 'TRUE';



       </copy>
       ````

   ![](./images/spatial_m4.PNG " ")

  
 **Notes:**

- The SDO-WITHIN-DISTANCE operator returns the customers from the customers table that are within 100 miles of warehouse 3. The first argument to SDO-WITHIN-DISTANCE (c.cust-geo-location in the example above) is the column to search. The second argument to SDO-WITHIN-DISTANCE (w.wh-geo-location in the example above) is the location you want to determine the distances from. No assumptions should be made about the order of the returned results. For example, the first row returned is not guaranteed to be the customer closest to warehouse 3.
- The DISTANCE parameter used within the SDO-WITHIN-DISTANCE operator specifies the distance value; in this example it is 100.
- The UNIT parameter used within the SDO-WITHIN-DISTANCE operator specifies the unit of measure of the DISTANCE parameter. The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters; in this example, it is miles.


**Perform location-based queries**



1. Find all the customers within 100 miles of warehouse named 'Sussex, NJ', put the results in order of distance, and give the distance in miles.   
   
    ````
    <copy>
    SELECT c.customer_id, c.cust_last_name,  c.GENDER,
    round( sdo_geom.sdo_distance (c.cust_geo_location,
    w.wh_geo_location, .005, 'unit=MILE'), 2) distance_in_miles
    FROM warehouses w, customers c
    WHERE sdo_within_distance (c.cust_geo_location,w.wh_geo_location, 
    distance = 100 unit=MILE') = 'TRUE'
    ORDER BY distance_in_miles;

 
    </copy>
     ````

     
     ![](./images/spatial_m5.PNG " ")

 **Notes**

- The SDO-GEOM.SDO-DISTANCE function computes the exact distance between the customer's location and warehouse 3. The first argument to SDO_GEOM.SDO-DISTANCE (c.cust-geo-location in the example above) contains the customer's location whose distance from warehouse 3 is to be computed. The second argument to SDO-WITHIN-DISTANCE (w.wh-geo-location in the example above) is the location of warehouse 3, whose distance from the customer's location is to be computed.
- The third argument to SDO-GEOM.SDO-DISTANCE (0.005) is the tolerance value. The tolerance is a round-off error value used by Oracle Spatial. The tolerance is in meters for longitude and latitude data. In this example, the tolerance is 5 mm.
- The UNIT parameter used within the SDO-GEOM.SDO-DISTANCE parameter specifies the unit of measure of the distance computed by the SDO-GEOM.SDO-DISTANCE function. The default unit is the unit of measure associated with the data. For longitude and latitude data, the default is meters. In this example it is miles.
- The ORDER BY DISTANCE-IN-MILES clause ensures that the distances are returned in order, with the shortest distance first and the distances measured in miles.





See an issue?  Please open up a request [here](https://github.com/oracle/learning-library/issues).