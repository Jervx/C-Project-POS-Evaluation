#### Point Of Sale Built on C

I did'nt include the C source file because some will just copy the code & claim it as theirs.

##### 

##### Features
* Have its own Configuration file *
* JSON Database for Accounts *
* JSON Database for Products *
* JSON Database for Sale History *
* Add New Products(Append new Data in Products.json) *
* Edit Existing Products(Alter Data in Products.json) *
* Save Copy of Transaction Receipts no. & info(Append new Record in Reports.json) *
* Receipt Directory(Every transaction saves here are the complete copy of receipt)*
* Can Change Tax & Discount value *
* Can Change 
* Can Change Store Name *
* Can Change Receipt Message *
* Can Change Theme *
..*Light
..*Light Gray
..*Gray Light
..*Dark Gray
..*Dark


###### Flaw Of This Application
* The editing Tax, Adding new Products, Editing new Products can be accessed by cashier
* No Sale Statistics

###### Possible Improvements Suggested by the respondents of our project
* The Admin stuff like (editing Tax, Adding new Products, Editing new Products) can only be edited by admin user
* Implement Statistics like (Sales Stat for Today, For this week, and for Months)

###### Notes
* Pos Cannot Run without the configuration file "config.txt"
```
 This is because we created a program that is dynamic & will always
 based its data on the files that is manipulated before rather than
 using a variable that can be lost its data after closing the program
 ```

