# Exercise 3 - Service Consumption Model

In this exercise we will explore.....

## Exercise 3.1 Create....

After completing these steps you will have created....

1.	Return to the SAP API Hub, and specifically to this page which shows the overview of the Bank Details API we've been working with.  
    https://api.sap.com/api/API_BANKDETAIL_SRV/overview   
    Click on the "API Specification" link.
<br>![](/exercises/ex3/images/03_01_0010.png)

2.	Next downbload the EDMX file to your desktop.
<br>![](/exercises/ex3/images/03_01_0020.png)

3.	Right-click on your group package, and choose "New", then "Other ABAP Repository Object"
<br>![](/exercises/ex3/images/03_01_0030.png)

4.	Click "Business Services", then choose "Service Consumption Model", then click "Next".
<br>![](/exercises/ex3/images/03_01_0040.png)

5.	Give the name as "ZSCM_BANK_DETAILS_XXX" where XXX is your group number.  Also, give a meaningful description, then click "Next".
<br>![](/exercises/ex3/images/03_01_0050.png)

6.	Click the "Browse" button.
<br>![](/exercises/ex3/images/03_01_0060.png)

7.	Navigate to the location where you saved the EDMX file eariler. 
<br>![](/exercises/ex3/images/03_01_0070.png)

8.	Change the name of the ABAP Artifact Name which had been generated for you. Name the object as ZA_BANKDETAIL_XXX where XXX is your group number.  Then click "Next".
<br>![](/exercises/ex3/images/03_01_0080.png)

9.	Once again, click "Next".
<br>![](/exercises/ex3/images/03_01_0090.png)

10.	Now click "Finish".
<br>![](/exercises/ex3/images/03_01_0100.png)

11.	The Service Consumption Model objects have now been created for you.  When you take a look at the SCM definition, you can see that it provides code snippets for you to use in your code for different CRUD operations. 
<br>![](/exercises/ex3/images/03_01_0110.png)

12. Use what you have learned and create a new class called ZCL_SCM_XXX where XXX is your group number.  Make sure to include the IF_OO_ADT_CLASSRUN interface and add the shell of the MAIN method implementation as shown below. 
<br>![](/exercises/ex3/images/03_01_0120.png)

13.	In the private section, add the following lines of code.  We want to encapsulate the logic into a functional method call which contains a returning parameter for the list of bank details.
```abap
    TYPES tt_za_bankdetail_xxx TYPE STANDARD TABLE OF za_bankdetail_xxx WITH EMPTY KEY.
    METHODS get_bank_details_scm RETURNING VALUE(rt_table) TYPE tt_za_bankdetail_xxx.
```

<br>![](/exercises/ex3/images/03_01_0130.png)

14.	Now add the shell of the method implementation as shown below.
<br>![](/exercises/ex3/images/03_01_0140.png)

15.	Return to the Service Consumption Model definition and choose the "Read List" operation, then click "Copy to Clipboard".
<br>![](/exercises/ex3/images/03_01_0150.png)

16. Uncomment the lines of code shown here, and make sure to update the view name as shown.  Make sure to replace XXX with your group number. 
<br>![](/exercises/ex3/images/03_01_0160.png)

17.	Copy the following code as the first lines within the TRY statement.  Here we will utilize the standard HTTP client classes provided by SAP. We will pass the URL and get an instance of the HTTP client object, where we then set the API key.  Make sure to insert your API key here from the API hub. 
```abap
        DATA: lv_url TYPE string VALUE 'https://sandbox.api.sap.com/'.
        lo_http_client = cl_web_http_client_manager=>create_by_http_destination(
                        i_destination = cl_http_destination_provider=>create_by_url( lv_url ) ).

        lo_http_client->get_http_request( )->set_header_fields( VALUE #(
             (  name = 'APIKey' value = '<insert API key here>') ) ).

```
<br>![](/exercises/ex3/images/03_01_0170.png)

18. Next, modify the next statement where the client proxy object is created.  Update the service root parameter value as shown here. 
```abap
        lo_client_proxy = cl_web_odata_client_factory=>create_v2_remote_proxy(
          EXPORTING
            iv_service_definition_name = 'ZSCM_BANK_DETAILS_XXX'
            io_http_client             = lo_http_client
            iv_relative_service_root   = '/s4hanacloud/sap/opu/odata/sap/API_BANKDETAIL_SRV' ).
```
<br>![](/exercises/ex3/images/03_01_0180.png)

19. Next, change the value passed to the SET_TOP method to 500.  And finally, change the importing parameter value to RT_TABLE.  RT_TABLE is the returning parameter of your method, so we are simply passing the results of the GET_BUSINESS_DATA method back to the caller so that we can write out the results to the console. 
<br>![](/exercises/ex3/images/03_01_0190.png)

20. Next, add the following line of code to the MAIN method implementatino.  Here we are simply writing out the results to the console. 
```abap
    out->write( get_bank_details_scm(  ) ).
```
<br>![](/exercises/ex3/images/03_01_0200.png)

21. Next, save and activate your work.
<br>![](/exercises/ex3/images/03_01_0210.png)

22. Now execute your class, but hitting F9.  You should see a list of bank details in the console.
<br>![](/exercises/ex3/images/03_01_0220.png)

23. Return to your class and uncomment the following lines of code.  Make sure to update the LT_RANGE_BANKCOUNTRY range table definition as shown here. Be sure to replace XXX with your group number.  Also, be sure to put a period at the end of this line as well.
<br>![](/exercises/ex3/images/03_01_0230.png)

24. Uncomment the following lines of code.  Here we will use the filtering classes to help us filter the results.
<br>![](/exercises/ex3/images/03_01_0240.png)

25. Add the following line of code, in between the two statements that you have just uncommented.  Here we are simply filling the range table with a filter value. We want to filter the results where BANKCOUNTRY = DE.
```abap
        lt_range_BANKCOUNTRY = VALUE #( ( sign = 'I' option = 'EQ' low = 'DE' high = ' ' ) ).
```
<br>![](/exercises/ex3/images/03_01_0250.png)

26. Next, since we only have one filtering node, update these two statements as shown here. 
<br>![](/exercises/ex3/images/03_01_0260.png)

27. Next, save and activate your work.
<br>![](/exercises/ex3/images/03_01_0270.png)

28. Finally, execute the class once again.  The results should now be shown with only rows where the BANKCOUNTRY = DE.
<br>![](/exercises/ex3/images/03_01_0280.png)

## Exercise 3.2 Create....

After completing these steps you will have....

1.	Right-click ....

2.	Provide the default description...

## Summary

You've now created your Business Object data model, defined and implemented your Behaviors, created Projections over both, and exposed your data model as an OData service using the Unmanaged Scenario of the ABAP RESTful Application Programming Model

Continue to - [Exercise 4 - ABAP RESTful Application Programming Model - Managed ](../ex4/README.md)
