# Exercise 2 - Consuming Services via HTTP

In this exercise, we will create a class which leverages the HTTP connectivity classes and consume an external API from the SAP API Hub.  You can view APIs of the SAP API Hub at the following URL.

https://api.sap.com/package/SAPS4HANACloud?section=Artifacts

In this exercise, we will use the Bank – Read API, which allows us to retrieve a list of bank details.  This page gives you all the information about calling the API and also supplies the API key which we will need to supply when calling from our ABAP class.

https://api.sap.com/api/API_BANKDETAIL_SRV/resource

## Exercise 2.1 Create API Hub Manager Class

After completing these steps you will have created a class which calls the whitelisted HTTP connectivity classes.

1. Right-click on your package and choose “New“, “ABAP Class“.
<br>![](/exercises/ex2/images/02_01_0010.png)

2. Name your new class as “ZCL_API_HUB_MANAGER_XXX“.  Click “Next“, then click “Finish“. Make sure to replace XXX with your group number.
<br>![](/exercises/ex2/images/02_01_0020.png)

3. In the PUBLIC section,  define the constructor method and one other method called get_bank_details as shown here.
```abap
METHODS: constructor,
        get_bank_details RETURNING VALUE(r_json) TYPE string.
```

4. Define two private global variables as shown here. 
```abap
DATA: gv_url TYPE string VALUE 'https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/'.
DATA: go_http_client TYPE REF TO  if_web_http_client.
```

5. Your code should now look like this.
<br>![](/exercises/ex2/images/02_01_0050.png)

6. Next, implement the CONSTRUCTOR method.  Here we will leverage the SAP delivered HTTP connectivity classes in order to estabish a connection to an external HTTP endpoint.
```abap
go_http_client = cl_web_http_client_manager=>create_by_http_destination(
            i_destination = cl_http_destination_provider=>create_by_url( gv_url ) ).
```

7. Your code should now look like this.
<br>![](/exercises/ex2/images/02_01_0070.png)

8.	Open the browser and go to the following URL and click “Log On“. If you don't have an account, please create one.
https://api.sap.com/api/API_BANKDETAIL_SRV/resource 
<br>![](/exercises/ex2/images/02_01_0080.png)

9.	Click “Show API Key“.
<br>![](/exercises/ex2/images/02_01_0090.png)

10. Click “Copy Key and Close“.  Copy this key somewhere to use in the next step.
<br>![](/exercises/ex2/images/02_01_0100.png)

11. Next, implement the GET_BANK_DETAILS method.  In this method, get the request object and set the header fields with the content types. Adjust the URI to include the full end point and query parameters.  Also make sure that you update the APIKey value. Finally, execute the request and get the response passing it tot he returning parameter of the method.
```abap
DATA(lo_request) = go_http_client->get_http_request( ).

lo_request->set_header_fields( VALUE #(
   (  name = 'Content-Type' value = 'application/json' )
   (  name = 'Accept' value = 'application/json' )
   (  name = 'APIKey' value = '<insert API key here>') ) ).

lo_request->set_uri_path( 
   i_uri_path = gv_url && 'API_BANKDETAIL_SRV/A_BankDetail?$top=25&$format=json'  ).

TRY.
  DATA(lv_response) = go_http_client->execute( i_method = if_web_http_client=>get )->get_text(  ).
      CATCH cx_web_http_client_error.
ENDTRY.

r_json = lv_response.
```

12.	Your code should now look like the following.
<br>![](/exercises/ex2/images/02_01_0120.png)

13.	Save and activate your work.
<br>![](/exercises/ex2/images/02_01_0130.png)

## Exercise 2.2 Update Hello World 

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
