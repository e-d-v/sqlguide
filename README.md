# Databases and SQL

Welcome to Social Coding's final Workshop of the semester - Databases and SQL! This workshop will cover all the basics of working with database systems, such as SQL queries, database schema and an introduction to a variety of frameworks that are commonly used today. You can use all these topics in your projects immediately - whether they be in Social Coding or on your own time. This workshop series is taught by [Kanishk Kacholia](mailto:kacho005@umn.edu) and [Evan Voogd](mailto:voogd004@umn.edu) of Social Coding.

# Week 2: Introduction to SQL Commands
1. [How to Run SQL Commands](#RUN)
2. [Commands](#CMDS)
   1. [`CREATE`](#CREATE)
      1. [Constraints](#CONS)
   2. [`INSERT INTO`](#INSERT)
   3. [`SELECT`](#SELECT) 
      1. [`WHERE`](#WHERE)
   4. [`DELETE`](#DELETE)
   5. [`ALTER`](#ALTER)
      1. [`ADD`](#ADD)  
      2. [`DROP`](#DROP)
3. [Walkthrough](#WALK) 
4. [Project](#PROJ)

## How to run SQL Commands<a id="RUN"/>
For this workshop, we'll be using LibreOffice Base as our Database environment. This isn't optimal, as we described last week, but it is the easiest environment to use on lab machines, so this is what we'll be using. All of these commands should work with any SQL-compliant database, however. Firstly, make sure that you're logged into a CSE lab machine in your browser, the department has a guide to connecting to one <a href="https://cse.umn.edu/cseit/self-help-guides/connect-linux-cse-labs-computer-graphical-user-interface">here</a>. Once you're logged in, click the "Activities" button in the top left corner and search for "LibreOffice Base". On this screen click "Finish" to keep the default options and save your database with a helpful name and location. 

<img src="https://i.imgur.com/nnjGGna.jpg" alt="The default screen upon opening LibreOffice Base" width="48%" /> <img src="https://i.imgur.com/PMHuvbZ.jpg" alt="The save file picker in GNOME" width="48%" align="right" />

LibreOffice has a few graphical tools to automate commands, but as this workshop is teaching you exactly <i>how</i> databases work, we'll be sticking to SQL. To run a SQL command click "Tools->SQL..." in the menu bar, and type your command into the "Command to execute:" box. When you're done, click the "Execute" button.

<div align="center">
<img src="https://i.imgur.com/pRcuJ9b.jpg" alt="The Tools->SQL... button in LibreOffice Base" width="50%" />
</div>

## Commands<a id="CMDS"/>
Creating tables, inserting data, and querying your database are all done with SQL commands. SQL Commands are case-insensitive, but by convention commands like `CREATE` or `REFERENCES` are put in all-caps to distinguish them from the fields themselves. Similarly, whitespace doesn't matter - you can put new lines and commas pretty much wherever you want, although we recommend that you adopt a consistent convention that isn't putting everything on one line for readability's sake. One quirk of LibreOffice Base to note is that user defined strings like table names, field names, and constraint names should be put in quotation marks. Not all SQL implementations require this, so if you end up using outside resources this is something to be aware of.

### `CREATE`<a id="CREATE"/>
As we discussed last week, all of our data in our database live in Tables, which are essentially analagous to Classes in object-oriented systems. Creating a table in SQL is fairly simple, and is done using the `CREATE TABLE` command. These commands roughly follow this structure:  
```
CREATE TABLE "[TableName]" (  
  [Field1] TYPE1,  
  [Field2] TYPE2,  
  ...  
  [FieldN] TYPEN  
);
```

For instance, if we were creating a table for a menu item at a restaraunt we might have something like this:  
```
CREATE TABLE "Items" (  
  "Name" VarChar(255),  
  "Price" Float,  
  "Calories" Int  
 );  
```
#### Constraints<a id="CONS"/>
Sometimes there are limitations that you want to put on a field in your table. For instance, you might want to ensure that a field is always filled or identify that an individual data item can be identified using one specific unique field. For these situations, constraints come in handy.

To handle this first case, we can add `NOT NULL` at the end of the field declaration in `CREATE TABLE`. For instance, if we want to make sure each item has a name, we could replace the third line with `"Price" Float NOT NULL,`.

To handle this second case, we can use to seperate constraints. First, we want to create a primary key. A primary key is an identifier that we can use to lookup a specific value without needing to know the other fields. We can do this by adding a constraint to our `CREATE TABLE` command:
```
CREATE TABLE "Items" (
   "Name" VarChar(255),
   "Price" Float,
   "Calories" Int,
   CONSTRAINT "PK_Item" PRIMARY KEY ("Name")
 );
 ```
This constraint sets `"Name"` as the identifying field of this class. However, using a field can create problems. Some tables inherently have unique identifying fields, such as a student ID in a Student table. However, it's certainly possible that menu items could have duplicate names, so using "Name" as our primary key is problematic. Price and Calories have similar problems. Because of this, we want to add a separate field for the primary key, which is usually called "ID". As the ID has no inherent meaning other than for identifying the data, we can autogenerate it so we don't have to worry about it. This constraint goes by different names depending on implementation, in some it's referred to as `AUTO INCREMENT` but in LibreOffice Base it's `IDENTITY`. Adding this field to our item Table looks like:
```
CREATE TABLE "Items" (
  "ID" INT IDENTITY,
  "Name" VarChar(255),
  "Price" Float,
  "Calories" Int,
  CONSTRAINT "PK_Item" PRIMARY KEY ("ID")
);
```

### `INSERT INTO`<a id="INSERT"/>
Now that we have a table, we want to add data to it. This is done using the `INSERT INTO` command. This command takes the general form of:
```
INSERT INTO "[TABLE NAME]" ("[FIELD1]", "[FIELD2]", ..., "[FIELD3]")
  VALUES ("[VAL1]", "[VAL2]", ..., "[VAL3]");
```
If we wanted to insert a hamburger with 430 calories that costs $3.20 into our "Items" table from earlier we would simply run:
```
INSERT INTO "Items" ("Name", "Price", "Calories")
  VALUES ("Hamburger", 3.20, 430);
```
Note that we didn't include the "ID" field in our command. This is because it's automatically generated, so we don't need to worry about specifying it in our command. This goes for all non-null commands as well. For instance, if we want to add a cheeseburger, but don't know how many calories it contains, we can run the following command and it will simply leave the "Calories" field blank.
```
INSERT INTO "Items" ("Name", "Price")
  VALUES ('Cheeseburger', 3.20);
```

### `SELECT`<a id="SELECT"/>
Now that our table contains data, how do we actually access it? We do this using the `SELECT` command. If we want to see all the data in our table, we can run `SELECT * FROM [TableName]`. For instance, if we want to see our data in the "Items" table, we can run `SELECT * FROM "Items"`, which outputs:
```
0,Hamburger,3.2,430,
1,Cheeseburger,3.2,,
```

#### `WHERE`<a id="WHERE"/>
Often, our tables have a large amount of data, and we want to filter it down to a more manageable size. To do this, we add a `WHERE` clause to our `SELECT` command. For instance, if we have a more populated table and want to see all items with a price under $2, we could run:
```
SELECT * FROM "Items"
  WHERE "Price"<2.00;
```
Which returns:
```
2,Fries,1.99,300,
3,Soda,0.99,180,
```
We can also run more complex queries using the keywords `AND` and `OR`, which exactly as they do in normal programming languages like Java or Python:
```
SELECT * FROM "Items"
  WHERE "Price"<2.00
  AND "Calories"<200;
```
Which returns:
```
8,Soda,0.99,180,
```

### `DELETE`<a id="DELETE"/>
What if we added data by accident or some of our data is no longer valid? We can do this using the `DELETE` command. Delete's syntax is very similar to the `SELECT` command. For instance, if we want to delete all data from a table, we can run `DELETE FROM "[TableName] WHERE *";` such as `DELETE FROM "Items" WHERE *;`. We can also use the `WHERE` command to give more detailed instructions on what to delete. For instance, if we want to delete all burgers from our menu, we could run:
```
DELETE FROM "Items"
WHERE "Name" LIKE '%burger';
```
The `LIKE` command returns all items that follow a specific pattern, and `%` means any string of characters, so `WHERE "Name" LIKE '%burger'` returns all items that end in the word "burger". This deletes "Hamburger" and "Cheeseburger" and our "Items" table now contains:
```
2,Fries,1.99,300,
3,Soda,0.99,180,
```

### `ALTER`<a id="ALTER"/>
Sometimes we made a mistake when creating a table or decided we needed to adjust them in some way. This is done in SQL using the `ALTER` command.

#### `ADD`<a id="ADD"/>
What if you want to add a field to a pre-existing table? We can do this using the `ADD` command. For instance, if we want to add a boolean field saying whether an item is vegan or not, we can run:
```
ALTER TABLE "Items"
    ADD "Vegan" BOOLEAN;
```
We can also add constraints using `ADD`. For instance, if we forgot to include a primary key in our table upon creation, we can first add an ID field:
```
ALTER TABLE "Items"
    ADD "ID" INT IDENTITY;
```
And then add the constraint:
```
ALTER TABLE "Items"
DROP PRIMARY KEY;

ALTER TABLE "Items"
ADD CONSTRAINT "PK_Items" PRIMARY KEY ("ID");
```
Notice that we needed to run `DROP` before adding the primary key. This is because LibreOffice Base automatically assigns the primary key constraint to a field upon running `CREATE TABLE`. What exactly does `DROP` do?

#### `DROP`<a id="DROP"/>
`DROP` is the destructive brother to `ADD`. Whereas `ADD` adds functionality to our table, `DROP` removes it. As discussed in the last section, we can remove a constraint using the `DROP` command:
```
ALTER TABLE "Items"
DROP PRIMARY KEY;
```
We can also remove fields:
```
ALTER TABLE "Items"
DROP "Vegan";
```
We can also remove entire tables using the `DROP` command. We do this by running `DROP TABLE "[TableName]"`.

## Walkthrough<a id="WALK"/>
First, follow [our general instructions for running SQL commands in LibreOffice Base](#RUN). Once we've done that, our first step is to create the Users and Classes tables using the `CREATE TABLE` command.

<img src="https://i.imgur.com/dYg5SOf.jpg" alt="The SQL screen of LibreOffice base with a CREATE TABLE command" width="48%" /> <img src="https://i.imgur.com/mavSgCb.jpg" alt="The SQL screen of LibreOffice Base with a CREATE TABLE command" width="48%" align="right" />

Once we've done this, we can confirm these tables were added by using View->Refresh Tables.

<img src="https://i.imgur.com/6XcP70C.jpg" alt="The View->Refresh Tables menu item" width="48%" /> <img src="https://i.imgur.com/Fp1mMCm.jpg" alt="The refreshed UI showing our newly added Users and Classes tables." width="48%" align="right" />

Now that we can see our tables, let's add some data using the `INSERT INTO` command.

<div align="center">
<img src="https://i.imgur.com/IMF5gZp.jpg" alt="The SQL Screen of LibreOffice bse with an INSERT INTO command" width="50%" />
</div>

After we've added this data, let's confirm that this worked by running a `SELECT` statement.

<div align="center">
<img src="https://i.imgur.com/3HFjoTX.jpg" alt="The SQL Screen of LibreOffice bse with a SELECT command" width="50%" />
</div>

## Project<a id="PROJ"/>
Now that we've walked through how to create the Users and Classes tables, it's your turn to create the Professors and Classrooms tables. As a reminder, the Professors table should have three fields (FirstName, LastName, and Ranking) and the Classrooms table should have two (Location and Seats). 
