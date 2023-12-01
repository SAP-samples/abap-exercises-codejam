# Exercise 3 - Service Consumption Model

In this exercise we will explore how to use the Service Consumption Model concept to consume various services in ABAP.

## Exercise 3.1 Create a Service Consumption Model Definition for an OData Service

After completing these steps you will have created a Service Consumption Model definition for an OData Service exposed from the SAP API Hub.

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

5.	Give the name as "ZSCM_BANK_DETAILS_XXX" where XXX is your group number and give a meaningful description.  Set the "Remote Consumption Mode" to OData, then click "Next".
<br>![](/exercises/ex3/images/03_01_0050.png)

6.	Click the "Browse" button.
<br>![](/exercises/ex3/images/03_01_0060.png)

7.	Navigate to the location where you saved the EDMX file eariler. 
<br>![](/exercises/ex3/images/03_01_0070.png)

8.	Then click "Next".
<br>![](/exercises/ex3/images/03_01_0080.png)

9.	Once again, click "Next".
<br>![](/exercises/ex3/images/03_01_0090.png)


THIS EXERCISE IS BROKEN FROM HERE, CURRENTLY SHOWING ERROR CODE WHEN CLICKING FINISH!

10.	Now click "Finish".
<br>![](/exercises/ex3/images/03_01_0100.png)

11.	The Service Consumption Model objects have now been created for you.  When you take a look at the SCM definition, you can see that it provides code snippets for you to use in your code for different CRUD operations.  Continue to the next step.
<br>![](/exercises/ex3/images/03_01_0110.png)

12. Use what you have learned and create a new class called ZCL_SCM_XXX where XXX is your group number.  Make sure to include the IF_OO_ADT_CLASSRUN interface and add the shell of the MAIN method implementation as shown below. 
<br>![](/exercises/ex3/images/03_01_0120.png)

13.	In the private section, add the following lines of code. Make sure to replace XXX with your group number.  We want to encapsulate the logic into a functional method call which contains a returning parameter for the list of bank details.
```abap
    TYPES tt_za_bankdetails TYPE STANDARD TABLE OF zscm_bank_details_xxx=>tys_a_bank_detail_type WITH EMPTY KEY.
    METHODS get_bank_details_scm RETURNING VALUE(rt_table) TYPE tt_za_bankdetails.
```
<!---<br>![](/exercises/ex3/images/03_01_0130.png) --->

14.	Now add the shell of the method implementation as shown below.
<br>![](/exercises/ex3/images/03_01_0140.png)

15.	Return to the Service Consumption Model definition and choose the "Read List" operation, then click "Copy to Clipboard".
<br>![](/exercises/ex3/images/03_01_0150.png)

16. Now, paste the code into the GET_BANK_DETAILS_SCM method. Update the view name as shown.  Make sure to replace XXX with your group number. 
<br>![](/exercises/ex3/images/03_01_0160.png)

17.	Copy the following code as the first lines within the TRY statement.  Here we will utilize the standard HTTP client classes provided by SAP. We will pass the URL and get an instance of the HTTP client object, where we then set the API key.  Make sure to insert your API key here from the API hub. 
```abap
        DATA: lv_url TYPE string VALUE 'https://sandbox.api.sap.com/'.
        lo_http_client = cl_web_http_client_manager=>create_by_http_destination(
                        i_destination = cl_http_destination_provider=>create_by_url( lv_url ) ).

        lo_http_client->get_http_request( )->set_header_fields( VALUE #(
             (  name = 'APIKey' value = '<insert API key here>') ) ).

```
<!---<br>![](/exercises/ex3/images/03_01_0170.png)--->

18. Next, modify the next statement where the client proxy object is created.  Update the service root parameter value as shown here. 
```abap
        lo_client_proxy = /iwbep/cl_cp_factory_remote=>create_v2_remote_proxy(
          EXPORTING
             is_proxy_model_key       = VALUE #( repository_id       = 'DEFAULT'
                                                 proxy_model_id      = 'ZSCM_BANK_DETAILS_XXX'
                                                 proxy_model_version = '0001' )
            io_http_client             = lo_http_client
            iv_relative_service_root   = '/s4hanacloud/sap/opu/odata/sap/API_BANKDETAIL_SRV' ).
```
<!---<br>![](/exercises/ex3/images/03_01_0180.png)--->

19. Next, change the value passed to the SET_TOP method to 500.  And finally, change the importing parameter value to RT_TABLE.  RT_TABLE is the returning parameter of your method, so we are simply passing the results of the GET_BUSINESS_DATA method back to the caller so that we can write out the results to the console. 
<br>![](/exercises/ex3/images/03_01_0190.png)

20. Next, add the following line of code to the MAIN method implementatino.  Here we are simply writing out the results to the console. 
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
        lt_range_BANK_COUNTRY = VALUE #( ( sign = 'I' option = 'EQ' low = 'DE' high = ' ' ) ).
```
<!---<br>![](/exercises/ex3/images/03_01_0250.png)--->

26. Next, uncomment these two lines of code, and since we only have one filtering node, update these two statements as shown here. 
<br>![](/exercises/ex3/images/03_01_0260.png)


27.  You completed code should now look like this.
```abap
CLASS zcl_scm_xxx DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.

    INTERFACES if_oo_adt_classrun .
  PROTECTED SECTION.
  PRIVATE SECTION.

    TYPES tt_za_bankdetails TYPE STANDARD TABLE OF za_bankdetails_xxx WITH EMPTY KEY.
    METHODS get_bank_details_scm RETURNING VALUE(rt_table) TYPE tt_za_bankdetails.

ENDCLASS.



CLASS zcl_scm_xxx IMPLEMENTATION.

  METHOD if_oo_adt_classrun~main.

    out->write( get_bank_details_scm(  ) ).

  ENDMETHOD.

  METHOD get_bank_details_scm.


    DATA:
      lt_business_data TYPE TABLE OF za_bankdetails_xxx,
      lo_http_client   TYPE REF TO if_web_http_client,
      lo_client_proxy  TYPE REF TO /iwbep/if_cp_client_proxy,
      lo_request       TYPE REF TO /iwbep/if_cp_request_read_list,
      lo_response      TYPE REF TO /iwbep/if_cp_response_read_lst.

    DATA:
      lo_filter_factory    TYPE REF TO /iwbep/if_cp_filter_factory,
      lo_filter_node_1     TYPE REF TO /iwbep/if_cp_filter_node,
* lo_filter_node_2    TYPE REF TO /iwbep/if_cp_filter_node,
      lo_filter_node_root  TYPE REF TO /iwbep/if_cp_filter_node,
      lt_range_BANKCOUNTRY TYPE RANGE OF za_bankdetails_xxx-BankCountry.
* lt_range_BANKINTERNALID TYPE RANGE OF <element_name>.


    TRY.
        " Create http client
*DATA(lo_destination) = cl_http_destination_provider=>create_by_comm_arrangement(
*                                             comm_scenario  = '<Comm Scenario>'
*                                             comm_system_id = '<Comm System Id>'
*                                             service_id     = '<Service Id>' ).
*lo_http_client = cl_web_http_client_manager=>create_by_http_destination( lo_destination ).

        DATA: lv_url TYPE string VALUE 'https://sandbox.api.sap.com/'.
        lo_http_client = cl_web_http_client_manager=>create_by_http_destination(
                        i_destination = cl_http_destination_provider=>create_by_url( lv_url ) ).

        lo_http_client->get_http_request( )->set_header_fields( VALUE #(
             (  name = 'APIKey' value = '<insert APi key here>') ) ).

        lo_client_proxy = cl_web_odata_client_factory=>create_v2_remote_proxy(
          EXPORTING
            iv_service_definition_name = 'ZSCM_BANK_DETAILS_XXX'
            io_http_client             = lo_http_client
            iv_relative_service_root   = '/s4hanacloud/sap/opu/odata/sap/API_BANKDETAIL_SRV' ).


        " Navigate to the resource and create a request for the read operation
        lo_request = lo_client_proxy->create_resource_for_entity_set( 'A_BANKDETAIL' )->create_request_for_read( ).

        " Create the filter tree
        lo_filter_factory = lo_request->create_filter_factory( ).

        lt_range_BANKCOUNTRY = VALUE #( ( sign = 'I' option = 'EQ' low = 'DE' high = ' ' ) ).

        lo_filter_node_1  = lo_filter_factory->create_by_range( iv_property_path     = 'BANKCOUNTRY'
                                                                it_range             = lt_range_BANKCOUNTRY ).
*lo_filter_node_2  = lo_filter_factory->create_by_range( iv_property_path     = 'BANKINTERNALID'
*                                                        it_range             = lt_range_BANKINTERNALID ).

        lo_filter_node_root = lo_filter_node_1.
        lo_request->set_filter( lo_filter_node_root ).

        lo_request->set_top( 500 )->set_skip( 0 ).

        " Execute the request and retrieve the business data
        lo_response = lo_request->execute( ).
        lo_response->get_business_data( IMPORTING et_business_data = rt_table ).

      CATCH /iwbep/cx_cp_remote INTO DATA(lx_remote).
        " Handle remote Exception
        " It contains details about the problems of your http(s) connection

      CATCH /iwbep/cx_gateway INTO DATA(lx_gateway).
        " Handle Exception

    ENDTRY.

  ENDMETHOD.

ENDCLASS.
```

28. Next, save and activate your work.
<br>![](/exercises/ex3/images/03_01_0270.png)

29. Finally, execute the class once again.  The results should now be shown with only rows where the BANKCOUNTRY = DE.
<br>![](/exercises/ex3/images/03_01_0280.png)




## Summary

You've now created your a Service Consumption Model for consuming an external service.

