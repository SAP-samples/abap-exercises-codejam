# Exercise 4 - ABAP RESTful Programming Model - Managed

In this exercise we will explore the new ABAP RESTful Programming Model, specifically the Managed Scenario.  The Managed scenario provides a framework for the developer to created applications quickly and easily while focusing mainly on the business logic as opposed to the nuts and bolts of the transaction. The framework is is designed to handle the CRUD operations and save mechanism automatically without the developer having to code for it.  The developer can focus more on the business logic of the application and add in additional validations. We will create views over our data model, define behaviors, and service enable the data model as well.   

## Exercise 4.1 Create the Business Object Views

After completing these steps you will have created the Business Object Views over the base database tables.

1.	From your group package, right-click and choose “New“, then “Other ABAP Repository Object“.
<br>![](/exercises/ex4/images/04_01_0010.png)

2.	Expand Core Data Services and choose “Data Definition”.  Click “Next”.
<br>![](/exercises/ex4/images/04_01_0020.png)

3.	Enter the name of the CDS view as Z_I_TRAVEL_M_XXX where XXX is your group number.  Give a meaningful description.  Click “Next”.
<br>![](/exercises/ex4/images/04_01_0030.png)

4.	Click “Next”.
<br>![](/exercises/ex4/images/04_01_0040.png)

5.	Choose Define View from the selection box and click “Finish“.
<br>![](/exercises/ex4/images/04_01_0050.png)

6.	The CDS definition editor will open with the following code.
<br>![](/exercises/ex4/images/04_01_0060.png)

7.	First, change the name of the sqlViewName annotation as shown here where XXX is your group number.  If you DO NOT want to type all of this code in the following steps, you may copy it from the solution.
<br>![](/exercises/ex4/images/04_01_0070.png)

8.	There is a lot of code to add here. We’ll try to take it by sections.  This code is not only defining the associations to other tables/views as well as the columns that we want in our view, but also how these columns are presented in the user interface later on. The annotations are what drive this funtionality.  In this first section at the top, make sure that you add these annotations if they are not already there. 
<br>![](/exercises/ex4/images/04_01_0080.png)

9.	After the annotations at the top, adjust the define view statement as shown here. Add the word “root“ and change the select from to “/dmo/travel as Travel“.  This is an existing table already in the system.
```abap
define root view Z_I_TRAVEL_M_XXX
  as select from /dmo/travel_m as Travel

```
10.	Next, add the composition and associations to other tables/views.  In this case we add a composition to the Booking view which we have not created yet. Make sure to replace XXX with your group number.
```abap
  composition [0..*] of Z_I_Booking_M_XXX as _Booking

  association [0..1] to /DMO/I_Agency    as _Agency   on $projection.agency_id     = _Agency.AgencyID
  association [0..1] to /DMO/I_Customer  as _Customer on $projection.customer_id   = _Customer.CustomerID
  association [0..1] to I_Currency       as _Currency on $projection.currency_code = _Currency.Currency  


```
11.	Finally add the columns and association references.
```abap
{   
  key travel_id,      
    agency_id,         
    customer_id,  
    begin_date,  
    end_date,   
    @Semantics.amount.currencyCode: 'currency_code'
    booking_fee,
    @Semantics.amount.currencyCode: 'currency_code'
    total_price,  
    @Semantics.currencyCode: true
    currency_code,  
    overall_status,
    description,   
    @Semantics.user.createdBy: true 
    created_by,
    @Semantics.systemDateTime.createdAt: true       
    created_at,
    @Semantics.user.lastChangedBy: true    
    last_changed_by,
    @Semantics.systemDateTime.lastChangedAt: true
    last_changed_at,                -- used as etag field
    
    /* Associations */
    _Booking,
    _Agency,
    _Customer, 
    _Currency
}


```

12.	Save your work, but do not try to activate at this point as we have not created the **Booking** view yet.
<br>![](/exercises/ex4/images/04_01_0120.png)

13.	Now, use what you have learned and create another view in the same way called Z_I_BOOKING_M_XXX.  As you did before, add the code as shown here. Make sure to use your group nubmer where there is XXX. 
```abap
@AbapCatalog.sqlViewName: 'ZIBOOKING_M_XXX'
@AbapCatalog.compiler.compareFilter: true
@AbapCatalog.preserveKey: true
@AccessControl.authorizationCheck: #NOT_REQUIRED

@Metadata.ignorePropagatedAnnotations:true

@EndUserText.label: 'Booking view'

define view Z_I_Booking_M_XXX
  as select from /dmo/booking_m as Booking
  association        to parent Z_I_Travel_M_XXX as _Travel     on  $projection.travel_id = _Travel.travel_id
  composition [0..*] of Z_I_BookSuppl_M_XXX     as _BookSupplement

  association [1..1] to /DMO/I_Customer        as _Customer   on  $projection.customer_id = _Customer.CustomerID
  association [1..1] to /DMO/I_Carrier         as _Carrier    on  $projection.carrier_id = _Carrier.AirlineID
  association [1..1] to /DMO/I_Connection      as _Connection on  $projection.carrier_id    = _Connection.AirlineID
                                                              and $projection.connection_id = _Connection.ConnectionID

{
  key travel_id,
  key booking_id,

      booking_date,
      customer_id,
      carrier_id,
      connection_id,
      flight_date,
      @Semantics.amount.currencyCode: 'currency_code'
      flight_price,
      @Semantics.currencyCode: true
      currency_code,
      booking_status,
      @UI.hidden: true
      _Travel.last_changed_at,

      /* Associations */
      _Travel,
      _BookSupplement,
      _Customer,
      _Carrier,
      _Connection

}


```
14.	Save your work.
<br>![](/exercises/ex4/images/04_01_0140.png)

15.	Use what you have learned and create another view in the same way called Z_I_BOOKINGSUPPLEMENT_M_XXX.  As you did before, add the code as shown here. Make sure to use your group nubmer where there is XXX.
```abap
@AbapCatalog.sqlViewName: 'ZIBOOKSUPP_M_XXX'
@AbapCatalog.compiler.compareFilter: true
@AbapCatalog.preserveKey: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Booking Supplement view - CDS data model'

define view Z_I_BookingSupplement_M_XXX
  as select from /DMO/book_suppl_m as BookingSupplement
  association        to parent Z_I_Booking_M_XXX  as _Booking     on  $projection.travel_id    = _Booking.travel_id
                                                                 and $projection.booking_id   = _Booking.booking_id

  association [1..1] to /DMO/I_Travel_M       as _Travel         on  $projection.travel_id    = _Travel.travel_id
  association [1..1] to /DMO/I_Supplement     as _Product        on $projection.supplement_id = _Product.SupplementID
  association [1..*] to /DMO/I_SupplementText as _SupplementText on $projection.supplement_id = _SupplementText.SupplementID
{
  key travel_id,
  key booking_id,
  key booking_supplement_id,
      supplement_id,
      @Semantics.amount.currencyCode: 'currency_code'
      price,
      @Semantics.currencyCode: true
      currency_code,

      @UI.hidden
      _Travel.last_changed_at,  

      /* Associations */
      _Travel, 
      _Booking,
      _Product, 
      _SupplementText
}





```
16.	Save your work.
<br>![](/exercises/ex4/images/04_01_0160.png)

17.	Click “Activate All“.
<br>![](/exercises/ex4/images/04_01_0170.png)

18.	Click “Select All“, then “Activate“.  
<br>![](/exercises/ex4/images/04_01_0180.png)

## Exercise 4.2 Create the Business Object Behavior Definition and Implementation

After completing these steps you will hae created the Behavior Definitionand Implementation for your Busines Object Views.  The behavior definition defines what operations are possible for your Business Object.

1.	Right-click on the Z_I_TRAVEL_M_XXX CDS Artifact and choose “New Behavior Defintion“.
<br>![](/exercises/ex4/images/04_02_0010.png)

2.	Provide the default description, and change the Implementation Type to “Managed”.  Then click “Next”, then “Finish”.
<br>![](/exercises/ex4/images/04_02_0020.png)

3.	You will then see the Behavior Defintion editor. You should see 3 seperate behavior definitions here, one for Travel, Bookings, and BookingSupplement. 
<br>![](/exercises/ex4/images/04_02_0030.png)

4.	Let’s modify this behavior definition.  We’l take it one at a time.  Start with the behavior for **Travel**.  Give the alias and define the implementation class as shown here. Keep the persistant table definition as it is. Uncomment the lock master and etag lines.  For etag, define the field as shown
```abap
define behavior for Z_I_TRAVEL_M_XXX alias travel
implementation in class z_bp_i_travel_m_xxx unique
persistent table /DMO/TRAVEL_M 
lock master
//authorization master ( instance )
etag master last_changed_at
{
  create;
  update;
  delete;
  association _Booking { create; }
}

```
5.	Drop down to the definition for **Booking**.  Update this definition as shown here.
```abap
define behavior for Z_I_BOOKING_M_XXX alias booking
implementation in class Z_BP_I_BOOKING_M_XXX unique
persistent table /DMO/BOOKING_M
lock dependent( travel_id = travel_id )
//authorization dependent( <local_field_name> = <target_field_name> )
etag master last_changed_at
{
  update;
  delete;
  association _BookSupplement { create; }
}

```
6.	Finally drop down tot he definition for **BookingSupplement**.  Update this definition as shown here.
```abap
define behavior for Z_I_BookSuppl_M_XXX alias booksuppl
implementation in class Z_BP_I_BOOKINGSUPPLEMENT_M_XXX unique
persistent table /DMO/BOOKSUPPL_M
etag master last_changed_at
lock dependent( travel_id = travel_id ) //authorization dependent( <local_field_name> = <target_field_name> )

{
  update;
  delete;
}

```
7.	At this point, this is all you need in order to have a working transactional application.  Later, we will come back and add additional functionality to your behavior definition. Save and activate your work.
<br>![](/exercises/ex4/images/04_02_0070.png)


## Exercise 4.3 Create the Projection Views

After completing these steps you will have created Projection Views over your Business Object Views. The Projection Views are meant to further filter the data which is exposed. 

1.	From the Data Definition folder, right-click and choose “New Data Definition“.
<br>![](/exercises/ex4/images/04_03_0010.png)

2.	Enter the name of the projection view as Z_C_TRAVEL_M_XXX where XXX is your group number.  Click “Next”.
<br>![](/exercises/ex4/images/04_03_0020.png)

3.	Click “Next“.
<br>![](/exercises/ex4/images/04_03_0030.png)

4.	Choose Define Projection View from the selection box and click “Finish“.
<br>![](/exercises/ex4/images/04_03_0040.png)

5.	Enter the code as shown here. 
```abap
@EndUserText.label: 'Travel Projection View'
@AccessControl.authorizationCheck: #NOT_REQUIRED

@Metadata.allowExtensions: true

@Search.searchable: true
define root view entity Z_C_TRAVEL_M_XXX 
    as projection on Z_I_Travel_M_XXX
{
 
 key travel_id          as TravelID,


      @Consumption.valueHelpDefinition: [{ entity : {name: '/DMO/I_Agency', element: 'AgencyID'  } }]

      @ObjectModel.text.element: ['AgencyName']
      @Search.defaultSearchElement: true
      agency_id          as AgencyID,
      _Agency.Name       as AgencyName,


      @Consumption.valueHelpDefinition: [{ entity : {name: '/DMO/I_Customer', element: 'CustomerID'  } }]

      @ObjectModel.text.element: ['CustomerName']
      @Search.defaultSearchElement: true
      customer_id        as CustomerID,
      _Customer.LastName as CustomerName,

      begin_date         as BeginDate,

      end_date           as EndDate,

      @Semantics.amount.currencyCode: 'CurrencyCode'
      booking_fee        as BookingFee,

      @Semantics.amount.currencyCode: 'CurrencyCode'
      total_price        as TotalPrice,

      @Consumption.valueHelpDefinition: [{entity: {name: 'I_Currency', element: 'Currency' }}]
      currency_code      as CurrencyCode,

      overall_status     as TravelStatus,

      description        as Description,

      last_changed_at    as LastChangedAt,

      /* Associations */
      _Booking : redirected to composition child Z_C_Booking_M_XXX,
      _Agency,
      _Customer 
    
}

```

6.	Use what you have learned and create another projection view for **Booking**. 
```abap
@EndUserText.label: 'Booking Projection View'
@AccessControl.authorizationCheck: #NOT_REQUIRED

@Metadata.allowExtensions: true

@Search.searchable: true

define view entity Z_C_Booking_M_XXX
  as projection on Z_I_Booking_M_XXX
{

      @Search.defaultSearchElement: true
  key travel_id          as TravelID,

      @Search.defaultSearchElement: true
  key booking_id         as BookingID,

      booking_date       as BookingDate,


      @Consumption.valueHelpDefinition: [{entity: {name: '/DMO/I_Customer', element: 'CustomerID' }}]
      @ObjectModel.text.element: ['CustomerName']
      @Search.defaultSearchElement: true
      customer_id        as CustomerID,
      _Customer.LastName as CustomerName,


      @Consumption.valueHelpDefinition: [{entity: {name: '/DMO/I_Carrier', element: 'AirlineID' }}]
      @ObjectModel.text.element: ['CarrierName']
      carrier_id         as CarrierID,
      _Carrier.Name      as CarrierName,


      @Consumption.valueHelpDefinition: [ {entity: {name: '/DMO/I_Flight', element: 'ConnectionID'},
                                           additionalBinding: [ { localElement: 'FlightDate',   element: 'FlightDate'},
                                                                { localElement: 'CarrierID',    element: 'AirlineID'},
                                                                { localElement: 'FlightPrice',  element: 'Price'},
                                                                { localElement: 'CurrencyCode', element: 'CurrencyCode' } ] } ]
      connection_id      as ConnectionID,


      @Consumption.valueHelpDefinition: [ {entity: {name: '/DMO/I_Flight', element: 'FlightDate' },
                                           additionalBinding: [ { localElement: 'ConnectionID', element: 'ConnectionID'},
                                                                { localElement: 'CarrierID',    element: 'AirlineID'},
                                                                { localElement: 'FlightPrice',  element: 'Price' },
                                                                { localElement: 'CurrencyCode', element: 'CurrencyCode' }]}]
      flight_date        as FlightDate,


      @Semantics.amount.currencyCode: 'CurrencyCode'
      flight_price       as FlightPrice,

      @Consumption.valueHelpDefinition: [{entity: {name: 'I_Currency', element: 'Currency' }}]
      currency_code      as CurrencyCode,

      booking_status     as BookingStatus,

      @UI.hidden: true
      last_changed_at    as LastChangedAt, -- Take over from parent

      /* Associations */
      _Travel         : redirected to parent Z_C_Travel_M_XXX,
      _BookSupplement : redirected to composition child Z_C_BookSuppl_M_XXX,
      _Customer,
      _Carrier

}

```

7.	Use what you have learned and create another projection view for **BookingSupplement** called Z_C_BOOKSUPPL_M_XXX where XXX is your group number. Enter the following code as shown here. 
```abap
@EndUserText.label: 'Booking Supplement Projection View'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@Metadata.allowExtensions: true

@Search.searchable: true

define view entity Z_C_BookSuppl_M_XXX 
     as projection on Z_I_BookSuppl_M_XXX
{

    @Search.defaultSearchElement: true
    key travel_id                     as TravelID,
  
    @Search.defaultSearchElement: true
    key booking_id                    as BookingID,

    key booking_supplement_id         as BookingSupplementID,

    @Consumption.valueHelpDefinition: [ {entity: {name: '/DMO/I_SUPPLEMENT', element: 'SupplementID' } ,
                                         additionalBinding: [ { localElement: 'Price',  element: 'Price' },
                                                              { localElement: 'CurrencyCode', element: 'CurrencyCode' }] }]
    @ObjectModel.text.element: ['SupplementDescription']
    supplement_id                     as SupplementID,
    _SupplementText.Description       as SupplementDescription: localized,            
    

    @Semantics.amount.currencyCode: 'CurrencyCode'  
    price                             as Price,

    @Consumption.valueHelpDefinition: [{entity: {name: 'I_Currency', element: 'Currency' }}]
    currency_code                     as CurrencyCode,
    
    last_changed_at                   as LastChangedAt,

    /* Associations */
    _Booking: redirected to parent Z_C_Booking_M_XXX,
    _SupplementText    
}

```

8.	Save your work and click “Activate All“.
<br>![](/exercises/ex4/images/04_03_0080.png)

9.	Click “Select All“, and “Activate“.
<br>![](/exercises/ex4/images/04_03_0090.png)


## Exercise 4.4 Create the Metadata Extensions

After completing these steps have created the Metadata Extensions. The Metadata Extensions are meant to seperate the UI specific annotations from the rest of the data model.  This makes for much cleaner code.

1.	Right-click on the Z_C_TRAVEL_M_XXX projection view and choose “New Metadata Extension“.
<br>![](/exercises/ex4/images/04_04_0010.png)

2.	Give the name as the same as the view, Z_C_TRAVEL_M_XXX where XXX is your group number.  Give a meaningful description and click “Next”, then “Finish”.
<br>![](/exercises/ex4/images/04_04_0020.png)

3.	Enter the code as shown here. 
```abap
@Metadata.layer: #CORE

@UI: { headerInfo: { typeName: 'Travel', 
                     typeNamePlural: 'Travels', 
                     title: { type: #STANDARD, 
                              value: 'TravelID' } } }

annotate view Z_C_TRAVEL_M_XXX
    with 
{
     @UI.facet: [ { id:              'Travel',
                     purpose:         #STANDARD,
                     type:            #IDENTIFICATION_REFERENCE,
                     label:           'Travel',
                     position:        10 },
                   { id:              'Booking',
                     purpose:         #STANDARD,
                     type:            #LINEITEM_REFERENCE,
                     label:           'Booking',
                     position:        20,
                     targetElement:   '_Booking'}]

      @UI: {
          lineItem:       [ { position: 10, importance: #HIGH } ],
          identification: [ { position: 10, label: 'Travel ID [1,...,99999999]' } ] }
     TravelID;

      @UI: {
          lineItem:       [ { position: 20, importance: #HIGH } ],
          identification: [ { position: 20 } ],
          selectionField: [ { position: 20 } ] }

      AgencyID;

      @UI: {
          lineItem:       [ { position: 30, importance: #HIGH } ],
          identification: [ { position: 30 } ],
          selectionField: [ { position: 30 } ] }

       CustomerID;

      @UI: {
          lineItem:       [ { position: 40, importance: #MEDIUM } ],
          identification: [ { position: 40 } ] }
      BeginDate;

      @UI: {
          lineItem:       [ { position: 41, importance: #MEDIUM } ],
          identification: [ { position: 41 } ] }
       EndDate;


      @UI: {
          identification: [ { position: 42 } ] }     
       BookingFee;

      @UI: {
          lineItem:       [ { position: 43, importance: #MEDIUM } ],
          identification: [ { position: 43, label: 'Total Price' } ] }
       TotalPrice;

      @UI: {
          lineItem:       [ { position: 15, importance: #HIGH },
                            { type: #FOR_ACTION, dataAction: 'acceptTravel', label: 'Accept Travel' },
                            { type: #FOR_ACTION, dataAction: 'rejectTravel', label: 'Reject Travel' } ],
          identification: [ { position: 15 }, 
                            { type: #FOR_ACTION, dataAction: 'acceptTravel', label: 'Accept Travel' },
                            { type: #FOR_ACTION, dataAction: 'rejectTravel', label: 'Reject Travel' } ] ,
          selectionField: [ { position: 40 } ] }
       TravelStatus;

      @UI: {
          identification:[ { position: 46 } ]  }
       Description;

      @UI.hidden: true
       LastChangedAt;
    
}

```
4.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_04_0040.png)

5.	Use what you have learned and create another metadata extension for Z_C_BOOKING_M_XXX where XXX is your group numnber.  Enter the code as shown here. 
```abap
@Metadata.layer: #CORE

@UI: {
  headerInfo: { typeName: 'Booking',
                typeNamePlural: 'Bookings',
                title: { type: #STANDARD, value: 'BookingID' } } }


annotate view Z_C_Booking_M_XXX
    with 
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


      TravelID;

      @UI: { lineItem:       [ { position: 20, importance: #HIGH } ],
             identification: [ { position: 20 } ] }
      @Search.defaultSearchElement: true
      BookingID;

      @UI: { lineItem:       [ { position: 30, importance: #HIGH } ],
             identification: [ { position: 30 } ] }
      BookingDate;

      @UI: { lineItem:       [ { position: 40, importance: #HIGH } ],
             identification: [ { position: 40 } ] }
      CustomerID;


      @UI: { lineItem:       [ { position: 50, importance: #HIGH } ],
             identification: [ { position: 50 } ] }
      CarrierID;


      @UI: { lineItem:       [ { position: 60, importance: #HIGH } ],
             identification: [ { position: 60 } ] }
      ConnectionID;

      @UI: { lineItem:       [ { position: 70, importance: #HIGH } ],
             identification: [ { position: 70 } ] }
     FlightDate;

      @UI: { lineItem:       [ { position: 80, importance: #HIGH } ],
             identification: [ { position: 80 } ] }
      FlightPrice;

      @UI: { lineItem:       [ { position: 90, importance: #HIGH, label: 'Status' } ],
             identification: [ { position: 90, label: 'Status [N(New)| X(Canceled)| B(Booked)]' } ] }
     BookingStatus;

      @UI.hidden: true
      LastChangedAt; -- Take over from parent
    
}

```


6.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_04_0060.png)

7.	Use what you have learned and create another metadata extension for Z_C_BOOKSUPPL_M_XXX where XXX is your group numnber.  Enter the code as shown here.
```abap
@Metadata.layer: #CORE

@UI: { headerInfo: { typeName:       'Booking Supplement',
                     typeNamePlural: 'Booking Supplements',
                     title:          { type: #STANDARD, 
                                       label: 'Booking Supplement', 
                                       value: 'BookingSupplementID' } } }

annotate view Z_C_BookSuppl_M_XXX
    with 
{


@UI.facet: [ { id:              'BookingSupplement',
                   purpose:         #STANDARD,
                   type:            #IDENTIFICATION_REFERENCE,
                   label:           'Booking Supplement',
                   position:        10 }  ]

    @Search.defaultSearchElement: true
    TravelID;
  
    @Search.defaultSearchElement: true
    BookingID;

    @UI: { lineItem:       [ { position: 10, importance: #HIGH } ],
           identification: [ { position: 10 } ] }
    BookingSupplementID;

    @UI: { lineItem:       [ { position: 20, importance: #HIGH } ],
           identification: [ { position: 20 } ] }
    SupplementID;

    @UI: { lineItem:       [ { position: 30, importance: #HIGH } ],
           identification: [ { position: 30 } ] }
    Price;
    
    @UI.hidden
    LastChangedAt;
    
}
```
8.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_04_0080.png)

## Exercise 4.5 Create the Behavior Definition Projection

After completing these steps you will have created the Behavior Definition Projection.  This is meant to further restrict the behavior operations. The idea is that you will have one monolithic Business Object data model, while having several Projection Views and Behavior Projections on top. For example, if there were different operations that would be allowed for a "processor" and a "manager", these would be two different Projection Views and Behavior Projections, each exposing differnt data and/or allowing different operations.

1.	Right-click on the Z_C_TRAVEL_M_XXX projection view and choose “New Behavior Definition“.
<br>![](/exercises/ex4/images/04_05_0010.png)

2.	Accept the defaults and click “Next“, then “Finish“.
<br>![](/exercises/ex4/images/04_05_0020.png)

3.	Give aliases to all of the behavior definitions, add the user etag and comment out all use delete statements. 
```abap
projection;

define behavior for Z_C_TRAVEL_M_XXX alias travel
use etag
{
  use create;
  use update;
//  use delete;
  use association _BOOKING { create; }
}

define behavior for Z_C_Booking_M_XXX alias booking
use etag
{
  use update;
//  use delete;
  use association _BOOKSUPPLEMENT { create; }
}

define behavior for Z_C_BookSuppl_M_XXX alias bookingsupplement
use etag
{
  use update;
//  use delete;
}

```
4.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_05_0040.png)


## Exercise 4.6 Create the Service Definition and Service Binding

After completing these steps you will have created the Service Definition which exposes the Projection Views, and the Service Binding which binds the Service Definition to a specific protocal and usage.

1.	Right-click on the “Service Definition“ folder and choose “New Service Definition“.
<br>![](/exercises/ex4/images/04_06_0010.png)

2.	Enter the name as Z_SD_C_TRAVEL_M_XXX where XXX is your group number.  “Next“, then “Finish“.
<br>![](/exercises/ex4/images/04_06_0020.png)

3.	Enter the code as shown here.  
```abap
@EndUserText.label: 'Service Definition for Travel'
define service Z_SD_C_TRAVEL_M_XXX {
  expose Z_C_Travel_M_XXX as Travel;
  expose Z_C_Booking_M_XXX as Booking;
  expose Z_C_BookSuppl_M_XXX as BookingSupplement;
    
}
```
4.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_06_0040.png)

5.	Right-click on the service definition and choose “New Service Binding“.
<br>![](/exercises/ex4/images/04_06_0050.png)

6.	Give the name as Z_UI_C_TRAVEL_M_XXX where XXX is your group number.  Click “Next“, then “Finish“.
<br>![](/exercises/ex4/images/04_06_0060.png)

7.	Click “Activate“.
<br>![](/exercises/ex4/images/04_06_0070.png)

8.	Select the **Travel** entity and click “Preview“.
<br>![](/exercises/ex4/images/04_06_0080.png)

9.	The browser should open, log in and view your application preview.   Play around with the create, update, and delete functionality. Again, these operations are handled for you by the framework. 
<br>![](/exercises/ex4/images/04_06_0090.png)

## Exercise 4.7 Add Validations to the Behavior Definition

After completing these steps you will have added additional validation to your Behavior Definitions for checking the values for *Customer* and *Begin_Date* and *End_Date*.

1.	Return to your behavior definition called  Z_I_TRAVEL_M_XXX.  After the delete statement für behavior Z_I_TRAVEL_M_XXX, enter the following code. 
```abap
   // validations
  validation validateCustomer on save { field customer_id; }
  validation validateDates    on save { field begin_date, end_date; }

```
2.	Save and activate your work.
<br>![](/exercises/ex4/images/04_07_0020.png)

3.	Now that we have some logic that we need to implement, we can now create our behavior implementation class for Z_I_TRAVEL_M_XXX.  Right-click on the behavior definition and choose “New Behavior Definition“.
<br>![](/exercises/ex4/images/04_07_0030.png)

4.	Enter the name of the class as Z_BP_I_TRAVEL_M_XXX where XXX is your group number.  Click “Next“, then “Finish“.
<br>![](/exercises/ex4/images/04_07_0040.png)

5.	Click “Local Types“ tab.  You will see that the local class defintion has been inserted for you with the validation methods that you have defined in the behavior definition.   
<br>![](/exercises/ex4/images/04_07_0050.png)

6.	In the *validateCustomer* method, enter the following code.  Make sure to substitute your group number where you see XXX in this code. 
```abap
  METHOD validateCustomer.
 
  READ ENTITY Z_I_Travel_M_XXX\\travel FIELDS ( customer_id ) WITH
      VALUE #( FOR <root_key> IN keys ( %key = <root_key> ) )
      RESULT DATA(lt_travel).

    DATA lt_customer TYPE SORTED TABLE OF /dmo/customer WITH UNIQUE KEY customer_id.

    " Optimization of DB select: extract distinct non-initial customer IDs
    lt_customer = CORRESPONDING #( lt_travel DISCARDING DUPLICATES MAPPING customer_id = customer_id EXCEPT * ).
    DELETE lt_customer WHERE customer_id IS INITIAL.
    CHECK lt_customer IS NOT INITIAL.

    " Check if customer ID exist
    SELECT FROM /dmo/customer FIELDS customer_id
      FOR ALL ENTRIES IN @lt_customer
      WHERE customer_id = @lt_customer-customer_id
      INTO TABLE @DATA(lt_customer_db).

    " Raise msg for non existing customer id
    LOOP AT lt_travel INTO DATA(ls_travel).
      IF ls_travel-customer_id IS NOT INITIAL AND NOT line_exists( lt_customer_db[ customer_id = ls_travel-customer_id ] ).
        APPEND VALUE #(  travel_id = ls_travel-travel_id ) TO failed.
        APPEND VALUE #(  travel_id = ls_travel-travel_id
                         %msg      = new_message( id       = '/DMO/CM_FLIGHT_LEGAC'
                                                  number   = '002'
                                                  v1       = ls_travel-customer_id
                                                  severity = if_abap_behv_message=>severity-error )
                         %element-customer_id = if_abap_behv=>mk-on ) TO reported.
      ENDIF.

    ENDLOOP. 
 
  ENDMETHOD.

```
7.	In the *validateDates* method, enter the following code.  Make sure to substitute your group number where you see XXX in this code.
```abap
  METHOD validateDates.
  
  READ ENTITY Z_I_Travel_M_XXX\\travel FIELDS ( begin_date end_date )  WITH
        VALUE #( FOR <root_key> IN keys ( %key = <root_key> ) )
        RESULT DATA(lt_travel_result).

    LOOP AT lt_travel_result INTO DATA(ls_travel_result).

      IF ls_travel_result-end_date < ls_travel_result-begin_date.  "end_date before begin_date

        APPEND VALUE #( %key        = ls_travel_result-%key
                        travel_id   = ls_travel_result-travel_id ) TO failed.

        APPEND VALUE #( %key     = ls_travel_result-%key
                        %msg     = new_message( id       = /dmo/cx_flight_legacy=>end_date_before_begin_date-msgid
                                                number   = /dmo/cx_flight_legacy=>end_date_before_begin_date-msgno
                                                v1       = ls_travel_result-begin_date
                                                v2       = ls_travel_result-end_date
                                                v3       = ls_travel_result-travel_id
                                                severity = if_abap_behv_message=>severity-error )
                        %element-begin_date = if_abap_behv=>mk-on
                        %element-end_date   = if_abap_behv=>mk-on ) TO reported.

      ELSEIF ls_travel_result-begin_date < cl_abap_context_info=>get_system_date( ).  "begin_date must be in the future

        APPEND VALUE #( %key        = ls_travel_result-%key
                        travel_id   = ls_travel_result-travel_id ) TO failed.

        APPEND VALUE #( %key = ls_travel_result-%key
                        %msg = new_message( id       = /dmo/cx_flight_legacy=>begin_date_before_system_date-msgid
                                            number   = /dmo/cx_flight_legacy=>begin_date_before_system_date-msgno
                                            severity = if_abap_behv_message=>severity-error )
                        %element-begin_date = if_abap_behv=>mk-on
                        %element-end_date   = if_abap_behv=>mk-on ) TO reported.
      ENDIF.

    ENDLOOP.
  
  ENDMETHOD.

```
8.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_07_0080.png)

9.	Once again, launch the preview from the service binding and check that your validations are being executed when creating a travel record.
<br>![](/exercises/ex4/images/04_07_0090.png)


## Exercise 4.8 Add Field Attributes and Custom Actions to the Behavior Definition

After completing these steps you will have added  Field Attributes for defining mandatory fields statically, and defining these Field Attributes dynamically in the implementation.  You will have also created Custom Actions for your Behavior.

1.	Go back to the Z_I_TRAVEL_M_XXX behavior definition. Before the create statement, add the following lines.  Here we are setting fields as mandatory, and also setting the travel_id field so that we can set the attributes of this field programatically in our behavior implementation.  We will circle back on that later.
```abap
  // mandatory fields that are required to create a travel
  field ( mandatory ) agency_id, overall_status, booking_fee, currency_code;
  field (features : instance ) travel_id;

```

2.	Before the validation statements, enter the following codes as shown here.
```abap
// instance action and dynamic action control
  action  ( features: instance ) acceptTravel result [1] $self;
  action  ( features: instance ) rejectTravel result [1] $self;

```

3.	Save and activate your work.
<br>![](/exercises/ex4/images/04_08_0030.png)

4.	Return to the projection behavior definition called Z_C_TRAVEL_M_XXX.  As we have created new actions in our business object behavior definition, we now have to expose these in our projection as well.  Add the following lines tot he Z_C_TRAVEL_M_XXX definition after the commented *use delete* statement.
```abap
  use action acceptTravel;
  use action rejectTravel;

```

5.	Return to the behavior implementation called Z_BP_I_TRAVEL_M_XXX. Define two new methods for handling the *accept* and *reject* actions.
```abap
    METHODS set_status_completed  FOR MODIFY 
      IMPORTING   keys FOR ACTION travel~acceptTravel RESULT result.
    METHODS set_status_cancelled  FOR MODIFY 
      IMPORTING   keys FOR ACTION travel~rejectTravel RESULT result.

```

6.	Add the implementation fort he *set_status_completed* method as shown here.  Make sure to replace XXX with your group number.
```abap
  METHOD set_status_completed.

    " Modify in local mode: BO-related updates that are not relevant for authorization checks
    MODIFY ENTITIES OF Z_I_Travel_M_XXX IN LOCAL MODE
           ENTITY travel
              UPDATE FIELDS ( overall_status )
                 WITH VALUE #( for key in keys ( travel_id      = key-travel_id
                                                 overall_status = 'A' ) ) " Accepted
           FAILED   failed
           REPORTED reported.

    " Read changed data for action result
    READ ENTITIES OF Z_I_Travel_M_XXX IN LOCAL MODE
         ENTITY travel
           FIELDS ( agency_id
                    customer_id
                    begin_date
                    end_date
                    booking_fee
                    total_price
                    currency_code
                    overall_status
                    description
                    created_by
                    created_at
                    last_changed_at
                    last_changed_by )
             WITH VALUE #( for key in keys ( travel_id = key-travel_id ) )
         RESULT DATA(lt_travel).

    result = VALUE #( for travel in lt_travel ( travel_id = travel-travel_id
                                                %param    = travel ) ).

  ENDMETHOD.

```
7.	Add the implementation for the *set_status_cancelled* method as shown here. Make sure to replace XXX with your group number.
```abap
METHOD set_status_cancelled.

    MODIFY ENTITIES OF Z_I_Travel_M_XXX  IN LOCAL MODE
           ENTITY travel
              UPDATE FROM VALUE #( for key in keys ( travel_id = key-travel_id
                                                     overall_status = 'X'   " Canceled
                                                     %control-overall_status = if_abap_behv=>mk-on ) )
           FAILED   failed
           REPORTED reported.

    " read changed data for result
        READ ENTITIES OF Z_I_Travel_M_XXX IN LOCAL MODE
         ENTITY travel
           FIELDS ( agency_id
                    customer_id
                    begin_date
                    end_date
                    booking_fee
                    total_price
                    currency_code
                    overall_status
                    description
                    created_by
                    created_at
                    last_changed_at
                    last_changed_by )
             WITH VALUE #( for key in keys ( travel_id = key-travel_id ) )
         RESULT DATA(lt_travel).

    result = VALUE #( for travel in lt_travel ( travel_id = travel-travel_id
                                                %param    = travel
                                              ) ).

  ENDMETHOD.

```
8.	We want to control the display have these accept and reject buttons programmically.  In the behavior definition we used the keyword “feature“ when defining the actions.  Now in our implementation, we can get access to these features and control access to them.  Once again, add a new method called *get_features*.
```abap
   METHODS get_features  FOR FEATURES 
      IMPORTING keys REQUEST  requested_features FOR travel  RESULT result.

```
9.	Add the implementation fort he set_status_cancelled method as shown here. Make sure to replace XXX with your group number. This method reads the entity and gives us access to the actions and allows us to set the buttons to enable/disabled based on that selected lines current status.  Here we are also programmically setting the “Travel ID“ field to read-only in edit mode.
```abap
METHOD get_features.

    READ ENTITY Z_I_Travel_M_XXX
         FIELDS (  travel_id overall_status )
           WITH VALUE #( FOR keyval IN keys (  %key = keyval-%key ) )
         RESULT DATA(lt_travel_result).


    result = VALUE #( FOR ls_travel IN lt_travel_result
                       ( %key                           = ls_travel-%key
                         %field-travel_id               = if_abap_behv=>fc-f-read_only
                         %features-%action-rejectTravel = COND #( WHEN ls_travel-overall_status = 'X'
                                                                    THEN if_abap_behv=>fc-o-disabled 
                                                                      ELSE if_abap_behv=>fc-o-enabled  )
                         %features-%action-acceptTravel = COND #( WHEN ls_travel-overall_status = 'A'
                                                                    THEN if_abap_behv=>fc-o-disabled 
                                                                      ELSE if_abap_behv=>fc-o-enabled   )
                      ) ).

  ENDMETHOD.

```
10.	Save and activate your work. 
<br>![](/exercises/ex4/images/04_08_0100.png)

11.	Once again, launch the preview from the service binding and check that your new buttons are working correctly.
<br>![](/exercises/ex4/images/04_08_0110.png)

12.	Also, check that your fields have been marked as mandatory.
<br>![](/exercises/ex4/images/04_08_0120.png)

## Summary

You've now created your Business Object data model, defined and implemented your Behaviors, created Projections over both, added Validations and Field Attributes, and exposed your data model as an Odata service using the Managed Scenario of the RESTful Programming Model

