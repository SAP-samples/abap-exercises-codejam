# Exercise 3 - ABAP RESTful Application Programming Model - Unmanaged

In this exercise we will explore the new ABAP RESTful Application Programming Model, specifically the Unmanaged Scenario.  The unmanaged scenario provides a framework for the developer to have full control over the CRUD operations of their application. It is designed to allow developers to leverage existing assets, such as existing class methods and function modules. The developer is fully responsible for handling all aspects of the transaction, from validation and locking of the data, to the save mechanism. We will create views over our data model, define behaviors, and service enable the data model as well.  

## Exercise 3.1 Create the Business Object Views

After completing these steps you will have created the Business Object Views over the base database tables.

1.	From your group package, right-click and choose “New“, then “Other ABAP Repository Object“.
<br>![](/exercises/ex3/images/03_01_0010.png)

2.	Expand Core Data Services and choose “Data Definition”.  Click “Next”.
<br>![](/exercises/ex3/images/03_01_0020.png)

3.	Enter the name of the CDS view as Z_I_TRAVEL_U_XXX where XXX is your group number.  Give a meaningful description.  Click “Next”.
<br>![](/exercises/ex3/images/03_01_0030.png)

4.	Click “Next”.
<br>![](/exercises/ex3/images/03_01_0040.png)

5.	Click “Finish”.
<br>![](/exercises/ex3/images/03_01_0050.png)

6.	The CDS definition editor will open with the following code.
<br>![](/exercises/ex3/images/03_01_0060.png)

7.	First, change the name of the sqlViewName annotation as shown here where XXX is your group number.  
<br>![](/exercises/ex3/images/03_01_0070.png)

8.	There is a lot of code to add here. We’ll try to take it by sections.  This code is defining the associations to other tables/views as well as the columns that we want in our view.  In this first section at the top, make sure that you add these annotations if they are not already there. 
<br>![](/exercises/ex3/images/03_01_0080.png)

9.	After the annotations at the top, adjust the define view statement as shown here. Add the word “root“ and change the select from to “/dmo/travel as Travel“.  This is an existing table already in the system.
```abap
define root view Z_I_TRAVEL_U_XXX
  as select from /dmo/travel as Travel

```
10.	Next, add the composition and associations to other tables/views.  In this case we add a composition to the **Booking** view which we have not created yet. Make sure to replace XXX with your group number.
```abap
  composition [0..*] of Z_I_BOOKING_U_XXX as _Booking

  association [0..1] to /DMO/I_Agency    as _Agency    
                     on $projection.AgencyID        = _Agency.AgencyID
  association [0..1] to /DMO/I_Customer  as _Customer  
                     on $projection.CustomerID      = _Customer.CustomerID
  association [0..1] to I_Currency       as _Currency  
                     on $projection.CurrencyCode    = _Currency.Currency
```
11.	Finally add the columns and association references
```abap
{
 key Travel.travel_id     as TravelID,
     Travel.agency_id     as AgencyID,
     Travel.customer_id   as CustomerID,
     Travel.begin_date    as BeginDate,
     Travel.end_date      as EndDate,
     @Semantics.amount.currencyCode: 'CurrencyCode'
     Travel.booking_fee   as BookingFee,
     @Semantics.amount.currencyCode: 'CurrencyCode'
     Travel.total_price   as TotalPrice,
     @Semantics.currencyCode: true
     Travel.currency_code as CurrencyCode,
     Travel.description   as Memo,
     Travel.status        as Status,
     Travel.lastchangedat as LastChangedAt,

     /* public associations */
     _Booking,
     _Agency,
     _Customer,
     _Currency
}
```
12.	Save your work, but do not try to activate at this point as we have not created the **Booking** view yet.
<br>![](/exercises/ex3/images/03_01_0120.png)

13.	Now, use what you have learned and create another view in the same way called Z_I_BOOKING_U_XXX.  As you did before, add the code as shown here. Make sure to use your group nubmer where there is XXX. 
```abap
@AbapCatalog.sqlViewName: 'ZIBOOKING_U_XXX'
@AbapCatalog.compiler.compareFilter: true
@AbapCatalog.preserveKey: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Booking View'

define view Z_I_BOOKING_U_XXX
     as select from /dmo/booking as Booking

  association to parent Z_I_TRAVEL_U_XXX     as _Travel    
                     on  $projection.TravelID = _Travel.TravelID
  composition [0..*] of Z_I_BookingSupplement_U_XXX as _BookSupplement

  association [1..1] to /DMO/I_Customer       as _Customer   
                     on  $projection.CustomerID = _Customer.CustomerID
  association [1..1] to /DMO/I_Carrier        as _Carrier    
                     on  $projection.AirlineID = _Carrier.AirlineID
  association [1..1] to /DMO/I_Connection     as _Connection 
                     on  $projection.AirlineID    = _Connection.AirlineID
                   and $projection.ConnectionID = _Connection.ConnectionID
{
  key Booking.travel_id     as TravelID,
  key Booking.booking_id    as BookingID,
      Booking.booking_date  as BookingDate,
      Booking.customer_id   as CustomerID,
      Booking.carrier_id    as AirlineID,
      Booking.connection_id as ConnectionID,
      Booking.flight_date   as FlightDate,
      @Semantics.amount.currencyCode: 'CurrencyCode'
      Booking.flight_price  as FlightPrice,
      @Semantics.currencyCode: true
      Booking.currency_code as CurrencyCode,
//    _Travel.LastChangedAt as LastChangedAt, -- Take over ETag from parent

      /* public associations */
      _Travel,
      _BookSupplement,
      _Customer,
      _Carrier,
      _Connection
}

```
14. Save your work.
<br>![](/exercises/ex3/images/03_01_0140.png)

15. Use what you have learned and create another view in the same way called Z_I_BOOKINGSUPPLEMENT_U_XXX.  As you did before, add the code as shown here. Make sure to use your group nubmer where there is XXX.
```abap
@AbapCatalog.sqlViewName: 'ZIBOOKSUPP_U_XXX'
@AbapCatalog.compiler.compareFilter: true
@AbapCatalog.preserveKey: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Booking Supplement view - CDS data model'

define view Z_I_BookingSupplement_U_XXX
  as select from /dmo/book_suppl as BookingSupplement

  association        to parent Z_I_Booking_U_XXX as _Booking        on  $projection.TravelID  = _Booking.TravelID
                                                                   and $projection.BookingID = _Booking.BookingID
  association [1..1] to Z_I_Travel_U_XXX        as _Travel         on  $projection.TravelID  = _Travel.TravelID  
  association [1..1] to /DMO/I_Supplement       as _Product        on  $projection.SupplementID = _Product.SupplementID
  association [1..*] to /DMO/I_SupplementText   as _SupplementText on  $projection.SupplementID = _SupplementText.SupplementID

{
  key BookingSupplement.travel_id             as TravelID,

  key BookingSupplement.booking_id            as BookingID,

  key BookingSupplement.booking_supplement_id as BookingSupplementID,

      BookingSupplement.supplement_id         as SupplementID,

      @Semantics.amount.currencyCode: 'CurrencyCode'
      BookingSupplement.price                 as Price,

      @Semantics.currencyCode: true
      BookingSupplement.currency_code         as CurrencyCode,

//      _Booking.LastChangedAt                  as LastChangedAt,

      /* Associations */
      _Booking,
      _Travel,
      _Product,
      _SupplementText
}

```
16. Save your work.
<br>![](/exercises/ex3/images/03_01_0160.png)

17. Select all three of the views that you have created.  
<br>![](/exercises/ex3/images/03_01_0170.png)

18. Right-click on the selection and choose “Activate“.
<br>![](/exercises/ex3/images/03_01_0180.png)


## Exercise 3.2 Create the Business Object Behavior Definition and Implementation

After completing these steps you will have created the Behavior Definition and Implementation for your Business Object Views.  The behavior definition defines what operations are possible for your Business Object. 

1.	Right-click on the Z_I_TRAVEL_U_XXX CDS Artifact and choose “New Behavior Defintion“.
<br>![](/exercises/ex3/images/03_02_0010.png)

2.	Provide the default description, and change the Implementation Type to “Unmanaged”.  Then click “Next”, then “Finish”.
<br>![](/exercises/ex3/images/03_02_0020.png)

3.	You will then see the Behavior Defintion editor. You should see 3 separate behavior definitions here, one for Travel, Bookings, and BookingSupplement. 
<br>![](/exercises/ex3/images/03_02_0030.png)

4.	Let’s modify this definition.  In this case, we don’t want a single implementation class for Travel, Booking, and BookingSupplement.  Instead lets define the implementation classes specifically for each definition.  First remove that from the first line so that it only contains the unmanaged key word.
<br>![](/exercises/ex3/images/03_02_0040.png)

5.	Give aliases to all definitions. Also add these lines for the implementation classes for each definition. Be sure to replace XXX with your group number.
<br>![](/exercises/ex3/images/03_02_0050.png)

6.	Inside the definition, add the following code before the create statement in the first definition.  The field keyword here allows you to set specific attributes of fields, for example, to set as read-only  or as required. 
```abap
field ( read only ) TravelID;
  field ( mandatory ) AgencyID, CustomerID, BeginDate, EndDate;
```
7.	After the delete statement in the first definition, add the following line of code which defines a custom action called *set_status_booked*. 
```abap 
action set_status_booked result [1] $self;
```

8.	In the first definition, after the association keyword, add the following mapping.  This allows us to map columns of the view back to the underlying table columns. We can then use this in the behavior implementation in our MOVE CORRESPONDING statements.
```abap
  mapping for /dmo/travel
  {
    AgencyID = agency_id;
    BeginDate = begin_date;
    BookingFee = booking_fee;
    CurrencyCode = currency_code;
    CustomerID = customer_id;
    EndDate = end_date;
    LastChangedAt = lastchangedat;
    Memo = description;
    Status = status;
    TotalPrice = total_price;
    TravelID = travel_id;
  }

```
9.	Next drop down to the second definition, the definition for Z_I_BOOKING_U_XXX.  Add the following lines of code before the create keyword.
```abap
  field ( read only ) TravelID, BookingID;
  field ( mandatory ) BookingDate, CustomerID, AirlineID, ConnectionID, FlightDate;
```

10.	Now remove the create keyword, since we will handle the creation of bookings in association with the travel. So we only need the update and delete keywords.
<br>![](/exercises/ex3/images/03_02_0100.png)

11. After the association keyword, add the following association and mapping statement.
```abap
  association _Travel;

  mapping for /dmo/booking
  {
    AirlineID = carrier_id;
    BookingDate = booking_date;
    BookingID = booking_id;
    ConnectionID = connection_id;
    CurrencyCode = currency_code;
    CustomerID = customer_id;
    FlightDate = flight_date;
    FlightPrice = flight_price;
    TravelID = travel_id;
  }

```

12. Next, drop down to the Z_I_BOOKINGSUPPLEMENT_XXX definition. Add the following lines of code before the create keyword.
```abap
  field ( read only ) TravelID, BookingID, BookingSupplementID;
  field ( mandatory ) SupplementID, Price;

```
13.	Now remove the create keyword, since we will handle the creation of bookingsupplement in association with the booking. So we only need the update and delete keywords.
<br>![](/exercises/ex3/images/03_02_0130.png)

14.	After the delete keyword, add the following association and mapping statement.
```abap
association _Travel;
mapping for /dmo/book_suppl
  {
    BookingID           = booking_id;
    BookingSupplementID = booking_supplement_id;
    CurrencyCode        = currency_code;
    Price               = price;
    SupplementID        = supplement_id;
    TravelID            = travel_id;
  }

```
15. The completed behavior definition should now look very similar to this.
```abap
implementation unmanaged;

define behavior for Z_I_TRAVEL_U_XXX alias travel
   implementation in class z_bp_i_travel_u_xxx unique
//late numbering
//lock master
//etag master <field_name>
{

  field ( read only ) TravelID;
  field ( mandatory ) AgencyID, CustomerID, BeginDate, EndDate;

  create;
  update;
  delete;

  action set_status_booked result [1] $self;

  association _Booking { create; }
  
  mapping for /dmo/travel
  {
    AgencyID = agency_id;
    BeginDate = begin_date;
    BookingFee = booking_fee;
    CurrencyCode = currency_code;
    CustomerID = customer_id;
    EndDate = end_date;
    LastChangedAt = lastchangedat;
    Memo = description;
    Status = status;
    TotalPrice = total_price;
    TravelID = travel_id;
  }

}


define behavior for Z_I_BOOKING_U_XXX alias booking
   implementation in class z_bp_i_booking_u_XXX unique
//late numbering
//lock dependent( <local_field_name> = <target_field_name> )
//etag master <field_name>
{
  field ( read only ) TravelID, BookingID;
  field ( mandatory ) BookingDate, CustomerID, AirlineID, ConnectionID, FlightDate;

  update;
  delete;

  association _BookSupplement abbreviation _Supplement { create; }
  association _Travel;
  
  mapping for /dmo/booking
  {
    AirlineID = carrier_id;
    BookingDate = booking_date;
    BookingID = booking_id;
    ConnectionID = connection_id;
    CurrencyCode = currency_code;
    CustomerID = customer_id;
    FlightDate = flight_date;
    FlightPrice = flight_price;
    TravelID = travel_id;
  }
}

// behavior defintion for the BOOKING SUPPLEMENT sub node
define behavior for Z_I_BookingSupplement_U_XXX alias bookingsupplement
implementation in class z_bp_i_bookingsupplement_U_XXX unique

{

  field ( read only ) TravelID, BookingID, BookingSupplementID;
  field ( mandatory ) SupplementID, Price;


  update;
  delete;
  
  association _Travel;

  mapping for /dmo/book_suppl
  {
    BookingID           = booking_id;
    BookingSupplementID = booking_supplement_id;
    CurrencyCode        = currency_code;
    Price               = price;
    SupplementID        = supplement_id;
    TravelID            = travel_id;
  }
}

```

16.	Save and Activate your work.  Yes, you have not created the behavior implementation classes yet, but that is ok. You can still activate at this point.
<br>![](/exercises/ex3/images/03_02_0160.png)

17.	Now we will create Behavior Implementations for *Travel, Booking and BookingSupplement*. Right-click on your behavior definition and choose, “New Behavior Implementation“.
<br>![](/exercises/ex3/images/03_02_0170.png)

18.	Give the name of the ABAP Class as Z_BP_I_TRAVEL_U_XXX.  Be sure to replace XXX with your group number. Technically, we refer to this as a “Behavior Pool“.  Click “Next“ and then “Finish“.
<br>![](/exercises/ex3/images/03_02_0180.png)

19.	The class editor will then be shown.   Click the “Local Types“ tab.
<br>![](/exercises/ex3/images/03_02_0190.png)

20.	We will create a set of local classes here which will implement the behavior. 
<br>![](/exercises/ex3/images/03_02_0200.png)

21.	Enter the class definition for lhc_travel as shown here.  This class definition includes methods for all of the require operations.
```abap
**********************************************************************
*
* Handler class for managing travels
*
**********************************************************************
CLASS lhc_travel DEFINITION INHERITING FROM cl_abap_behavior_handler.

  PRIVATE SECTION.
    TYPES:
         tt_travel_update TYPE TABLE FOR UPDATE /dmo/i_travel_u.

    METHODS:
      create_travel     FOR MODIFY
        IMPORTING it_travel_create FOR CREATE travel,
      update_travel     FOR MODIFY
        IMPORTING it_travel_update FOR UPDATE travel,
      delete_travel     FOR MODIFY
        IMPORTING it_travel_delete FOR DELETE travel,
      read_travel       FOR READ
        IMPORTING it_travel FOR READ travel
        RESULT    et_travel,

      set_travel_status FOR MODIFY
        IMPORTING it_travel_set_status_booked FOR ACTION travel~set_status_booked
        RESULT    et_travel_set_status_booked,

      read_booking_ba   FOR READ
        IMPORTING it_travel  FOR READ travel\_Booking
        FULL iv_full_requested
        RESULT    et_booking
        LINK et_link_table,

      cba_booking       FOR MODIFY
        IMPORTING it_booking_create_ba FOR CREATE travel\_booking.

ENDCLASS.


CLASS lhc_travel IMPLEMENTATION.
ENDCLASS.

```

22.	Next, inside the class implementation section, define the implementation for the *CREATE_TRAVEL* method as shown. 
```abap
**********************************************************************
*
* Create travel instances
*
**********************************************************************
  METHOD create_travel.

    DATA lt_messages   TYPE /dmo/if_flight_legacy=>tt_message.
    DATA ls_travel_in  TYPE /dmo/travel.
    DATA ls_travel_out TYPE /dmo/travel.

    LOOP AT it_travel_create ASSIGNING FIELD-SYMBOL(<fs_travel_create>).

      ls_travel_in = CORRESPONDING #( <fs_travel_create> MAPPING FROM ENTITY USING CONTROL ).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_CREATE'
        EXPORTING
          is_travel   = CORRESPONDING /dmo/if_flight_legacy=>ts_travel_in( ls_travel_in )
        IMPORTING
          es_travel   = ls_travel_out
          et_messages = lt_messages.

      IF lt_messages IS INITIAL.
        INSERT VALUE #( %cid = <fs_travel_create>-%cid  travelid = ls_travel_out-travel_id )
                       INTO TABLE mapped-travel.
      ELSE.
        /dmo/cl_travel_auxiliary=>handle_travel_messages(
          EXPORTING
            iv_cid       = <fs_travel_create>-%cid
            it_messages  = lt_messages
          CHANGING
            failed       = failed-travel
            reported     = reported-travel ).
      ENDIF.

    ENDLOOP.

  ENDMETHOD.

```

23.	After the *CREATE_TRAVEL* method, implement the *UPDATE_TRAVEL* method as shown here.
```abap
**********************************************************************
*
* Update data of existing travel instances
*
**********************************************************************
  METHOD update_travel.

    DATA lt_messages    TYPE /dmo/if_flight_legacy=>tt_message.
    DATA ls_travel      TYPE /dmo/travel.
    DATA ls_travel_inx TYPE /dmo/if_flight_legacy=>ts_travel_inx. 

    LOOP AT it_travel_update ASSIGNING FIELD-SYMBOL(<fs_travel_update>).

      ls_travel = CORRESPONDING #( <fs_travel_update> MAPPING FROM ENTITY ).

      CLEAR ls_travel_inx.
      ls_travel_inx-travel_id = <fs_travel_update>-TravelID.
      ls_travel_inx-agency_id     = xsdbool( <fs_travel_update>-%control-agencyid     = if_abap_behv=>mk-on ).
      ls_travel_inx-customer_id   = xsdbool( <fs_travel_update>-%control-customerid   = if_abap_behv=>mk-on ).
      ls_travel_inx-begin_date    = xsdbool( <fs_travel_update>-%control-begindate    = if_abap_behv=>mk-on ).
      ls_travel_inx-end_date      = xsdbool( <fs_travel_update>-%control-enddate      = if_abap_behv=>mk-on ).
      ls_travel_inx-booking_fee   = xsdbool( <fs_travel_update>-%control-bookingfee   = if_abap_behv=>mk-on ).
      ls_travel_inx-total_price   = xsdbool( <fs_travel_update>-%control-totalprice   = if_abap_behv=>mk-on ).
      ls_travel_inx-currency_code = xsdbool( <fs_travel_update>-%control-currencycode = if_abap_behv=>mk-on ).
      ls_travel_inx-description   = xsdbool( <fs_travel_update>-%control-memo         = if_abap_behv=>mk-on ).
      ls_travel_inx-status        = xsdbool( <fs_travel_update>-%control-status       = if_abap_behv=>mk-on ).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
        EXPORTING
          is_travel   = CORRESPONDING /dmo/if_flight_legacy=>ts_travel_in( ls_travel )
          is_travelx  = ls_travel_inx
        IMPORTING
          et_messages = lt_messages.

      /dmo/cl_travel_auxiliary=>handle_travel_messages(
        EXPORTING
          iv_cid       = <fs_travel_update>-%cid_ref
          iv_travel_id = <fs_travel_update>-travelid
          it_messages  = lt_messages
        CHANGING
          failed   = failed-travel
          reported = reported-travel ).

    ENDLOOP.

  ENDMETHOD.

```

24.	After the *UPDATE_TRAVEL* method, implement the *DELETE_TRAVEL* method.
```abap
**********************************************************************
*
* Delete of travel instances
*
**********************************************************************
  METHOD delete_travel.

    DATA lt_messages TYPE /dmo/if_flight_legacy=>tt_message.

    LOOP AT it_travel_delete ASSIGNING FIELD-SYMBOL(<fs_travel_delete>).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_DELETE'
        EXPORTING
          iv_travel_id = <fs_travel_delete>-travelid
        IMPORTING
          et_messages  = lt_messages.

      /dmo/cl_travel_auxiliary=>handle_travel_messages(
        EXPORTING
          iv_cid       = <fs_travel_delete>-%cid_ref
          iv_travel_id = <fs_travel_delete>-travelid
          it_messages  = lt_messages
        CHANGING
          failed       = failed-travel
          reported     = reported-travel ).

    ENDLOOP.

  ENDMETHOD.


```

25. Next, implement the READ_TRAVEL method.
```abap
**********************************************************************
*
* Read travel data from buffer
*
**********************************************************************
  METHOD read_travel.
    DATA: ls_travel_out TYPE /dmo/travel,
          lt_message    TYPE /dmo/if_flight_legacy=>tt_message.

    LOOP AT it_travel INTO DATA(ls_travel_to_read).
      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id = ls_travel_to_read-travelid
        IMPORTING
          es_travel    = ls_travel_out
          et_messages  = lt_message.

      IF lt_message IS INITIAL.
        "fill result parameter with flagged fields
        INSERT
            VALUE #( travelid      = ls_travel_out-travel_id
                     agencyid      = COND #( WHEN ls_travel_to_read-%control-AgencyID      = cl_abap_behv=>flag_changed THEN ls_travel_out-agency_id )
                     customerid    = COND #( WHEN ls_travel_to_read-%control-CustomerID    = cl_abap_behv=>flag_changed THEN ls_travel_out-customer_id )
                     begindate     = COND #( WHEN ls_travel_to_read-%control-BeginDate     = cl_abap_behv=>flag_changed THEN ls_travel_out-begin_date )
                     enddate       = COND #( WHEN ls_travel_to_read-%control-EndDate       = cl_abap_behv=>flag_changed THEN ls_travel_out-end_date )
                     bookingfee    = COND #( WHEN ls_travel_to_read-%control-BookingFee    = cl_abap_behv=>flag_changed THEN ls_travel_out-booking_fee )
                     totalprice    = COND #( WHEN ls_travel_to_read-%control-TotalPrice    = cl_abap_behv=>flag_changed THEN ls_travel_out-total_price )
                     currencycode  = COND #( WHEN ls_travel_to_read-%control-CurrencyCode  = cl_abap_behv=>flag_changed THEN ls_travel_out-currency_code )
                     memo          = COND #( WHEN ls_travel_to_read-%control-Memo          = cl_abap_behv=>flag_changed THEN ls_travel_out-description )
                     status        = COND #( WHEN ls_travel_to_read-%control-Status        = cl_abap_behv=>flag_changed THEN ls_travel_out-status )
                     lastchangedat = COND #( WHEN ls_travel_to_read-%control-LastChangedAt = cl_abap_behv=>flag_changed THEN ls_travel_out-lastchangedat ) )
            INTO TABLE et_travel.


      ELSE.
        "fill failed table in case of error
        failed-travel = VALUE #(  BASE failed-travel
                                  FOR msg IN lt_message (  %key = ls_travel_to_read-%key
                                                           %fail-cause = COND #( WHEN msg-msgty = 'E' AND msg-msgno = '016'
                                                                                 THEN if_abap_behv=>cause-not_found
                                                                                 ELSE if_abap_behv=>cause-unspecific ) ) ).

      ENDIF.

    ENDLOOP.

  ENDMETHOD.

```

26. Implement the *SET_STATUS_BOOKED* method.
```abap
**********************************************************************
*
* Implement travel action(s) (in our case: for setting travel status)
*
**********************************************************************

  METHOD set_travel_status.

    DATA lt_messages TYPE /dmo/if_flight_legacy=>tt_message.
    DATA ls_travel_out TYPE /dmo/travel.

    CLEAR et_travel_set_status_booked.

    LOOP AT it_travel_set_status_booked ASSIGNING FIELD-SYMBOL(<fs_travel_set_status_booked>).

      DATA(lv_travelid) = <fs_travel_set_status_booked>-travelid.

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_SET_BOOKING'
        EXPORTING
          iv_travel_id = lv_travelid
        IMPORTING
          et_messages  = lt_messages.

      IF lt_messages IS INITIAL.
        CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
          EXPORTING
            iv_travel_id = lv_travelid
          IMPORTING
            es_travel    = ls_travel_out.

        APPEND VALUE #( travelid = lv_travelid
                        %param   = VALUE #( travelid      = lv_travelid
                                            agencyid      = ls_travel_out-agency_id
                                            customerid    = ls_travel_out-customer_id
                                            begindate     = ls_travel_out-begin_date
                                            enddate       = ls_travel_out-end_date
                                            bookingfee    = ls_travel_out-booking_fee
                                            totalprice    = ls_travel_out-total_price
                                            currencycode  = ls_travel_out-currency_code
                                            memo          = ls_travel_out-description
                                            status        = ls_travel_out-status
                                            lastchangedat = ls_travel_out-lastchangedat )
                      ) TO et_travel_set_status_booked.
      ELSE.

        /dmo/cl_travel_auxiliary=>handle_travel_messages(
          EXPORTING
            iv_cid       = <fs_travel_set_status_booked>-%cid_ref
            iv_travel_id = lv_travelid
            it_messages  = lt_messages
          CHANGING
            failed       = failed-travel
            reported     = reported-travel ).
      ENDIF.

    ENDLOOP.

  ENDMETHOD.

```
27. Implement the CBA_BOOKING method. 
```abap
**********************************************************************
*
* Create associated booking instances
*
**********************************************************************
  METHOD cba_booking.
    DATA lt_messages        TYPE /dmo/if_flight_legacy=>tt_message.
    DATA lt_booking_old     TYPE /dmo/if_flight_legacy=>tt_booking.
    DATA ls_booking         TYPE /dmo/booking.
    DATA lv_last_booking_id TYPE /dmo/booking_id VALUE '0'.

    LOOP AT it_booking_create_ba ASSIGNING FIELD-SYMBOL(<fs_booking_create_ba>).

      DATA(lv_travelid) = <fs_booking_create_ba>-travelid.

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id = lv_travelid
        IMPORTING
          et_booking   = lt_booking_old
          et_messages  = lt_messages.

      IF lt_messages IS INITIAL.

        IF lt_booking_old IS NOT INITIAL.
          lv_last_booking_id = lt_booking_old[ lines( lt_booking_old ) ]-booking_id.
        ENDIF.

        LOOP AT <fs_booking_create_ba>-%target ASSIGNING FIELD-SYMBOL(<fs_booking_create>).
          ls_booking = CORRESPONDING #( <fs_booking_create> MAPPING FROM ENTITY USING CONTROL ) .

          lv_last_booking_id += 1.
          ls_booking-booking_id = lv_last_booking_id.

          CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
            EXPORTING
              is_travel   = VALUE /dmo/if_flight_legacy=>ts_travel_in( travel_id = lv_travelid )
              is_travelx  = VALUE /dmo/if_flight_legacy=>ts_travel_inx( travel_id = lv_travelid )
              it_booking  = VALUE /dmo/if_flight_legacy=>tt_booking_in( ( CORRESPONDING #( ls_booking ) ) )
              it_bookingx = VALUE /dmo/if_flight_legacy=>tt_booking_inx( ( booking_id  = ls_booking-booking_id
                                                                           action_code = /dmo/if_flight_legacy=>action_code-create ) )
            IMPORTING
              et_messages = lt_messages.

          IF lt_messages IS INITIAL.
            INSERT VALUE #( %cid = <fs_booking_create>-%cid  travelid = lv_travelid  bookingid = ls_booking-booking_id )
                INTO TABLE mapped-booking.
          ELSE.

            LOOP AT lt_messages INTO DATA(ls_message) WHERE msgty = 'E' OR msgty = 'A'.
              INSERT VALUE #( %cid = <fs_booking_create>-%cid ) INTO TABLE failed-booking.

              INSERT VALUE #(
                    %cid     = <fs_booking_create>-%cid
                    travelid = <fs_booking_create>-TravelID
                    %msg     = new_message(
                              id       = ls_message-msgid
                              number   = ls_message-msgno
                              severity = if_abap_behv_message=>severity-error
                              v1       = ls_message-msgv1
                              v2       = ls_message-msgv2
                              v3       = ls_message-msgv3
                              v4       = ls_message-msgv4
                        )
               )
              INTO TABLE reported-booking.

            ENDLOOP.

          ENDIF.

        ENDLOOP.

      ELSE.

        /dmo/cl_travel_auxiliary=>handle_travel_messages(
         EXPORTING
           iv_cid       = <fs_booking_create_ba>-%cid_ref
           iv_travel_id = lv_travelid
           it_messages  = lt_messages
         CHANGING
           failed       = failed-travel
           reported     = reported-travel ).

      ENDIF.

    ENDLOOP.

  ENDMETHOD.

```
28.	Implement the READ_BOOKING_BA method.
```abap
**********************************************************************
*
* Read booking data by association from buffer
*
**********************************************************************
  METHOD read_booking_ba.
    DATA: ls_travel_out  TYPE /dmo/travel,
          lt_booking_out TYPE /dmo/if_flight_legacy=>tt_booking,
          lt_message     TYPE /dmo/if_flight_legacy=>tt_message.


    LOOP AT it_travel ASSIGNING FIELD-SYMBOL(<fs_travel_rba>).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id = <fs_travel_rba>-travelid
        IMPORTING
          es_travel    = ls_travel_out
          et_booking   = lt_booking_out
          et_messages  = lt_message.

      IF lt_message IS INITIAL.
        LOOP AT lt_booking_out ASSIGNING FIELD-SYMBOL(<fs_booking>).
          "fill link table with key fields
          INSERT
           VALUE #( source-%key = <fs_travel_rba>-%key
                                   target-%key = VALUE #( TravelID = <fs_booking>-travel_id
                                                          BookingID = <fs_booking>-booking_id ) )
          INTO TABLE et_link_table.

          "fill result parameter with flagged fields
          IF iv_full_requested = abap_true.
            INSERT
                VALUE #( travelid      = COND #( WHEN <fs_travel_rba>-%control-TravelID      = cl_abap_behv=>flag_changed THEN <fs_booking>-travel_id )
                         bookingid     = COND #( WHEN <fs_travel_rba>-%control-BookingID     = cl_abap_behv=>flag_changed THEN <fs_booking>-booking_id )
                         bookingdate   = COND #( WHEN <fs_travel_rba>-%control-BookingDate   = cl_abap_behv=>flag_changed THEN <fs_booking>-booking_date )
                         customerid    = COND #( WHEN <fs_travel_rba>-%control-CustomerID    = cl_abap_behv=>flag_changed THEN <fs_booking>-customer_id )
                         airlineid     = COND #( WHEN <fs_travel_rba>-%control-AirlineID     = cl_abap_behv=>flag_changed THEN <fs_booking>-carrier_id )
                         connectionid  = COND #( WHEN <fs_travel_rba>-%control-ConnectionID  = cl_abap_behv=>flag_changed THEN <fs_booking>-connection_id )
                         flightdate    = COND #( WHEN <fs_travel_rba>-%control-FlightDate    = cl_abap_behv=>flag_changed THEN <fs_booking>-flight_date )
                         flightprice   = COND #( WHEN <fs_travel_rba>-%control-FlightPrice   = cl_abap_behv=>flag_changed THEN <fs_booking>-flight_price )
                         currencycode  = COND #( WHEN <fs_travel_rba>-%control-CurrencyCode  = cl_abap_behv=>flag_changed THEN <fs_booking>-currency_code )
*                         lastchangedat = COND #( WHEN <fs_travel_rba>-%control-LastChangedAt = cl_abap_behv=>flag_changed THEN ls_travel_out-lastchangedat )
)
                         INTO TABLE et_booking.
          ENDIF.
        ENDLOOP.

      ELSE.
        "fill failed table in case of error
        failed-travel = VALUE #(  BASE failed-travel
                                    FOR msg IN lt_message (  %key = <fs_travel_rba>-TravelID
                                                             %fail-cause = COND #( WHEN msg-msgty = 'E' AND msg-msgno = '016'
                                                                                   THEN if_abap_behv=>cause-not_found
                                                                                   ELSE if_abap_behv=>cause-unspecific ) ) ).
      ENDIF.

    ENDLOOP.

  ENDMETHOD.

```
29.	Next we have one final local class to define. This is our saver class. Enter the code as shown here.
```abap
**********************************************************************
*
* Saver class implements the save sequence for data persistence
*
**********************************************************************
CLASS lsc_saver DEFINITION INHERITING FROM cl_abap_behavior_saver.
  PROTECTED SECTION.
    METHODS finalize          REDEFINITION.
    METHODS check_before_save REDEFINITION.
    METHODS save              REDEFINITION.
    METHODS cleanup           REDEFINITION.
ENDCLASS.
```

30.	Enter the following code for class implementation.  Here we are only adding to the SAVE and CLEANUP methods.
```abap
CLASS lsc_saver IMPLEMENTATION.
  METHOD finalize.
  ENDMETHOD.

  METHOD check_before_save.
  ENDMETHOD.

  METHOD save.
    CALL FUNCTION '/DMO/FLIGHT_TRAVEL_SAVE'.
  ENDMETHOD.

  METHOD cleanup.
    CALL FUNCTION '/DMO/FLIGHT_TRAVEL_INITIALIZE'.
  ENDMETHOD.
ENDCLASS.

```

31.	Save and activate your work.
<br>![](/exercises/ex3/images/03_02_0310.png)

32.	Use what you have learned and create another ABAP Class for a another Behavior Pool and call it Z_BP_I_BOOKING_U_XXX.  Copy the following code to the “Local Types“ tab.  As you can see, this Behavior Pool contains implementations for the READ, UPDATE and DELETE operations for **Bookings** as well as READ and CREATE for **BookingSupplement**.  
```abap
**********************************************************************
*
* Handler class for managing bookings
*
**********************************************************************
CLASS lhc_booking DEFINITION INHERITING FROM cl_abap_behavior_handler.

  PRIVATE SECTION.

    TYPES:
      tt_booking_update           TYPE TABLE FOR UPDATE  /dmo/i_booking_u,
      tt_bookingsupplement_create TYPE TABLE FOR CREATE  /dmo/i_bookingsupplement_u.

    METHODS:
      update_booking FOR MODIFY
        IMPORTING it_booking_update FOR UPDATE booking,
      delete_booking FOR MODIFY
        IMPORTING it_booking_delete FOR DELETE booking,
      read_booking FOR READ
        IMPORTING it_booking_read FOR READ booking
        RESULT    et_booking,

      cba_supplement FOR MODIFY
        IMPORTING it_supplement_create_ba FOR CREATE booking\_booksupplement,
      read_supplement_ba FOR READ
        IMPORTING it_booking   FOR READ booking\_booksupplement FULL ev_full_requested
        RESULT    et_booksuppl
        LINK et_link_table.


ENDCLASS.


CLASS lhc_booking IMPLEMENTATION.

**********************************************************************
*
* Implements the UPDATE operation for a set of booking instances
*
**********************************************************************
  METHOD update_booking.

    DATA lt_messages TYPE /dmo/if_flight_legacy=>tt_message.
    DATA it_booking TYPE /dmo/booking.
    data ls_booking_inx TYPE /dmo/if_flight_legacy=>ts_booking_inx.

    LOOP AT it_booking_update ASSIGNING FIELD-SYMBOL(<fs_booking_update>).
      it_booking = CORRESPONDING #( <fs_booking_update> MAPPING FROM ENTITY ).

    CLEAR ls_booking_inx.
    ls_booking_inx-booking_id    = <fs_booking_update>-bookingid.
    ls_booking_inx-action_code   = /dmo/if_flight_legacy=>action_code-update.
    ls_booking_inx-booking_date  = xsdbool( <fs_booking_update>-%control-bookingdate  = if_abap_behv=>mk-on ).
    ls_booking_inx-customer_id   = xsdbool( <fs_booking_update>-%control-customerid   = if_abap_behv=>mk-on ).
    ls_booking_inx-carrier_id    = xsdbool( <fs_booking_update>-%control-airlineid    = if_abap_behv=>mk-on ).
    ls_booking_inx-connection_id = xsdbool( <fs_booking_update>-%control-connectionid = if_abap_behv=>mk-on ).
    ls_booking_inx-flight_date   = xsdbool( <fs_booking_update>-%control-flightdate   = if_abap_behv=>mk-on ).
    ls_booking_inx-flight_price  = xsdbool( <fs_booking_update>-%control-flightprice  = if_abap_behv=>mk-on ).
    ls_booking_inx-currency_code = xsdbool( <fs_booking_update>-%control-currencycode = if_abap_behv=>mk-on ).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
        EXPORTING
          is_travel   = VALUE /dmo/if_flight_legacy=>ts_travel_in( travel_id = <fs_booking_update>-travelid )
          is_travelx  = VALUE /dmo/if_flight_legacy=>ts_travel_inx( travel_id = <fs_booking_update>-travelid )
          it_booking  = VALUE /dmo/if_flight_legacy=>tt_booking_in( ( CORRESPONDING #( it_booking ) ) )
          it_bookingx = VALUE /dmo/if_flight_legacy=>tt_booking_inx( ( ls_booking_inx ) )
        IMPORTING
          et_messages = lt_messages.


      /dmo/cl_travel_auxiliary=>handle_booking_messages(
        EXPORTING
          iv_cid        = <fs_booking_update>-%cid_ref
          iv_travel_id  = <fs_booking_update>-travelid
          iv_booking_id = <fs_booking_update>-bookingid
          it_messages   = lt_messages
        CHANGING
          failed   = failed-booking
          reported = reported-booking ).

    ENDLOOP.

  ENDMETHOD.

**********************************************************************
*
* Implements the DELETE operation for a set of booking instances
*
**********************************************************************
  METHOD delete_booking.

    DATA lt_messages TYPE /dmo/if_flight_legacy=>tt_message.

    LOOP AT it_booking_delete INTO DATA(ls_booking_delete).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
        EXPORTING
          is_travel   = VALUE /dmo/if_flight_legacy=>ts_travel_in( travel_id = ls_booking_delete-travelid )
          is_travelx  = VALUE /dmo/if_flight_legacy=>ts_travel_inx( travel_id = ls_booking_delete-travelid )
          it_booking  = VALUE /dmo/if_flight_legacy=>tt_booking_in( ( booking_id = ls_booking_delete-bookingid ) )
          it_bookingx = VALUE /dmo/if_flight_legacy=>tt_booking_inx( ( booking_id  = ls_booking_delete-bookingid
                                                                       action_code = /dmo/if_flight_legacy=>action_code-delete ) )
        IMPORTING
          et_messages = lt_messages.

      IF lt_messages IS NOT INITIAL.

        /dmo/cl_travel_auxiliary=>handle_booking_messages(
         EXPORTING
           iv_cid        = ls_booking_delete-%cid_ref
           iv_travel_id  = ls_booking_delete-travelid
           iv_booking_id = ls_booking_delete-bookingid
           it_messages   = lt_messages
         CHANGING
           failed   = failed-booking
           reported = reported-booking ).

      ENDIF.

    ENDLOOP.

  ENDMETHOD.

**********************************************************************
*
* Read booking data from buffer
*
**********************************************************************

  METHOD read_booking.

    DATA: ls_travel_out  TYPE /dmo/travel,
          lt_booking_out TYPE /dmo/if_flight_legacy=>tt_booking,
          lt_message     TYPE /dmo/if_flight_legacy=>tt_message.

    "Only one function call for each requested travelid
    LOOP AT it_booking_read ASSIGNING FIELD-SYMBOL(<fs_travel_read>)
                            GROUP BY <fs_travel_read>-travelid .

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id = <fs_travel_read>-travelid
        IMPORTING
          es_travel    = ls_travel_out
          et_booking   = lt_booking_out
          et_messages  = lt_message.

      IF lt_message IS INITIAL.
        "For each travelID find the requested bookings
        LOOP AT GROUP <fs_travel_read> ASSIGNING FIELD-SYMBOL(<fs_booking_read>).

          READ TABLE lt_booking_out INTO DATA(ls_booking) WITH KEY travel_id  = <fs_booking_read>-%key-TravelID
                                                                   booking_id = <fs_booking_read>-%key-BookingID .
          "if read was successfull
          IF sy-subrc = 0.

            "fill result parameter with flagged fields
            INSERT
              VALUE #( travelid      =   ls_booking-travel_id
                       bookingid     =   ls_booking-booking_id
                       bookingdate   =   COND #( WHEN <fs_booking_read>-%control-BookingDate      = cl_abap_behv=>flag_changed THEN ls_booking-booking_date   )
                       customerid    =   COND #( WHEN <fs_booking_read>-%control-CustomerID       = cl_abap_behv=>flag_changed THEN ls_booking-customer_id    )
                       airlineid     =   COND #( WHEN <fs_booking_read>-%control-AirlineID        = cl_abap_behv=>flag_changed THEN ls_booking-carrier_id     )
                       connectionid  =   COND #( WHEN <fs_booking_read>-%control-ConnectionID     = cl_abap_behv=>flag_changed THEN ls_booking-connection_id  )
                       flightdate    =   COND #( WHEN <fs_booking_read>-%control-FlightDate       = cl_abap_behv=>flag_changed THEN ls_booking-flight_date    )
                       flightprice   =   COND #( WHEN <fs_booking_read>-%control-FlightPrice      = cl_abap_behv=>flag_changed THEN ls_booking-flight_price   )
                       currencycode  =   COND #( WHEN <fs_booking_read>-%control-CurrencyCode     = cl_abap_behv=>flag_changed THEN ls_booking-currency_code  )
*                       lastchangedat =   COND #( WHEN <fs_booking_read>-%control-LastChangedAt    = cl_abap_behv=>flag_changed THEN ls_travel_out-lastchangedat   )
                     ) INTO TABLE et_booking.
          ELSE.
            "BookingID not found
            INSERT
              VALUE #( travelid    = <fs_booking_read>-TravelID
                       bookingid   = <fs_booking_read>-BookingID
                       %fail-cause = if_abap_behv=>cause-not_found )
              INTO TABLE failed-booking.
          ENDIF.
        ENDLOOP.
      ELSE.
        "TravelID not found or other fail cause
        LOOP AT GROUP <fs_travel_read> ASSIGNING <fs_booking_read>.
          failed-booking = VALUE #(  BASE failed-booking
                                     FOR msg IN lt_message ( %key-TravelID    = <fs_booking_read>-TravelID
                                                             %key-BookingID   = <fs_booking_read>-BookingID
                                                             %fail-cause      = COND #( WHEN msg-msgty = 'E' AND msg-msgno = '016'
                                                                                        THEN if_abap_behv=>cause-not_found
                                                                                        ELSE if_abap_behv=>cause-unspecific ) ) ).
        ENDLOOP.

      ENDIF.

    ENDLOOP.

  ENDMETHOD.

***********************************************************************
*
* Create associated booking supplements
*
***********************************************************************
  METHOD cba_supplement.

    DATA lt_messages               TYPE /dmo/if_flight_legacy=>tt_message.
    DATA lt_booksupplement_old     TYPE /dmo/if_flight_legacy=>tt_booking_supplement.
    DATA ls_booksupplement         TYPE /dmo/book_suppl.
    DATA lv_last_booksupplement_id TYPE /dmo/booking_supplement_id.

    " Loop at parent - booking
    LOOP AT it_supplement_create_ba ASSIGNING FIELD-SYMBOL(<fs_supplement_create_ba>).
      DATA(ls_parent_key) = <fs_supplement_create_ba>-%key.

      " Retrieve booking supplements related to the imported travel ID
      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id          = ls_parent_key-travelid
        IMPORTING
          et_booking_supplement = lt_booksupplement_old
          et_messages           = lt_messages.

      IF lt_messages IS INITIAL.

        " Look up for maximum booking supplement ID for a given travel/booking
        " lt_booksupplement_old provides sorted values, therefore the last value is maximum value
        lv_last_booksupplement_id = REDUCE #( INIT res = 0
                                              FOR old IN lt_booksupplement_old
                                                USING KEY primary_key
                                                WHERE ( travel_id  = ls_parent_key-travelid
                                                    AND booking_id = ls_parent_key-bookingid )
                                              NEXT res = old-booking_supplement_id ).

        LOOP AT <fs_supplement_create_ba>-%target INTO DATA(ls_supplement_create).

          " Increase value of booking supplement ID with 1
          lv_last_booksupplement_id += 1.

          " Do mapping between the element names of the CDS view and the original table fields
          ls_booksupplement = CORRESPONDING #( ls_supplement_create MAPPING FROM ENTITY USING CONTROL ).

          " Assign the key elements
          ls_booksupplement-travel_id = ls_parent_key-TravelID.
          ls_booksupplement-booking_id = ls_parent_key-BookingID.
          ls_booksupplement-booking_supplement_id = lv_last_booksupplement_id.

          " Create a new booking supplement and update a booking instance
          CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
            EXPORTING
              is_travel              = VALUE /dmo/if_flight_legacy=>ts_travel_in( travel_id = ls_parent_key-travelid )
              is_travelx             = VALUE /dmo/if_flight_legacy=>ts_travel_inx( travel_id = ls_parent_key-travelid )
              it_booking_supplement  = VALUE /dmo/if_flight_legacy=>tt_booking_supplement_in( ( CORRESPONDING #( ls_booksupplement ) ) )
              it_booking_supplementx = VALUE /dmo/if_flight_legacy=>tt_booking_supplement_inx( ( VALUE #(
                                                                                                             booking_id = ls_booksupplement-booking_id
                                                                                                             booking_supplement_id = ls_booksupplement-booking_supplement_id
                                                                                                             action_code = /dmo/if_flight_legacy=>action_code-create )
                                                                                                 ) )
            IMPORTING
              et_messages            = lt_messages.

          IF lt_messages IS INITIAL.
            INSERT VALUE #( %cid      = ls_supplement_create-%cid
                            travelid  = ls_parent_key-travelid
                            bookingid = ls_parent_key-bookingid
                            bookingsupplementid = ls_booksupplement-booking_supplement_id )
                   INTO TABLE mapped-bookingsupplement.
          ELSE.

            " Issue a message in case of error ('E') or abort ('A')
            LOOP AT lt_messages INTO DATA(ls_message) WHERE msgty = 'E' OR msgty = 'A'.
              INSERT VALUE #( %cid      = ls_supplement_create-%cid
                              travelid  = ls_supplement_create-TravelID
                              bookingid = ls_supplement_create-BookingID )
                     INTO TABLE failed-bookingsupplement.

              INSERT VALUE #( %cid      = ls_supplement_create-%cid
                              TravelID  = ls_supplement_create-TravelID
                              BookingID = ls_supplement_create-BookingID
                              %msg      = new_message(
                                            id       = ls_message-msgid
                                            number   = ls_message-msgno
                                            severity = if_abap_behv_message=>severity-error
                                            v1       = ls_message-msgv1
                                            v2       = ls_message-msgv2
                                            v3       = ls_message-msgv3
                                            v4       = ls_message-msgv4 ) )
                    INTO TABLE reported-bookingsupplement.


            ENDLOOP.

          ENDIF.

        ENDLOOP.

      ELSE.
        /dmo/cl_travel_auxiliary=>handle_booking_messages(
           EXPORTING
             iv_cid       = <fs_supplement_create_ba>-%cid_ref
             iv_travel_id = ls_parent_key-travelid
             it_messages  = lt_messages
           CHANGING
             failed   = failed-booking
             reported = reported-booking ).

      ENDIF.

    ENDLOOP.

  ENDMETHOD.


**********************************************************************
*
* Read booking supplement by association
*
**********************************************************************
  METHOD read_supplement_ba.
    DATA: ls_travel_out    TYPE /dmo/travel,
          lt_booksuppl_out TYPE /dmo/if_flight_legacy=>tt_booking_supplement,
          lt_message       TYPE /dmo/if_flight_legacy=>tt_message.

    "Only one function call for each travelid
    LOOP AT it_booking ASSIGNING FIELD-SYMBOL(<fs_travel>)
                       GROUP BY <fs_travel>-TravelID.

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id          = <fs_travel>-travelid
        IMPORTING
          es_travel             = ls_travel_out
          et_booking_supplement = lt_booksuppl_out
          et_messages           = lt_message.

      IF lt_message IS INITIAL.
        "Get BookingSupplements for each travel
        LOOP AT GROUP <fs_travel> ASSIGNING FIELD-SYMBOL(<fs_booking>).
          LOOP AT lt_booksuppl_out ASSIGNING FIELD-SYMBOL(<fs_booksuppl>) WHERE  travel_id  = <fs_booking>-%key-TravelID
                                                                          AND    booking_id = <fs_booking>-%key-BookingID .
            "Fill link table with key fields
            INSERT
                 VALUE #( source-%key = VALUE #( TravelID            = <fs_booking>-%key-TravelID
                                                 BookingID           = <fs_booking>-%key-BookingID )
                          target-%key = VALUE #( TravelID            = <fs_booksuppl>-travel_id
                                                 BookingID           = <fs_booksuppl>-booking_id
                                                 BookingSupplementID = <fs_booksuppl>-booking_supplement_id  ) )
                 INTO TABLE et_link_table.

            "fill result parameter with flagged fields
            IF ev_full_requested = abap_true.
              INSERT
                 VALUE #( travelid            = SWITCH #( <fs_booking>-%control-TravelID            WHEN cl_abap_behv=>flag_changed THEN <fs_booksuppl>-travel_id )
                          bookingid           = SWITCH #( <fs_booking>-%control-BookingID           WHEN cl_abap_behv=>flag_changed THEN <fs_booksuppl>-booking_id )
                          bookingsupplementid = SWITCH #( <fs_booking>-%control-BookingSupplementID WHEN cl_abap_behv=>flag_changed THEN <fs_booksuppl>-booking_supplement_id )
                          supplementid        = SWITCH #( <fs_booking>-%control-SupplementID        WHEN cl_abap_behv=>flag_changed THEN <fs_booksuppl>-supplement_id )
                          price               = SWITCH #( <fs_booking>-%control-Price               WHEN cl_abap_behv=>flag_changed THEN <fs_booksuppl>-price )
                          currencycode        = SWITCH #( <fs_booking>-%control-CurrencyCode        WHEN cl_abap_behv=>flag_changed THEN <fs_booksuppl>-currency_code )
*                          lastchangedat       = SWITCH #( <fs_booking>-%control-LastChangedAt       WHEN cl_abap_behv=>flag_changed THEN ls_travel_out-lastchangedat )
)
                 INTO TABLE et_booksuppl.
            ENDIF.
          ENDLOOP.
        ENDLOOP.

      ELSE.
        "Fill failed table in case of error
        LOOP AT GROUP <fs_travel> ASSIGNING <fs_booking>.
          failed-booking = VALUE #(  BASE failed-booking
                                FOR msg IN lt_message ( %key-TravelID    = <fs_booking>-%key-TravelID
                                                        %key-BookingID   = <fs_booking>-%key-BookingID
                                                        %fail-cause      = COND #( WHEN msg-msgty = 'E' AND msg-msgno = '016'
                                                                                   THEN if_abap_behv=>cause-not_found
                                                                                   ELSE if_abap_behv=>cause-unspecific ) ) ).

        ENDLOOP.

      ENDIF.

    ENDLOOP.

  ENDMETHOD.


ENDCLASS.

```
33.	Save and activate your work.
<br>![](/exercises/ex3/images/03_02_0330.png)

34.	Use what you have learned and create another ABAP Class for a another Behavior Pool and call it Z_BP_I_BOOKINGSUPPLEMENT_U_XXX.  Copy the following code to the “Local Types“ tab.  As you can see, this Behavior Pool contains implementations for the READ, UPDATE and DELETE operations for **BookingSupplement**.  
```abap
**********************************************************************
*
* Handler class implements UPDATE and DELETE for booking supplements
*
**********************************************************************
CLASS lhc_supplement DEFINITION INHERITING FROM cl_abap_behavior_handler.

  PRIVATE SECTION.

    TYPES tt_bookingsupplement_failed   TYPE TABLE FOR FAILED   /dmo/i_bookingsupplement_u.
    TYPES tt_bookingsupplement_reported TYPE TABLE FOR REPORTED /dmo/i_bookingsupplement_u.

    TYPES:
        tt_booking_update           TYPE TABLE FOR UPDATE    /dmo/i_booking_u,
        tt_bookingsupplement_update TYPE TABLE FOR UPDATE    /dmo/i_bookingsupplement_u.


    METHODS:
        update_bookingsupplement FOR MODIFY
                                    IMPORTING it_bookingsupplement_update FOR UPDATE bookingsupplement,
        delete_bookingsupplement FOR MODIFY
                                    IMPORTING it_bookingsupplement_delete FOR DELETE bookingsupplement,
      read_bookingsupplement FOR READ
        IMPORTING it_bookingsupplement_read FOR READ bookingsupplement
        RESULT    et_bookingsupplement.

ENDCLASS.


CLASS lhc_supplement IMPLEMENTATION.

**********************************************************************
*
* Implements the UPDATE operation for a set of booking supplements
*
**********************************************************************
  METHOD update_bookingsupplement.

    DATA lt_messages TYPE /dmo/if_flight_legacy=>tt_message.
    DATA lt_book_supplement TYPE /dmo/book_suppl.
    data ls_bookingsupplement_inx type /dmo/if_flight_legacy=>ts_booking_supplement_inx.

    LOOP AT it_bookingsupplement_update ASSIGNING FIELD-SYMBOL(<fs_bookingsupplement_update>).
    lt_book_supplement = CORRESPONDING  #( <fs_bookingsupplement_update> MAPPING FROM ENTITY ).

    CLEAR ls_bookingsupplement_inx.
    ls_bookingsupplement_inx-booking_supplement_id = <fs_bookingsupplement_update>-bookingsupplementid.
    ls_bookingsupplement_inx-action_code           = /dmo/if_flight_legacy=>action_code-update.
    ls_bookingsupplement_inx-booking_id            = <fs_bookingsupplement_update>-bookingid.
    ls_bookingsupplement_inx-supplement_id         = xsdbool( <fs_bookingsupplement_update>-%control-supplementid = if_abap_behv=>mk-on ).
    ls_bookingsupplement_inx-price                 = xsdbool( <fs_bookingsupplement_update>-%control-price        = if_abap_behv=>mk-on ).
    ls_bookingsupplement_inx-currency_code         = xsdbool( <fs_bookingsupplement_update>-%control-currencycode = if_abap_behv=>mk-on ).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
        EXPORTING
          is_travel              = VALUE /dmo/if_flight_legacy=>ts_travel_in( travel_id = <fs_bookingsupplement_update>-travelid )
          is_travelx             = VALUE /dmo/if_flight_legacy=>ts_travel_inx( travel_id = <fs_bookingsupplement_update>-travelid )
          it_booking_supplement  = VALUE /dmo/if_flight_legacy=>tt_booking_supplement_in( ( CORRESPONDING #( lt_book_supplement ) ) )
          it_booking_supplementx = VALUE /dmo/if_flight_legacy=>tt_booking_supplement_inx( ( ls_bookingsupplement_inx ) )
        IMPORTING
          et_messages = lt_messages.


      /dmo/cl_travel_auxiliary=>handle_booksupplement_messages(
        EXPORTING
          iv_cid                  = <fs_bookingsupplement_update>-%cid_ref
          iv_travel_id            = <fs_bookingsupplement_update>-travelid
          iv_booking_id           = <fs_bookingsupplement_update>-bookingid
          iv_bookingsupplement_id = <fs_bookingsupplement_update>-bookingsupplementid
          it_messages             = lt_messages
        CHANGING
            failed   = failed-bookingsupplement
            reported = reported-bookingsupplement ).

    ENDLOOP.

  ENDMETHOD.

**********************************************************************
*
* Implements the DELETE operation for a set of booking supplements
*
**********************************************************************
  METHOD delete_bookingsupplement.

    DATA lt_messages TYPE /dmo/if_flight_legacy=>tt_message.

    LOOP AT it_bookingsupplement_delete INTO DATA(ls_bookingsupplement_delete).

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_UPDATE'
        EXPORTING
          is_travel              = VALUE /dmo/if_flight_legacy=>ts_travel_in( travel_id = ls_bookingsupplement_delete-travelid )
          is_travelx             = VALUE /dmo/if_flight_legacy=>ts_travel_inx( travel_id = ls_bookingsupplement_delete-travelid )
          it_booking             = VALUE /dmo/if_flight_legacy=>tt_booking_in( ( booking_id = ls_bookingsupplement_delete-bookingid ) )
          it_bookingx            = VALUE /dmo/if_flight_legacy=>tt_booking_inx( ( booking_id  = ls_bookingsupplement_delete-bookingid ) )
          it_booking_supplement  = VALUE /dmo/if_flight_legacy=>tt_booking_supplement_in( (  booking_supplement_id = ls_bookingsupplement_delete-bookingSupplementid
                                                                                             booking_id            = ls_bookingsupplement_delete-BookingID ) )
          it_booking_supplementx = VALUE /dmo/if_flight_legacy=>tt_booking_supplement_inx( ( booking_supplement_id = ls_bookingsupplement_delete-bookingsupplementid
                                                                                             booking_id            = ls_bookingsupplement_delete-bookingid
                                                                                             action_code           = /dmo/if_flight_legacy=>action_code-delete ) )
        IMPORTING
          et_messages = lt_messages.

      IF lt_messages IS NOT INITIAL.

        /dmo/cl_travel_auxiliary=>handle_booksupplement_messages(
         EXPORTING
           iv_cid                  = ls_bookingsupplement_delete-%cid_ref
           iv_travel_id            = ls_bookingsupplement_delete-travelid
           iv_booking_id           = ls_bookingsupplement_delete-bookingid
           iv_bookingsupplement_id = ls_bookingsupplement_delete-bookingsupplementid
           it_messages = lt_messages
         CHANGING
           failed   = failed-bookingsupplement
           reported = reported-bookingsupplement ).

      ENDIF.

    ENDLOOP.

  ENDMETHOD.

**********************************************************************
*
* Implements the READ operation for a set of booking supplements
*
**********************************************************************
  METHOD read_bookingsupplement.

    DATA: ls_travel_out    TYPE /dmo/travel,
          lt_booksuppl_out TYPE /dmo/if_flight_legacy=>tt_booking_supplement,
          lt_message       TYPE /dmo/if_flight_legacy=>tt_message.

    LOOP AT it_bookingsupplement_read ASSIGNING FIELD-SYMBOL(<fs_travel_read>)
                                      GROUP BY <fs_travel_read>-TravelID.

      CALL FUNCTION '/DMO/FLIGHT_TRAVEL_READ'
        EXPORTING
          iv_travel_id          = <fs_travel_read>-travelid
        IMPORTING
          es_travel             = ls_travel_out
          et_booking_supplement = lt_booksuppl_out
          et_messages           = lt_message.

      IF lt_message IS INITIAL.
        LOOP AT GROUP <fs_travel_read> ASSIGNING FIELD-SYMBOL(<fs_bookingsuppl_read>).

          READ TABLE lt_booksuppl_out INTO DATA(ls_bookingsuppl) WITH KEY  travel_id             = <fs_bookingsuppl_read>-%key-TravelID
                                                                           booking_id            = <fs_bookingsuppl_read>-%key-BookingID
                                                                           booking_supplement_id = <fs_bookingsuppl_read>-%key-BookingSupplementID.
          IF sy-subrc = 0 .
            "fill result parameter with flagged fields
            INSERT
               VALUE #( travelid            = ls_bookingsuppl-travel_id
                        bookingid           = ls_bookingsuppl-booking_id
                        bookingsupplementid = ls_bookingsuppl-booking_supplement_id
                        supplementid        = COND #( WHEN <fs_bookingsuppl_read>-%control-SupplementID        = cl_abap_behv=>flag_changed THEN ls_bookingsuppl-supplement_id  )
                        price               = COND #( WHEN <fs_bookingsuppl_read>-%control-Price               = cl_abap_behv=>flag_changed THEN ls_bookingsuppl-price  )
                        currencycode        = COND #( WHEN <fs_bookingsuppl_read>-%control-CurrencyCode        = cl_abap_behv=>flag_changed THEN ls_bookingsuppl-currency_code  )
*                       lastchangedat       = COND #( WHEN <fs_bookingsuppl_read>-%control-LastChangedAt       = cl_abap_behv=>flag_changed THEN ls_travel_out-lastchangedat   )
                      ) INTO TABLE et_bookingsupplement.



          ELSE.
            "BookingSupplementID not found
            INSERT
              VALUE #( travelid           = <fs_bookingsuppl_read>-TravelID
                       bookingid          = <fs_bookingsuppl_read>-BookingID
                       bookingsupplementid = <fs_bookingsuppl_read>-BookingSupplementID
                       %fail-cause = if_abap_behv=>cause-not_found )
              INTO TABLE failed-bookingsupplement.
          ENDIF.
        ENDLOOP.
      ELSE.

        "TravelID not found or other fail cause
        LOOP AT GROUP <fs_travel_read> ASSIGNING <fs_bookingsuppl_read>.
          failed-bookingsupplement = VALUE #(  BASE failed-bookingsupplement
                                     FOR msg IN lt_message ( %key-TravelID            = <fs_bookingsuppl_read>-TravelID
                                                             %key-BookingID           = <fs_bookingsuppl_read>-BookingID
                                                             %key-bookingsupplementid = <fs_bookingsuppl_read>-BookingSupplementID
                                                             %fail-cause              = COND #( WHEN msg-msgty = 'E' AND msg-msgno = '016'
                                                                                                THEN if_abap_behv=>cause-not_found
                                                                                                ELSE if_abap_behv=>cause-unspecific ) ) ).
        ENDLOOP.

      ENDIF.

    ENDLOOP.


  ENDMETHOD.

ENDCLASS.

```
35.	Save and activate your work.
<br>![](/exercises/ex3/images/03_02_0350.png)

## Exercise 3.3 Create the Projection Views

After completing these steps you will have created Projection Views over your Business Object Views. The Projection Views are meant to further filter the data which is exposed. 

1.	Right-click  on the “Data Definitions“ folder and choose “New Data Definition“.
<br>![](/exercises/ex3/images/03_03_0010.png)

2.	Give the name as Z_C_TRAVEL_U_XXX where XXX is your group number.  Give a meaningful description and click “Next”.
<br>![](/exercises/ex3/images/03_03_0020.png)

3. Click “Next“.
<br>![](/exercises/ex3/images/03_03_0030.png)

4. Choose Projection View from the selection box and click “Finish“.
<br>![](/exercises/ex3/images/03_03_0040.png)

5.	Enter the code as shown here, and replace XXX with your group number. Note, that in the DEFINE ROOT VIEW statement, there is the extension  AS PROJECTION ON pointing to your Business Object view Z_I_TRAVEL_U_XXX.  In this projection view, not only are we filtering columns, but we are adding value help declarations as well using annotations.  
```abap
@EndUserText.label: 'Travel Projection View'
@AccessControl.authorizationCheck: #NOT_REQUIRED

@Metadata.allowExtensions: true

@Search.searchable: true
define root view entity Z_C_Travel_U_XXX
  as projection on Z_I_TRAVEL_U_XXX
{   
  key TravelID,

      @Consumption.valueHelpDefinition: [{ entity: { name:    '/DMO/I_Agency',
                                                     element: 'AgencyID' } }]
      @ObjectModel.text.element: ['AgencyName']
      @Search.defaultSearchElement: true
      AgencyID,
      _Agency.Name       as AgencyName,

      @Consumption.valueHelpDefinition: [{ entity: { name:    '/DMO/I_Customer',
                                                     element: 'CustomerID'  } }]
      @ObjectModel.text.element: ['CustomerName']
      @Search.defaultSearchElement: true
      CustomerID,
      _Customer.LastName as CustomerName,

      BeginDate,
      EndDate,
      BookingFee,
      TotalPrice,

      @Consumption.valueHelpDefinition: [{entity: { name:    'I_Currency',
                                                    element: 'Currency' } }]
      CurrencyCode,

      Memo,
      Status,
      LastChangedAt,
      
      /* public associations */
      _Booking : redirected to composition child Z_C_Booking_U_XXX,
      _Agency,
      _Currency,
      _Customer
}

```
6.	Save your work, but don’t try to activate it yet because we haven’t created the projection view for **Booking** or **BookingSupplement** yet.
<br>![](/exercises/ex3/images/03_03_0060.png)

7.	Use what you have learned and create another projection view called Z_C_BOOKING_U_XXX. Enter the code as shown here. Make sure to replace XXX with your group number.  Once again, we are filtering the columns and adding value helps via annotations.
```abap
@EndUserText.label: 'Booking Projection View'
@AccessControl.authorizationCheck: #NOT_REQUIRED

@Metadata.allowExtensions: true

@Search.searchable: true
define view entity Z_C_Booking_U_XXX
  as projection on Z_I_BOOKING_U_XXX

{     
      @Search.defaultSearchElement: true
  key TravelID,
 
      @Search.defaultSearchElement: true
  key BookingID,

      BookingDate,

      @Consumption.valueHelpDefinition: [ { entity: { name:    '/DMO/I_Customer', 
                                                     element: 'CustomerID' } } ]
      @Search.defaultSearchElement: true
      @ObjectModel.text.element: ['CustomerName']
      CustomerID,
      _Customer.LastName    as CustomerName,

      @Consumption.valueHelpDefinition: [ { entity: { name:    '/DMO/I_Carrier', 
                                                      element: 'AirlineID' } } ]
      @ObjectModel.text.element: ['AirlineName']
      AirlineID,
      _Carrier.Name     as AirlineName,

      @Consumption.valueHelpDefinition: [ { entity: { name:    '/DMO/I_Flight', 
                                                      element: 'ConnectionID' },
                                            additionalBinding: [ { localElement: 'FlightDate',   element: 'FlightDate' },
                                                                 { localElement: 'AirlineID',    element: 'AirlineID' },
                                                                 { localElement: 'FlightPrice',  element: 'Price'  },
                                                                 { localElement: 'CurrencyCode', element: 'CurrencyCode' } ] } ]
      ConnectionID,
      
      @Consumption.valueHelpDefinition: [ { entity: { name:    '/DMO/I_Flight', 
                                                      element: 'FlightDate' },
                                            additionalBinding: [ { localElement: 'ConnectionID', element: 'ConnectionID' },
                                                                 { localElement: 'AirlineID',    element: 'AirlineID' },
                                                                 { localElement: 'FlightPrice',  element: 'Price' },
                                                                 { localElement: 'CurrencyCode', element: 'CurrencyCode' } ] } ]
      FlightDate,
      
      FlightPrice,
      
      @Consumption.valueHelpDefinition: [ {entity: { name:    'I_Currency', 
                                                     element: 'Currency' } } ]
      CurrencyCode,
      
      LastChangedAt,
      
      /* public associations */
      _Travel: redirected to parent Z_C_Travel_U_XXX,
_BookSupplement: redirected to composition child Z_C_BookingSupplement_U_XXX,
      _Carrier,
      _Connection,
      _Customer
}

```
8.	Save your work.
<br>![](/exercises/ex3/images/03_03_0080.png)

9.	Use what you have learned and create another projection view called Z_C_BOOKINGSUPPLEMENT_U_XXX. Enter the code as shown here. Make sure to replace XXX with your group number.  Once again, we are filtering the columns and adding value helps via annotations.
```abap
@EndUserText.label: 'Booking Supplement Projection View'
@AccessControl.authorizationCheck: #NOT_REQUIRED

@Metadata.allowExtensions: true

@Search.searchable: true
define view entity Z_C_BookingSupplement_U_XXX
  as projection on Z_I_BookingSupplement_U_XXX

{     ///DMO/I_BookingSupplement_U
      @Search.defaultSearchElement: true
  key TravelID,

      @Search.defaultSearchElement: true
  key BookingID,

  key BookingSupplementID,

      @Consumption.valueHelpDefinition: [ {entity: { name:    '/DMO/I_SUPPLEMENT',
                                                     element: 'SupplementID' },
                                           additionalBinding: [ { localElement: 'Price',        element: 'Price'},
                                                                { localElement: 'CurrencyCode', element: 'CurrencyCode' } ] } ]
      @ObjectModel.text.element: ['SupplementText']
      SupplementID,
      _SupplementText.Description as SupplementText : localized,

      Price,

      @Consumption.valueHelpDefinition: [ { entity: { name:    'I_Currency', 
                                                      element: 'Currency' } } ]
      CurrencyCode,

      LastChangedAt,

      /* Associations */
      ///DMO/I_BookingSupplement_U
      _Booking : redirected to parent Z_C_Booking_U_XXX,
      _Travel  : redirected to Z_C_Travel_U_XXX , 
      _Product,
      _SupplementText

}

```

10. Save your work.
<br>![](/exercises/ex3/images/03_03_0100.png)

11.	Select all projection views, and right-click and choose “Activate“.
<br>![](/exercises/ex3/images/03_03_0110.png)

## Exercise 3.4 Create the Metadata Extensions

After completing these steps you will have created the Metadata Extensions. The Metadata Extensions are meant to separate the UI specific annotations from the rest of the data model.  This makes for much cleaner code.

1.	Right-click on the Z_C_TRAVEL_U_XXX CDS artifact and choose “New Metadata Extension“.
<br>![](/exercises/ex3/images/03_04_0010.png)

2.	Give the name as Z_C_TRAVEL_U_XXX where XXX is your group number.  Give a meaningful description and click “Next”, then “Finish”.
<br>![](/exercises/ex3/images/03_04_0020.png)

3.	Enter the code as shown here, and replace XXX with your group number.  Here we are using the @UI annotations to define what the user interface will look like.    
```abap
@Metadata.layer: #CORE

@UI: { headerInfo: { typeName: 'Travel', 
                     typeNamePlural: 'Travels', 
                     title: { type: #STANDARD, 
                              value: 'TravelID' } } }
                              
annotate view Z_C_Travel_U_XXX with

{
  @UI.facet: [ { id:            'Travel',
                 purpose:       #STANDARD,
                 type:          #IDENTIFICATION_REFERENCE,
                 label:         'Travel',
                 position:      10 },
               { id:            'Booking',
                 purpose:       #STANDARD,
                 type:          #LINEITEM_REFERENCE,
                 label:         'Booking',
                 position:      20,
                 targetElement: '_Booking'}]
                   
  @UI: { lineItem:       [ { position: 10, 
                             importance: #HIGH } ], 
         identification: [ { position: 10 } ], 
         selectionField: [ { position: 10 } ] }
  TravelID;

  @UI: { lineItem:       [ { position: 20, 
                             importance: #HIGH } ], 
         identification: [ { position: 20 } ], 
         selectionField: [ { position: 20 } ] }
  AgencyID;
 
  @UI: { lineItem:       [ { position: 30, 
                             importance: #HIGH } ], 
         identification: [ { position: 30 } ], 
         selectionField: [ { position: 30 } ] }
  CustomerID; 
  
  @UI: { lineItem:       [ { position: 40, 
                              importance: #MEDIUM } ], 
         identification: [ { position: 40 } ] }
  BeginDate;
 
  @UI: { lineItem:       [ { position: 41, 
                             importance: #MEDIUM } ], 
         identification: [ { position: 41 } ] }
  EndDate;
  
  @UI: { identification: [ { position: 42 } ] }
  BookingFee;
  
  @UI: { identification: [ { position: 43 } ] }
  TotalPrice;
 
  @UI: { identification:[ { position: 45, 
                            label: 'Comment' } ] }
  Memo;
  
  @UI: { lineItem: [ { position: 50, 
                       importance: #HIGH },
                     { type: #FOR_ACTION, 
                       dataAction: 'set_status_booked', 
                       label: 'Set to Booked' } ] }
  Status;
}

```

4.	Save and activate.
<br>![](/exercises/ex3/images/03_04_0040.png)

5.	Use what you have learned and create another metadata extension called Z_C_BOOKING_U_XXX. Enter the code as shown here. Make sure to replace XXX with your group number.  Once again, we are defining the user interface with @UI annotations.
```abap
@Metadata.layer: #CORE

@UI: {
  headerInfo: { typeName: 'Booking',
                typeNamePlural: 'Bookings',
                title: { type: #STANDARD,
                         value: 'BookingID' } } }


annotate view Z_C_Booking_U_XXX with
{
  @UI.facet: [ { id:            'Booking',
                 purpose:       #STANDARD,
                 type:          #IDENTIFICATION_REFERENCE,
                 label:         'Booking',
                 position:      10 },
               { id:            'BookingSupplement',
                 purpose:       #STANDARD,
                 type:          #LINEITEM_REFERENCE,
                 label:         'Booking Supplement',
                 position:      20,
                 targetElement: '_BookSupplement'} ]


  @UI: { lineItem:       [ { position: 20,
                             importance: #HIGH } ],
         identification: [ { position: 20 } ] }
  BookingID;

  @UI: { lineItem:       [ { position: 30,
                             importance: #HIGH } ],
         identification: [ { position: 30 } ] }
  BookingDate;

  @UI: { lineItem:       [ { position: 40,
                             importance: #HIGH } ],
         identification: [ { position: 40 } ] }
  CustomerID;

  @UI: { lineItem:       [ { position: 50,
                             importance: #HIGH } ],
         identification: [ { position: 50 } ] }

  AirlineID;

  @UI: { lineItem:       [ { position: 60,
                             importance: #HIGH } ],
         identification: [ { position: 60 } ] }
  ConnectionID;

  @UI: { lineItem:       [ { position: 70,
                             importance: #HIGH } ],
         identification: [ { position: 70 } ] }

  FlightDate;
  @UI: { lineItem:       [ { position: 80,
                             importance: #HIGH } ],
                             identification: [ { position: 80 } ] }
  FlightPrice;

}

```

6.	Save and activate.
<br>![](/exercises/ex3/images/03_04_0060.png)

7.	Use what you have learned and create another metadata extension called Z_C_BOOKINGSUPPLEMENT_U_XXX. Enter the code as shown here. Make sure to replace XXX with your group number.  Once again, we are defining the user interface with @UI annotations.
```abap
@Metadata.layer: #CORE

@UI: { headerInfo: { typeName: 'Booking Supplement',
                     typeNamePlural: 'Booking Supplements',
                     title: { type: #STANDARD, label: 'Booking Supplement', value: 'BookingSupplementID' } } }

annotate view Z_C_BookingSupplement_U_XXX with
{
  @UI.facet: [ { id:            'BookingSupplement',
                 purpose:       #STANDARD,
                 type:          #IDENTIFICATION_REFERENCE,
                 label:         'Booking Supplement',
                 position:      10 } ,
                { id:              'PriceHeader',
                  type:            #DATAPOINT_REFERENCE,
                  purpose:         #HEADER,
                  targetQualifier: 'Price',
                  label:           'Price',
                  position:        20 } ]
                  
 
  @UI: { lineItem:       [ { position: 10, importance: #HIGH } ],
         identification: [ { position: 10 } ] }
  BookingSupplementID;
 
  @UI: { lineItem:       [ { position: 20, importance: #HIGH } ],
         identification: [ { position: 20 } ] }
  SupplementID;

  @UI: { lineItem:       [ { position: 30, importance: #HIGH } ],
         identification: [ { position: 30 } ],
         dataPoint:      { title: 'Price' }
         }
  Price;
}

```

8.	Save and activate.
<br>![](/exercises/ex3/images/03_04_0080.png)

## Exercise 3.5 Create the Behavior Definition Projection

After completing these steps you will have created the Behavior Definition Projection.  This is meant to further restrict the behavior operations. The idea is that you will have one monolithic Business Object data model, while having several Projection Views and Behavior Projections on top. For example, if there were different operations that would be allowed for a "processor" and a "manager", these would be two different Projection Views and Behavior Projections, each exposing differnt data and/or allowing different operations.

1.	Select the Z_C_TRAVEL_U_XXX CDS artifact and right-click it and choose “New Behavior Definition“.
<br>![](/exercises/ex3/images/03_05_0010.png)

2.	Notice the implementation type is Projection. Click “Next”, then “Finish”.
<br>![](/exercises/ex3/images/03_05_0020.png) 

3.	The behavior definition is prefilled with all operations and actions of the underlying Business Object.  Define aliases for the behaviors as shown. For now, we will allow all operations.  
<br>![](/exercises/ex3/images/03_05_0030.png) 

4.	Save and activate your work.
<br>![](/exercises/ex3/images/03_05_0040.png) 

## Exercise 3.6 Create the Service Definition and Service Binding

After completing these steps you will have created the Service Definition which exposes the Projection Views, and the Service Binding which binds the Service Definition to a specific protocol and usage.

1.	Select the Z_C_TRAVEL_U_XXX CDS artifact and right-click and choose “New Service Definition“.
<br>![](/exercises/ex3/images/03_06_0010.png)

2.	Give the name of the service definition as Z_SD_C_TRAVEL_U_XXX where XXX is your group number.  Give a meaningful description.  Click “Next“, then “Finish“.
<br>![](/exercises/ex3/images/03_06_0020.png)

3.	The service definition is prefilled for you.  Here we are exposing only the Z_C_TRAVEL_U_XXX entity.
<br>![](/exercises/ex3/images/03_06_0030.png)

4.	Update the definition to expose the Z_C_BOOKING_U_XXX entity and Z_C_BOOKINGSUPPLEMENT_U_XXX as well.  Also add aliases to all as shown here. 
```abap
@EndUserText.label: 'Service Definition for Travel'
define service Z_SD_C_TRAVEL_U_XXX {
  expose Z_C_Travel_U_XXX as Travel;
  expose Z_C_Booking_U_XXX as Booking;
  expose Z_C_BookingSupplement_U_XXX as BookingSupplement;
}

```
5.	Save and activate your work.
<br>![](/exercises/ex3/images/03_06_0050.png)

6.	Go to the sevice definition folder and right-click on your service definnition and choose “New Service Binding“.
<br>![](/exercises/ex3/images/03_06_0060.png)

7.	Give the name as Z_UI_C_TRAVEL_U_XXX where XXX is your group number.  Give a meaningful description.  Make sure the binding type is OData V2 -UI.  Click “Next“, then “Finish“.
<br>![](/exercises/ex3/images/03_06_0070.png)

8.	In the service binding, click the “Activate“ button.
<br>![](/exercises/ex3/images/03_06_0080.png)

9.	Select the Travel entity set, and click “Preview“.
<br>![](/exercises/ex3/images/03_06_0090.png)

10.	The browser will be launched where it may ask for you to log in.  Here you can preview the data and test the create, update, delete behaviors.  Play around with this preview UI and test your CRUD operations. Set some breakpoints in your code and see what methods are triggered in what scenarios.
<br>![](/exercises/ex3/images/03_06_0100.png)

## Summary

You've now created your Business Object data model, defined and implemented your Behaviors, created Projections over both, and exposed your data model as an OData service using the Unmanaged Scenario of the ABAP RESTful Application Programming Model

Continue to - [Exercise 4 - ABAP RESTful Application Programming Model - Managed ](../ex4/README.md)
