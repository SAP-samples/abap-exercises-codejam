# Exercise 2 - Exposing and Consuming Services via HTTP

In this exercise, we will create an HTTP serivce and custom ABAP HTTP handler class.  We will then consume an external HTTP service as well by leveraging the SAP API Hub.   You can view APIs of the SAP API Hub at the following URL.

https://api.sap.com/package/SAPS4HANACloud?section=Artifacts

In this exercise, we will use the Bank – Read API, which allows us to retrieve a list of bank details.  This page gives you all the information about calling the API and also supplies the API key which we will need to supply when calling from our ABAP class.

https://api.sap.com/api/API_BANKDETAIL_SRV/resource

## Exercise 2.1 Expose an HTTP Service in ABAP

After completing these steps you will have created an HTTP service along with its handler class. 

1. Right-click on your package and choose “New“, “Other ABAP Repository Object“.
<br>![](/exercises/ex2/images/02_01_0010.png)

2. Under "Connectivity, choose "HTTP Service" and click "Next".
<br>![](/exercises/ex2/images/02_01_0020.png)

3. Give the name of the service as ZMY_HTTPSRV_XXX where XXX is your group number.  Keep the handler class name as generated, then click "Next".
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
                       i_reason = 'Bad request').
      RETURN.
    ENDIF.
    CASE lr_params->value.
      WHEN `timestamp`.        
        response->set_text( |HTTP Service Application executed by {
                             cl_abap_context_info=>get_user_technical_name( ) } | &&
                              | on { cl_abap_context_info=>get_system_date( ) DATE = ENVIRONMENT } | &&
                              | at { cl_abap_context_info=>get_system_time( ) TIME = ENVIRONMENT } | ).

      WHEN OTHERS.
        response->set_status( i_code = 400 i_reason = 'Bad request').
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


## Exercise 2.2 Consume an external HTTP service

After completing these steps you will have consumed an external API from the SAP API Hub, and viewed the results via the browser.

1.	Now return to your ZCL_HELLO_WORLD_XXX class and modify it.  Add another WHEN condition to your CASE statement as shown here.  Make sure to replace XXX with your group number.
```abap
 when `bankdetails`.
    response->set_text( new zcl_api_hub_manager_xxx(  )->get_bank_details( ) ).
```

2.	Your code should now look like this.
<br>![](/exercises/ex2/images/02_02_0120.png)

3.	Return to the HTTP Service defintion and click the URL link.  
<br>![](/exercises/ex2/images/02_02_0130.png)

4.	When the browser opens, change the URL to include cmd=bankdetails  
<br>![](/exercises/ex2/images/02_02_0140.png)

5.	CHALLENGE!  Use what you have learned and implement a new method in your ZCL_API_HUB_MANAGER_XXX class for another API.  You can choose one from this page. https://api.sap.com/package/SAPS4HANACloud?section=Artifacts. Choose one that is an OData API and perhaps one that is a READ type operation
<br>![](/exercises/ex2/images/02_02_0150.png)

6. As your API keys are now exposed in your code, I would highly recommend that you return to your ZCL_API_HUB_MANAGER_XXX class and remove them.


## Summary

You've now created a new class which calls the SAP API Hub, updated your hello World to call this new class, and viewed your results in the browser. 

Continue to - [Exercise 3 - ABAP RESTful Programming Model - Unmanaged ](../ex3/README.md)
