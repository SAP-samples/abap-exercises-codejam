# Exercise 01 - Template

In this exercise...


## Exercise 1.1: Hello World

After completing these steps...

1. This this thing right here in this image

![](/exercises/ex1/images/0010.png)

2. Insert this code here

```DATA(lt_params) = request->get_form_fields(  ).

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
                             | on {  cl_abap_context_info=>get_system_date(  ) DATE = ENVIRONMENT } | && 
                             | at { cl_abap_context_info=>get_system_time(  ) TIME = ENVIRONMENT } | ).
  WHEN OTHERS.
   response->set_status( i_code = 400
                         i_reason = 'Bad request').
    ENDCASE.
```

## Summary

You've now enabled...