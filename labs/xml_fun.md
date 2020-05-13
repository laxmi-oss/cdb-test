# Oracle XML  


## Steps:

1.	Get the list of the customer and their purchased information from a geo graphical location 
    
    XMLEXISTS is an SQL/XML operator that you can use to query XML values in SQL, in a regular query I can use the xmlexists function to look if a specific value is present in an xmltype column
    ````
    <copy>
    SELECT t.object_value.getclobval() FROM   purchaseorder t
    WHERE xmlexists('/PurchaseOrder/ShippingInstructions/Address[city/text()=$CITY]' passing object_value, 'South San Francisco' AS "CITY" );


      </copy>
    ````
  
   ![](./images/xml_m6.PNG " ")

  **Notes:** XMLEXISTS is an SQL/XML operator that we can use to query XML values in SQL, in a regular query we can use the xmlexists function to look if a specific value is present in an xmltype column.

2. Customer purchase history  
    
    XMLTABLE: Convert XML Data into Rows and Columns using SQL. The XMLTABLE operator, which allows you to project columns on to XML data in an XMLTYPE , making it possible to query the data directly from SQL as if it were relational data.

    ````
    <copy>
    SELECT t.object_value.getclobval()
    FROM   purchaseorder p,
    XMLTABLE('for $r in /PurchaseOrder[Reference/text()=$REFERENCE] return $r' passing object_value, 'AHUNOLD-20141130' AS  "REFERENCE") t;  



      </copy>
    ````
  
   ![](./images/xml_m7.PNG " ")

   **Notes:** The XMLTABLE operator, which allows you to project columns on to XML data in an XMLTYPE , making it possible to query the data directly from SQL as if it were relational data.
   

3. Listing the product description those unit price matches to ‘$xx’
    
    XMLSERIALIZE is a SQL/XML operator that you can use to convert an XML type to a character type.

    ````
    <copy>
    SELECT XMLSERIALIZE(CONTENT COLUMN_VALUE AS CLOB INDENT SIZE=2) 
    FROM   Purchaseorder p, XMLTable('<Summary> { for $r in 
    /PurchaseOrder/LineItems/Part   return $r/Description }    </Summary>' passingobject_value   )
    WHERE  xmlexists('/PurchaseOrder/LineItems/Part[UnitPrice/text()=$UnitPrice]' passing object_value, '27.95' AS "UnitPrice" ); 
    /

      </copy>
    ````
  
   ![](./images/xml_m8.PNG " ")
    
    **Notes:** XMLSERIALIZE is a SQL/XML operator that you can use to convert an XML type to a character type.

4. Customer order summary – Cost center wise 
    
    XMLQUERY lets you query XML data in SQL statements. It takes an XQuery expression as a string literal, an optional context item, and other bind variables and returns the result of evaluating the XQuery expression using these input values. XQuery string is a complete XQuery expression, including prolog.

    ````
    <copy>
    SELECT xmlquery(
    '<POSummary lineItemCount="{count($XML/PurchaseOrder/LineItems/ItemNumber)}">{
    $XML/PurchaseOrder/User, $XML/PurchaseOrder/Requestor,
    $XML/PurchaseOrder/LineItems/LineItem[2]
    }
    </POSummary>' passingobject_value AS "XML"
    returning content 
    ).getclobval() initial_state
    FROM   PURCHASEORDER
    WHERE  xmlExists(
    '$XML/PurchaseOrder[CostCenter=$CS]'
    passingobject_value AS "XML",
    'A90' AS "CS"       )
    /


      </copy>
    ````
  
   ![](./images/xml_m9.PNG " ")

   **Notes:** XMLQUERY lets you query XML data in SQL statements. It takes an XQuery expression as a string literal, an optional context item, and other bind variables and returns the result of evaluating the XQuery expression using these input values. XQuery string is a complete XQuery expression, including prolog.
        
5. Custer Delivery Priority Instruction  
    
    Ex - Courier, Expidite,Surface Mail,Air Mail etc..

    ExistsNode() checks if xpath-expression returns at least one XML element or text node. If so, existsNode returns 1, otherwise, it returns 0. existsNode should only be used in the where clause of the select statement.


    ````
    <copy>
    SELECT extractValue(OBJECT_VALUE, '/PurchaseOrder/Reference') "REFERENCE"
    FROM purchaseorder WHERE existsNode(OBJECT_VALUE, '/PurchaseOrder[Special_Instructions="Next Day Air"]')=1;



      </copy>
    ````
  
    
       ![](./images/xml_m10_a.PNG " ")

6.	 Run Select Query
    
    
    ````
    <copy>
    SELECT extractValue(OBJECT_VALUE, '/PurchaseOrder/Reference') "REFERENCE"
  FROM purchaseorder
  WHERE existsNode(OBJECT_VALUE, '/PurchaseOrder[Special_Instructions="Priority Overnight"]')=1;


      </copy>
    ````
  
    ![](./images/xml_m10_b.PNG " ")
    ![](./images/xml_m10_c.PNG " ")
   
    **Notes**: ExistsNodechecks if xpath-expression returns at least one XML element or text node. If so, existsNode returns 1, otherwise, it returns 0. existsNode should only be used in the where clause of the select statement.

See an issue?  Please open up a request [here](https://github.com/oracle/learning-library/issues).        