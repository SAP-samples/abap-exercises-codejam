# Exercise 2 - Exposing and Consuming Services via HTTP

In this exercise, we will create an HTTP serivce and custom ABAP HTTP handler class.  We will then consume an external HTTP service as well by leveraging the SAP API Hub.   You can view APIs of the SAP API Hub at the following URL.

https://api.sap.com/package/SAPS4HANACloud/all

In this exercise, we will use the Bank – Read API, which allows us to retrieve a list of bank details.  This page gives you all the information about calling the API and also supplies the API key which we will need to supply when calling from our ABAP class.

https://api.sap.com/api/API_BANKDETAIL_SRV/resource

## Exercise 2.1 Expose an HTTP Service in ABAP

After completing these steps you will have created an HTTP service along with its handler class. 

1. Right-click on your package and choose “New“, “Other ABAP Repository Object“.
<br>![](/exercises/ex2/images/02_01_0010.png)

2. Under "Connectivity, choose "HTTP Service" and click "Next".
<br>![](/exercises/ex2/images/02_01_0020.png)

3. Give the name of the service as ZHTTPSRV_XXX where XXX is your group number.  Keep the handler class name as generated, then click "Next".
<br>![](/exercises/ex2/images/02_01_0030.png)

4. Click "Finish".
<br>![](/exercises/ex2/images/02_01_0040.png)

5. Your HTTP Service is now created, click on the "Handler Class" link, this will take you to that class so that you can implement it. 
<br>![](/exercises/ex2/images/02_01_0050.png)

6. Here you can see that the correct interface has been added to the PUBLIC section for you, and the HANDLE_REQUEST method shell is there in the implementation as well. 
<br>![](/exercises/ex2/images/02_01_0060.png)

7. Next, add the following code to the HANDLE_REQUEST method.  In this method, we will retrieve the URL parameters from the request object and check for the "cmd" parameter and get its value.  Based on that value, we will do something, for example, if cmd=timestamp, then we will write the curret timestamp to the response object. 
```abap
   DATA(lt_params) = request->get_form_fields(  ).
    READ TABLE lt_params REFERENCE INTO DATA(lr_params) WITH KEY name = 'cmd'.
    IF sy-subrc <> 0.
      response->set_status( i_code = 400
                       i_reason = 'Bad request' ).
      RETURN.
    ENDIF.
    CASE lr_params->value.
      WHEN `timestamp`.        
        response->set_text( |HTTP Service Application executed by {
                             cl_abap_context_info=>get_user_technical_name( ) } | &&
                              | on { cl_abap_context_info=>get_system_date( ) DATE = ENVIRONMENT } | &&
                              | at { cl_abap_context_info=>get_system_time( ) TIME = ENVIRONMENT } | ).

      WHEN OTHERS.
        response->set_status( i_code = 400 i_reason = 'Bad request' ).
    ENDCASE.
```

8. Your code should now look like this.
<br>![](/exercises/ex2/images/02_01_0070.png)

9. Save and activate your class.
<br>![](/exercises/ex2/images/02_01_0080.png)

10. Now return to the HTTP Service and click on the "URL" link.
<br>![](/exercises/ex2/images/02_01_0090.png)

11. The browser should open and you may have to log on. Once you are logged on, you will most likely get this screen. We are getting an error because we have not set the URL parameter called "cmd" yet. We will do this next. 
<br>![](/exercises/ex2/images/02_01_0100.png)

12. Add &cmd=timestamp to the end of the URL in the browser and hit enter. 
<br>![](/exercises/ex2/images/02_01_0110.png)

13. You should now see the correct output.
<br>![](/exercises/ex2/images/02_01_0120.png)


## Exercise 2.2 Consume an external HTTP service in ABAP

After completing these steps you will have consumed an external API from the SAP API Hub, and viewed the results via the browser.

1. First, lets go to the SAP API Hub. From the browser, go to https://api.sap.com/api/API_BANKDETAIL_SRV/resource.  Log on to the site, you may have to register first. <br>![](/exercises/ex2/images/02_02_0010.png)

2. Click the "Show API Key" button.
<br>![](/exercises/ex2/images/02_02_0020.png)

3. Next, click the "Copy Key and Close" button.  Paste this code somewhere in a safe place, we will need it a little later. 
<br>![](/exercises/ex2/images/02_02_0030.png)

4.	Now return to your ZCL_HTTPSRV_XXX class and modify it.  Add a METHOD called get_bank_details with a returning parameter of type string to the private section. Also add the shell of the METHOD implementation as well.  
<br>![](/exercises/ex2/images/02_02_0040.png)

5. Add the implementation of the GET_BANK_DETAILS method as shown below.  Here we are utlizing the SAP delivered HTTP interface classes.  We access the request object and insert the API key and the execute the request and retrieve the result.  We then pass the result to the output parameter as JSON.
```abap

    DATA: lv_url TYPE string VALUE 'https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/'.
    DATA: lo_http_client TYPE REF TO  if_web_http_client.

    lo_http_client = cl_web_http_client_manager=>create_by_http_destination(
              i_destination = cl_http_destination_provider=>create_by_url( lv_url ) ).

    DATA(lo_request) = lo_http_client->get_http_request( ).

    lo_request->set_header_fields( VALUE #(
       (  name = 'Content-Type' value = 'application/json' )
       (  name = 'Accept' value = 'application/json' )
       (  name = 'APIKey' value = '<insert API key here>') ) ).  "<<-- API KEY!

    lo_request->set_uri_path(
       i_uri_path = lv_url && 'API_BANKDETAIL_SRV/A_BankDetail?$top=25&$format=json'  ).

    TRY.
        DATA(lv_response) = lo_http_client->execute( i_method = if_web_http_client=>get )->get_text(  ).
      CATCH cx_web_http_client_error.
    ENDTRY.

    r_json = lv_response.

```
6.	Your code should now look like this.
<br>![](/exercises/ex2/images/02_02_0050.png)

7.	Next, go to the HANDLE_REQUEST method, and modify it.  Add another WHEN condition in the CASE statement.  When the URL parameter cmd is = getbankdetails, set the content type to 'application/json' and call the GET_BANK_DETAILS method and pass the returning string to the response object SET_TEXT method. 
<br>![](/exercises/ex2/images/02_02_0060.png)

8.	Save and activate your work.
<br>![](/exercises/ex2/images/02_02_0070.png)

9.	Once again return to the HTTP service and click the "URL" link.  Add the URL parameter as &cmd=getbankdetails and hit enter.  You should now see the bank details in JSON format. 
<br>![](/exercises/ex2/images/02_02_0080.png)

5.	CHALLENGE!  Use what you have learned and implement a new method in your ZCL_HTTPSRV_XXX class for another API.  You can choose one from this page. https://api.sap.com/package/SAPS4HANACloud/all. Choose one that is an OData API and perhaps one that is a READ type operation

6. As your API keys are now exposed in your code, I would highly recommend that you return to your ZCL_MY_HTTPSRV_XXX class and remove them.


## Summary

You've now created a new HTTP service that exposes functionality from ABAP as well as you have learned how to consume an external service via HTTP. 

Continue to - [Exercise 3 - Service Consumption Model ](../ex3/README.md)
