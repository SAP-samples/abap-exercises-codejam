# Exercise 1 - Hello World

In this exercise, we will create a very simple HTTP service in ABAP which can then be called from the browser.

## Exercise 1.1 Hello World

After completing these steps...

1. Right-click on your package and choose “New“, “Other ABAP Repository Object“.
<br>![](/exercises/ex1/images/01_01_0010.png)

2. Expand “Connectivity” folder and choose “HTTP Service”.  Click “Next”.
<br>![](/exercises/ex1/images/01_01_0020.png)

3.	Name the service as ZHELLO_WORLD_XXX where XXX is your group number.  Give a meaningful description.  Leave the default name for the Handler Class.  Click “Next”.  Make sure to replace XXX with your group number.
<br>![](/exercises/ex1/images/01_01_0030.png)

4.	Click “Finish”. 
<br>![](/exercises/ex1/images/01_01_0040.png)

5.	Click on “Handler Class“.
<br>![](/exercises/ex1/images/01_01_0050.png)

6.	The shell of the handler class is then shown.  Here we need to create the implementation for the handle_request method.
<br>![](/exercises/ex1/images/01_01_0060.png)

7.	Insert this line of code in the HANDLE_REQUEST method.
```abap
response->set_text( |Hello World! | ). 
```

8.	Your method should now look like this. 
<br>![](/exercises/ex1/images/01_01_0080.png)

9.	Save and activate your work.
<br>![](/exercises/ex1/images/01_01_0090.png)

10.	Return to the HTTP Service definition and click on URL
<br>![](/exercises/ex1/images/01_01_0100.png)

11.	The browser should open where you may be asked to log in.  Log in and you should get Hello World! Congratulations!
<br>![](/exercises/ex1/images/01_01_0110.png)



## Exercise 1.2 Extending Hello World

After completing these steps...

1.	Return to the handler class and modify the hand_request method as shown here.  Call the get_form_fields method of the request object to get URL parameters, then check for specific commands such as “hello“ and “timestamp“.
```abap
DATA(lt_params) = request->get_form_fields(  ).
READ TABLE lt_params REFERENCE INTO DATA(lr_params) WITH KEY name = 'cmd'.
  IF sy-subrc <> 0.
    response->set_status( i_code = 400
                     i_reason = 'Bad request').
    RETURN.
  ENDIF.
  CASE lr_params->value.
      WHEN `hello`.
        response->set_text( |Hello World! | ).
      WHEN `timestamp`.
        response->set_text( |Hello World! application executed by {
                             cl_abap_context_info=>get_user_technical_name(  ) } | &&
                              | on {  cl_abap_context_info=>get_system_date() DATE = ENVIRONMENT } | &&
                              | at { cl_abap_context_info=>get_system_time(  ) TIME = ENVIRONMENT } | ).
      WHEN OTHERS.
      response->set_status( i_code = 400 i_reason = 'Bad request').
  ENDCASE.
```

2.	You code should now look like this.
![](/exercises/ex1/images/01_02_0020.png)

3.	Save and activate your work.
![](/exercises/ex1/images/01_02_0030.png)

4.	Return to the HTTP Service definition and click on URL.
![](/exercises/ex1/images/01_02_0040.png)

5.	You should get a 400 error because now your service is expecting a URL parameter to determine what it needs to do.  
![](/exercises/ex1/images/01_02_0050.png)

6.	Go to the URL in the browser windown and add the parameter as shown.
![](/exercises/ex1/images/01_02_0060.png)

7.	Now you should once again get the “Hello World!“.
![](/exercises/ex1/images/01_02_0070.png)

8.	Finally, change the cmd parameter value in the URL to “timestamp“ and hit enter.
![](/exercises/ex1/images/01_02_0080.png)

9.	Now your service should return something a bit more.
![](/exercises/ex1/images/01_02_0090.png)

## Summary

You've now...

Continue to - [Exercise 2 - Consuming Services via HTTP ](../ex2/README.md)