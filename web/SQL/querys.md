



#### **SELECT**


users:

![[Pasted image 20251005090653.png]]

**select * from users;**

The first word SELECT, tells the database we want to retrieve some data; the * tells the database we want to receive back all columns from the table. For example, the table may contain three columns (id, username and password). "from users" tells the database we want to retrieve the data from the table named users. Finally, the semicolon at the end tells the database that this is the end of the query.

**select username,password from users;**

The next query is similar to the above, but this time, instead of using the * to return all columns in the database table, we are just requesting the username and password field.

![[Pasted image 20251005104413.png]]

**select * from users LIMIT 1;**

The following query, like the first, returns all the columns by using the * selector, and then the "LIMIT 1" clause forces the database to return only one row of data. Changing the query to 
"**LIMIT 1,1**" forces the query to skip the first result, and then "**LIMIT 2,1**" skips the first two results, and so on. You need to remember the first number tells the database how many results you wish to skip, and the second number tells the database how many rows to return
![[Pasted image 20251005104449.png]]

**`select * from users where username='admin';`**
Lastly, we're going to utilise the where clause; this is how we can finely pick out the exact data we require by returning data that matches our specific clauses:
![[Pasted image 20251005104540.png]]

  
**select * from users where username != 'admin';**
![[Pasted image 20251005104621.png]]

**select * from users where username='admin' or username='jon';**
![[Pasted image 20251005104716.png]]
**`select * from users where username='admin' and password='p4ssword';`**
![[Pasted image 20251005104723.png]]
  
**select * from users where username like 'a%';**
returns rows starting with the letter a

Using the like clause allows you to specify data that isn't an exact match but instead either **starts**, **contains** or **ends** with certain characters by choosing where to place the wildcard character represented by a percentage sign %.
![[Pasted image 20251005104816.png]]**`select * from users where username like '%n';

This returns any rows with a username ending with the letter n.
![[Pasted image 20251005104844.png]]

`select * from users where username like '%mi%';`
This returns any rows with a username containing the characters **mi** within them.
![[Pasted image 20251005105018.png]]

**
#### **UNION**  
**

The UNION statement combines the results of two or more SELECT statements to retrieve data from either single or multiple tables; the rules to this query are that the UNION statement must retrieve the same number of columns in each SELECT statement, the columns have to be of a similar data type, and the column order has to be the same. This might sound not very clear, so let's use the following analogy. Say a company wants to create a list of addresses for all customers and suppliers to post a new catalogue. We have one table called customers with the following content
![[Pasted image 20251005105129.png]]And another called suppliers with the following contents:
![[Pasted image 20251005105143.png]]Using the following SQL Statement, we can gather the results from the two tables and put them into one result set:
`SELECT name,address,city,postcode from customers UNION SELECT company,address,city,postcode from suppliers;`


#### **INSERT**

The **INSERT** statement tells the database we wish to insert a new row of data into the table. **"into users"** tells the database which table we wish to insert the data into, **"(username,password)"** provides the columns we are providing data for and then **"values ('bob','password');"** provides the data for the previously specified columns.

**insert into users (username,password) values ('bob','password123');**

#### UPDATE

The **UPDATE** statement tells the database we wish to update one or more rows of data within a table. You specify the table you wish to update using "**update %tablename% SET**" and then select the field or fields you wish to update as a comma-separated list such as "**username='root',password='pass123'**" then finally, similar to the SELECT statement, you can specify exactly which rows to update using the where clause such as "**where username='admin;**".

  

`update users SET username='root',password='pass123' where username='admin';`


#### DELETE

The **DELETE** statement tells the database we wish to delete one or more rows of data. Apart from missing the columns you wish to return, the format of this query is very similar to the SELECT. You can specify precisely which data to delete using the **where** clause and the number of rows to be deleted using the **LIMIT** clause.

  

`delete from users where username='martin';`

`delete from users;`
Because no WHERE clause was being used in the query, all the data was deleted from the table.
![[Pasted image 20251005105552.png]]



