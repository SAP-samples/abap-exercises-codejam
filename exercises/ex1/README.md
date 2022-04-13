# Exercise 1 - Hello World

In this exercise, we will create a very simple "Hello World" class which outputs to the console.  In the second part of the exercise, we will create another class which SELECTs data from the database and displays the list in the console.   For more information about system access and package creation, please see the [Getting Started](../ex0/README.md) section.

## Exercise 1.1 Creating "Hello World" class

After completing these steps you will have created a "Hello World" ABAP Class.

1. Right-click on your package and choose “New“, “ABAP Class“.
<br>![](/exercises/ex1/images/01_01_0010.png)

2. Give the name of the class as "ZCL_HELLO_XXX" where XXX is your group number.  Then click "Next".
<br>![](/exercises/ex1/images/01_01_0020.png)

3.	Click "Finish"
<br>![](/exercises/ex1/images/01_01_0030.png)

4.	In the PUBLIC section of your class,  define the use of the interface IF_OO_ADT_CLASSRUN.  This interface basically gives you access to the console in ADT, which allows the developer to write the results to the console.
<br>![](/exercises/ex1/images/01_01_0040.png)

5.	Next, we need to implement the MAIN method of the inteface.  Enter the code as shown below.  Here the inteface provides access to the "out" instance where we call the "write" method and pass the string "Hello World".
<br>![](/exercises/ex1/images/01_01_0050.png)

6.	Save and activate your class.
<br>![](/exercises/ex1/images/01_01_0060.png)

7.	Next, execute the class by hitting F9.  You should then see "Hello World" in the console tab in ADT.
<br>![](/exercises/ex1/images/01_01_0070.png)

## Exercise 1.2 SELECT data from a database table

After completing these steps you will have created another class which selects data from a database table and outputs the result set to the console.

1.	Use what you have learned and create another class called "ZCL_SELECT_XXX" where XXX is your group number.  In the public section, define the use of interface IF_OO_ADT_CLASSRUN and add the METHOD statements to the implementation section.
<br>![](/exercises/ex1/images/01_02_0010.png)

2.	Next, add the following statements to the PRIVATE section of your class. Here we are defining a table type which contains the structure of the database table which we want to read from.  Also, the METHODS statement defines that your class has a method called "GET_CARRIERS" which returns a list of carriers. 
<br>![](/exercises/ex1/images/01_02_0020.png)

3.	Next, add the implementation of the GET_CARRIERS method as shown.  Here we are doing a SELECT against the database table /DMO/CARRIER and putting results into the RETURNING parameter of the method.
<br>![](/exercises/ex1/images/01_02_0030.png)

4.	Now go to the implementation for the MAIN method and add a statement which calls the GET_CARRIER method and outputs the result set to the console.
<br>![](/exercises/ex1/images/01_02_0040.png)

5.	Save and activate your class
<br>![](/exercises/ex1/images/01_02_0050.png)

6.	Now execute your class by hitting F9.  THe results should be shown in the console.
<br>![](/exercises/ex1/images/01_02_0060.png)

## Summary

You've now created two classes showing how to output results to the console.

Continue to - [Exercise 2 - Exposing and Consuming Services via HTTP ](../ex2/README.md)
