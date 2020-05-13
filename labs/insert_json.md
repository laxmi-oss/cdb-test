
# Oracle JSON 



## Steps:#

 **Insert and Update on JSON data**

  We can use standard database APIs to insert or update JSON data in Oracle Database. We can work directly with JSON data contained in file-system files by creating an external table that exposes it to the database. We can use JSON Merge Patch to update a JSON document.


1. Take a count of the rows in the json table-
   
    ````
    <copy>
    select count(*) from purchase_order;
    </copy>
    ````
    
    ![](./images/insert_json.PNG " ")

2. Create a directory
    
    We will create a directory which will point to the location where json dump file is stored-
     
    ````
    <copy>
    INSERT INTO purchase_order
    VALUES (
    SYS_GUID(),
    to_date('05-MAY-2020'),
    '{"PONumber"             : 10001,
     "Reference"            : "SBELL-20141017",
     "Requestor"            : "Sarah Bell",
     "User"                 : "SBELL",
     "CostCenter"           : "A50",
     "ShippingInstructions" : {"name"    : "Sarah Bell",
     "Address" : {"street"  : "200 Sporting Green",
     "city"    : "South San Francisco",
     "state"   : "CA",
     "zipCode" : 99236,
     "country" : "United States of America"},
     "Phone"   : "983-555-6509"},
      "Special Instructions" : "Courier",
      "LineItems"            : [{"ItemNumber" : 1,
     "Part"       : {"Description" : "Making the Grade",
      "UnitPrice"   : 20,
      "UPCCode"     : 27616867759},
      "Quantity"   : 8.0},
     {"ItemNumber" : 2,
       "Part"       : {"Description" : "Nixon",
       "UnitPrice"   : 19.95,
        "UPCCode"     : 717951002396},
       "Quantity"   : 5},
      {"ItemNumber" : 3,
          "Part"       : {"Description" : "Eric Clapton: Best Of 1981-1999",
         "UnitPrice"   : 19.95,
         "UPCCode"     : 75993851120},
          "Quantity"   : 5.0}
          ]}');

    
             </copy>
    ````
  
  **UPDATE :** 
    
  We can use Oracle SQL function json-mergepatch or PL/SQL object-type method json-mergepatch() to update specific portions of a JSON document. In both cases we provide a JSON Merge Patch document, which declaratively specifies the changes to make to a a specified JSON document. JSON Merge Patch is an IETF standard.

3. Create a simple table to store JSON documents
   
   
    ````
    <copy>
    update purchase_order
     set    PO_DOCUMENT = json_mergepatch ( 
         PO_DOCUMENT,
         '{
           "Requestor" : "MSDhoni"
         }'
       )
    where id ='A4E055B4CF4A23A4E0530900000A60C2';

    </copy>
    ````
    
    ![](./images/json_lab7_6.PNG " ")


See an issue?  Please open up a request [here](https://github.com/oracle/learning-library/issues).