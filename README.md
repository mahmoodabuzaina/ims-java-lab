# ims-java-lab

This lab's goal is to show a Java developer how to access an IMS database through the IMS Universal JDBC driver.



## Getting started
This lab is designed specifically for coding Java in an Eclipse IDE. The laptops are already pre-configured for you with eclipse and the lab project pre-installed. If you get lost at any step, or have any questions, raise your hand to get help from the instructor or the assistants.

### Pre-requisites
Software:
* [Eclipse Neon or later](https://www.eclipse.org)
* [Java 7 or later (Java 8 recommended)](https://java.com/en/)

Skills:
* Java programming - Beginner level
* SQL programming - Beginner level
* IMS - Beginner level

## Writing a distributed Java application
In this lab we will be developing a distributed Java application. In this case when we say distributed, we're specifically talking about any non z/OS environment that supports Java.

Connections to IMS resources on the mainframe from a distributed environment requires a TCP/IP connection through an IMS Connect TCP/IP gateway. For our distributed application, we will be connecting through a pre-configured IMS Connect that resides on a public demo system.

We will cover two different query languages in this lab. Exercises 1 through 7 will cover the use of IMS JDBC driver to issue SQL queries and Exercises 8 through 10 will cover using the IMS Java DL/I API to issue DL/I queries.

### Opening up the lab Java file
1. Open up Eclipse.
2. You should have a **ims-java-lab** project in the **Package Explorer**.
3. Expand out the following folders: **ims-java-lab->src->com.ibm.ims.lab**
4. Double click on the `MyIMSJavaApplication.java` file. The majority of your work will be done in this file.

### Exercise 1: Creating a Type-4 JDBC connection to an IMS database
Overview:
The following information is required to connect to an IMS database from an external environment
1. Hostname/IP address of the IMS Connect
2. Port number for the IMS Connect
3. Username to authenticate against the system's resource access control facility (RACF)
4. Password to authenticate against RACF
5. The IMS Program specification block (PSB) name that the user will access

This information would be set in an **IMSDataSource** object. The following shows how to create the object:

```java
IMSDataSource ds = new IMSDataSource();
```

Now let's start:
In `MyIMSJavaApplication.java`, go ahead and uncomment the line under Exercise 1 in the `main()` method by removing the comments ('//').

```java
createAnImsConnection(4).close();
```

Now navigate to the `createAnImsConnection()` method and create your connection underneath the Exercise 1 section.

To create your connection first create an IMSDataSource object. We will use an IMSDataSource to create our connection but you could also alternatively create it using the standard JDBC [DriverManager](https://www.ibm.com/support/knowledgecenter/en/SSEPH2_15.1.0/com.ibm.ims15.doc.apg/ims_odbjdbcconndrivermgr.htm) interface.

The following line should already be in your java file:
```java
IMSDataSource ds = new IMSDataSource();
```
In `MyIMSJavaApplication.java`, go ahead and uncomment the following lines of code:
```java
ds.setHost("insert IP address");
ds.setPortNumber(insert port number); 
ds.setDriverType(insert driver type number); 
ds.setDatabaseName("insert database name"); 
```
You will notice some errors, that's because the values are not yet set. Now go ahead and use the following information to insert the correct values to set up your IMSDataSource object to establish a connection:
1. **host**: IP address to be provided by the lab instructor
2. **port number**: 2500
3. **driver type**: 4
4. **database name**: PHIDPHO1 *<-- This is actually the PSB name* (**Notice that O is not a zero**)


Once you've set all of the connection information, you can create a connection by calling the `getConnection()` method on your IMSDataSource object.

Uncomment the following line by removing ('//'):
```java
connection = ds.getConnection();
```

You can now run your application by selecting **Run->Run As->Java Application** from the top Eclipse menu. The console should show the following output:

```
Apr 16, 2018 1:52:18 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Server IMS Connect DDM level:  1
Apr 16, 2018 1:52:18 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Client IMS Connect DDM level:  1
Apr 16, 2018 1:52:18 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Server ODBM DDM level:  1 2 3 4 5 6
Apr 16, 2018 1:52:18 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Client ODBM DDM level:  1 2 3 4 5 6 7
Apr 16, 2018 1:52:18 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: ODBM DDM level is backlevel with respect to the Universal driver client. Some functionality in the driver will be disabled. Suggest upgrading ODBM to latest service level.
Apr 16, 2018 1:52:18 PM com.ibm.ims.dli.PSBInternalFactory createPSB
INFO: IMS Universal Drivers build number: 14066
```

The output shows that we created a connection to the system and validated functional levels between the server and the client.

Let's disable Excercise 1 before moving on by adding the comments back in the `main()` method

```java
//createAnImsConnection(4).close();
```

### Exercise 2: Discovering the database metadata
Overview:
Now that we have a connection to IMS, the next step is to discover what databases are available for access through the PSB (PHIDPHO1) defined in the connection from Exercise 1. This database metadata information would be stored in the IMS catalog which is IMS' trusted source for information. This information has been mapped to standard JDBC **DatabaseMetadata** discovery which many JDBC based tools use.

The following is a mapping of terms from IMS to the relational model that the JDBC interface uses:
* Program Control Block (PCB) == Schemas
* Database Segments == Database Tables
* Database Fields == Database Columns
* Database Records == Database Rows

Let's get started:
Similar to Exercise 1, we will want to uncomment the following line in the `main()` method:
```java
displayMetadata();
```

Now navigate to the `displayMetadata()` implementation, you'll notice that we are first establishing a connection to the IMS system by taking advantage of the code we wrote in Exercise 1.

Let's take that connection object and retrieve a queryable DatabaseMetaData object from that. Copy and paste the following line of code under the `// Exercise 2 ` comment:

```java
DatabaseMetaData dbmd = connection.getMetaData();
```

The `DatabaseMetaData` class contains [several methods](https://docs.oracle.com/javase/7/docs/api/java/sql/DatabaseMetaData.html) for discovery which typically returns back a ResultSet object. Let's discover what PCBs are available by using the `getSchemas()` method. Remember that PCBs have a one to one mapping with schemas. The following code will show how to invoke the `getSchemas()` method and display the output. Copy and paste the following code block after `DatabaseMetaData dbmd = connection.getMetaData();`:

```java
// Get IMS PCB information
ResultSet rs = dbmd.getSchemas("PHIDPHO1", null);
ResultSetMetaData rsmd = rs.getMetaData();
int colCount = rsmd.getColumnCount();

System.out.println("Displaying IMS PCB metadata");
while (rs.next()) {
  for (int i = 1; i <= colCount; i++) {
    System.out.println(rsmd.getColumnName(i) + ": " + rs.getString(i));
  }
}
```
Now run your application.

In addition to using DatabaseMetaData for discovery of the database, we also used ResultSetMetaData above to identify information on the ResultSet returned by the getSchemas() call. We will be using ResultSetMetaData in most of the following exercises in order to display a readable output like the following:

```
TABLE_SCHEM: PCB01
TABLE_CATALOG: PHIDPHO1
PCB_PROCESSING_OPTIONS: AP
DBD_NAME: DHIDPHO1
DBD_TIMESTAMP: 1810711232054
```

We can dig even further into the database segments and fields with the following queries. Replace the following query:
```java
ResultSet rs = dbmd.getSchemas("PHIDPHO1", null);
```
with the following query to get the segment information:
```java
// Get IMS segment information
ResultSet rs = dbmd.getTables("PHIDPHO1", "PCB01", null, null);
```
or with the following query to get the field information:
```java
// Get IMS field information
ResultSet rs = dbmd.getColumns("PHIDPHO1", "PCB01", "A1111111", null);
```

That completes Exercise 2. Let's go ahead and disable the following line in the `main()` method by commenting it out:
```java
//displayMetadata();
```

### Exercise 3: Querying the database
Now that we have a good understanding of what our database looks like. We can go ahead and start building queries against the database. Let's start by uncommenting the following line in the `main()` method.

```java
executeAndDisplaySqlQuery();
```

Let's now navigate to the `executeAndDisplaySqlQuery()` method and write our SQL SELECT statement to issue a read request against the database.

An initial query has already been written `SELECT * FROM PCB01.A1111111`. This is based off of our database metadata discovery where we know the PSB PHIDPHO1 contains a PCB PCB01 which has a segment A111111 that contains fields related to a phonebook.

The way we would execute a read query is through the `Statement.executeQuery()` method. We can get a `Statement` object off of the `Connection`. The following code shows how to do that. Copy and paste the following block of code after `String sql = "SELECT * FROM PCB01.A1111111";` :

```java
Statement st = connection.createStatement();
ResultSet rs = st.executeQuery(sql);
ResultSetMetaData rsmd = rs.getMetaData();
int colCount = rsmd.getColumnCount();

System.out.println("Displaying IMS PCB metadata");
while (rs.next()) {
  for (int i = 1; i <= colCount; i++) {
    System.out.println(rsmd.getColumnName(i) + ": " + rs.getString(i));
  }
}
```

You should see output similar to the following:
```
LASTNAME: LAST1
FIRSTNAME: FIRST1
EXTENTION: 8-111-1111
ZIPCODE: D01/R01
```
That completes Exercise 3. Let's go ahead and disable the following line in the `main()` method by commenting it out:
```java
//executeAndDisplaySqlQuery();
```

### Exercise 4: Looking at how IMS breaks down SQL queries
The native query language for an IMS database is DL/I. In order for IMS to process SQL queries, those queries will need to be translated into the DL/I equivalent. Sometimes, it's useful for debugging or tuning purposes to look at how a SQL query is broken down.

So where is this translation being done? In this case, the IMS JDBC driver handles all of the translation. It exposes the translation through the `Connection.nativeSql()` method.

Let's start by uncommenting the following line in the `main()` method.
```java
displayDliTranslationForSqlQuery();
```

Let's take a look at the the translation for the previous SQL query by adding the following code snippet to the `displayDliTranslationForSqlQuery()` method:
```java
System.out.println("DL/I translation for '" + sql + "' is:");
System.out.println(connection.nativeSQL(sql));
```

You should see the following output in your console:
```
DL/I translation for 'SELECT * FROM PCB01.A1111111' is:
GU   A1111111

(LOOP)
GN   A1111111

NOTE: GU/GN VALID only if not overruled by CONCUR_UPDATABLE ResultSet concurrency
```

The SQL SELECT query which can be considered a batch retrieve, is translated into a series of singleton DL/I calls. The first call is to a **GET UNIQUE** which retrieves the first record to match a qualifier. The IMS JDBC driver will then repeatedly call **GET NEXT** until it retrieves all records from the database that match the qualifier.

Feel free to come back to this exercise in order to look at the translation for the SQL statements in Exercise 5 and 6. For now, we'll go ahead and comment the following line in the `main()` method.

```java
//displayDliTranslationForSqlQuery();
```


### Exercise 5: Inserting a record into the database
Now that we have a base understanding of what the IMS database looks like and what data resides in that database, we'll go ahead and insert in a new phonebook record. Let's start off by uncommenting the following lines in the `main()` method
```java
executeASqlInsertOrUpdate();
executeAndDisplaySqlQuery();
```

Then navigate to the `executeASqlInsertOrUpdate()` method.

The database segment that we have been looking at is keyed off of the LASTNAME parameter, this means we will need a unique last name value from what's already in the database (which you should know from Exercise 3), **That value will be provided for you**.

The format for a SQL INSERT statement can be found [here](https://www.w3schools.com/sql/sql_insert.asp). Similar to what we did for a SQL SELECT, we will be using a `Statement` object to issue the SQL statement. However instead of using the `executeQuery()` method which is for database reads, we will want to use the `executeUpdate()` method for database inserts, updates and deletes.

The following code snippet will insert a record into the database. Make sure to modify the values for the entry you want to add. Also make sure your entry does not exceed 10 characters.
```java
sql = "INSERT INTO PCB01.A1111111 (LASTNAME, FIRSTNAME, EXTENSION, ZIPCODE) VALUES ('insert your last name', 'insert your first name', '123456A', '12345')";
Statement st = connection.createStatement();
System.out.println("Inserted " + st.executeUpdate(sql) + " record");
```

Run the Java application and verify that your new record was inserted properly. You should see something like the following in your output:
```
Inserted 1 record

Displaying query results
LASTNAME: *Your last name*
FIRSTNAME: *Your first name*
EXTENTION: 123456A
ZIPCODE: 12345
```

What happens if you try to insert the same record again? We would expect an error as we can't have two records with the same unique key. Try running your application again. You should see the following error message
```
com.ibm.ims.drda.base.DrdaException: An error occurred processing the database DHIDPHO1. AIB return code: 0x900. AIB reason code: 0x0. AIB error code extension: 0x0. DBPCB status code: II.
 ```

You'll notice that the we get some AIB return and reason code in addition to a DBPCB status code. This error information is actually returned by the IMS database as a result of attempting to execute the translated DL/I query. Looking at the [IMS knowledge center](https://www.ibm.com/support/knowledgecenter/en/SSEPH2_15.1.0/com.ibm.ims15.doc.msgs/msgs/ii.htm), we can see that the II status code is returned on a DL/I ISRT call when a record already exists in the database.

Before moving on to the next exercise, let's make sure we comment out any code we added **within** the `executeASqlInsertOrUpdate()` method.


### Exercise 6: Updating a record in the database
Let's take the record we inserted in the previous exercise and update it using a SQL UPDATE statement. The format for a SQL UPDATE can be found [here](https://www.w3schools.com/sql/sql_update.asp).

We want to make sure only update the record we inserted earlier. This can be done by qualifying on the LASTNAME field which we know is a unique field. The following code snippet shows how to issue a SQL UPDATE query, make sure to modify the fields and qualifier as necessary.
Inside `executeASqlInsertOrUpdate()` add the following (don't forget to insert your last name):
```java
sql = "UPDATE PCB01.A1111111 SET FIRSTNAME='BILBO' WHERE LASTNAME='insert your last name'";
Statement st = connection.createStatement();
System.out.println("Updated " + st.executeUpdate(sql) + " record(s)");
```
Ensure the last name you have entered in the SQL query is the same last name specified in exercise 5.
After running your application, you should see similar output in your console (you may have to scroll up to see the result):
```
Updated 1 record(s)

Displaying query results
LASTNAME: *Your last name*
FIRSTNAME: BILBO
EXTENTION: 123456A
ZIPCODE: 12345
```

This concludes the distributed portion of the lab. Make sure to clean up your application by going back into the `main()` method and commenting out the following lines:
```java
//executeASqlInsertOrUpdate();
//executeAndDisplaySqlQuery();
```


### Exercise 7: Creating a distributed DL/I connection
We're now going to shift over to the DL/I programming model. This is the native query language used by IMS. Typically, the only times people would use this is if they're issuing a DL/I call that doesn't have a SQL equivalent (such as Get Next In Parent (GNP) call) or when they're converting an existing application where the DL/I queries are already defined from one language (e.g., COBOL, PL/I) to Java .

Before we start coding let's quickly talk about PSBs. A PSB is the entry point into any IMS resource. For the IMS Java DL/I API, we will essentially be using the PSB to represent a physical connection to an IMS database. Most actions that you use to do on a JDBC `Connection` object, you could do the same to the `PSB` object. We'll see other similarities as well between the two interfaces but the main ones to keep in mind are:
* **Connection** == **PSB**
* **Statement** == **PCB**
* **ResultSet** == **PathSet**

Let's again start by uncommenting the following line in the `main()` method:
```java
createAnImsDliConnection(4).close();
```

Now navigate to the `createAnImsDliConnection()` method where we will implement the distributed (Type-4) connection.

We will again reuse the same connection information from our JDBC connection for this. This is because both APIs take advantage of the same back end systems including the IMS Connect TCP/IP gateway.

You'll first want to instantiate an `IMSConnectionSpec` object to hold all of the connection properties. The following code snippet shows how to do that:
```java
IMSConnectionSpec imsConnSpec = IMSConnectionSpecFactory.createIMSConnectionSpec();
```
In your java file, uncomment the following lines by removing ('//') and set the appropriate values in the setters:
```java
imsConnSpec.setDatastoreServer("insert IP address");
imsConnSpec.setPortNumber(insert port number);
imsConnSpec.setDatabaseName("insert database name");
imsConnSpec.setDriverType(insert driver type number);
```
Use the following information to insert the correct values to establish a connection:
1. **host**: IP address to be provided by the lab instructor
2. **port number**: 2500
3. **driver type**: 4
4. **database name**: PHIDPHO1 *<-- This is actually the PSB name* (**Notice that O is not a zero**)

Notice that the main difference here from what we did with the `IMSDataSource` object in Exercise 1, is we use `IMSConnectionSpec.setDatastoreServer()` instead of `IMSDataSource.setHost()` but they both point to the same hostname or ip address for your IMS Connect.

You can validate you are getting a valid connection when you run this if you see the same output from Exercise 1:
```
Apr 19, 2018 7:23:50 PM com.ibm.ims.dli.PSBInternalFactory createPSB
INFO: IMS Universal Drivers build number: 14066
Apr 19, 2018 7:23:51 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Server IMS Connect DDM level:  1
Apr 19, 2018 7:23:51 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Client IMS Connect DDM level:  1
Apr 19, 2018 7:23:51 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Server ODBM DDM level:  1 2 3 4 5 6 7
Apr 19, 2018 7:23:51 PM com.ibm.ims.drda.t4.T4ConnectionReply checkServerCompatibility
INFO: Client ODBM DDM level:  1 2 3 4 5 6 7
```

Now that our connection code is good, go back and comment out the following line in your `main()` method:
```java
//createAnImsDliConnection(4).close();
```

### Exercise 8: Read all records from the database using GU/GN calls
Now that we have a connection lets go ahead and retrieve all records from the database. This would be the equivalent of a `SELECT * FROM PCB01.A1111111` SQL query. In Exercise 4, we saw that such a query translated to the following in DL/I:
```
GU   A1111111

(LOOP)
GN   A1111111
```

So we know we will be issuing a Get Unique (GU) call and then looping through a bunch of Get Next (GN) calls with a SSA List of A1111111. Let's go ahead and uncomment the following line in the `main()` method.
```java
readAllRecordsWithDliGuGnCalls();
```

Inside the `readAllRecordsWithDliGuGnCalls()` method, we will take advantage of the `createAnImsDliConnection()` you wrote previously. From that we will get a `PSB` object which we will then used to retrieve a `PCB` and from that we will get a `SSAList` and finally a `Path` object. The following code snippet shows how to create each of those objects.
```java
PCB pcb = psb.getPCB("PCB01");
SSAList ssaList = pcb.getSSAList("A1111111");
Path path = ssaList.getPathForRetrieveReplace();
```

So the SSA list here is used for our qualification statement. Because we're retrieving all records, we only need to specify the segment name *A1111111* within the SSAList.

You can think of the Path object as a byte buffer that we will use to store record data. In this case we are preallocating the space to store the record once we retrieve it.

Now that we have all the necessary components, let's issue our GU call. Remember how earlier we said a `PCB` in the DL/I API works similarly to a `Statement` in JDBC. So instead of doing a `Statement.executeQuery()`, we will issue a `PCB.getUnique()` as shown below:
```java
pcb.getUnique(path, ssaList, false);
```

Now wait what is that third parameter used for? It's actually to denote whether we are doing a hold call which will preserve positioning within the IMS database. This would cause a Get Hold Unique (GHU) call to be flowed instead of a GU. In this case since we're just doing read operations, we will just set it to false.

The `PCB.getUnique()` method should populate our `Path` object that we passed in. Go ahead and display the data that was retrieved. The following code shows how to do that for the **FIRSTNAME** and **LASTNAME** fields (do not replace FIRSTNAME AND LASTNAME fields).
```java
System.out.println("FIRSTNAME: " + path.getString("FIRSTNAME"));
System.out.println("LASTNAME: " + path.getString("LASTNAME"));
```

We should of gotten console output similar to the following:
```
FIRSTNAME: BILBO
LASTNAME: *Your last name*
```

Now let's retrieve the remaining records using the `PCB.getNext()` method. Remember that we'll want to loop through this. The `PCB.getNext()` method will return a boolean value indicating whether another record exists in the database. Lets do a simple `while` loop to process and display data from the remaining records.
```java
while(pcb.getNext(path, ssaList, false)) {
	System.out.println("FIRSTNAME: " + path.getString("FIRSTNAME"));
}
```

Run your application and validate your output looks like what you would expect. Once you're satisfied, go back to the `main()` method and comment out the following line:
```java
//readAllRecordsWithDliGuGnCalls();
```

### Exercise 9: Reading a specific record with a DL/I GU call
This exercise will be very similar to the previous one except we will be retrieving just one specific record. To be precise, we will be retrieving the record that was inserted in Exercise 5. Let's start by uncommenting the following line in the `main()` method and then navigating to the `readASpecificRecordWithDliGu()` method
```java
readASpecificRecordWithDliGu();
```

Essentially we want to issue a query similar to the following:
`SELECT * FROM PCB01.A1111111 WHERE LASTNAME='insert your last name'`

Feel free to run your specific query through the translation but essentially the equivalent query will look like the following:
```
GU   A1111111(A1111111EQ*YOUR LAST NAME*   )
```

So basically the code should be the same as Exercise 8 except for where we build the qualification statement. To get the following **A1111111(A1111111EQ*YOUR LAST NAME*   )**, you would use the following code sample, don't forget to insert your last name:
```java
PCB pcb = psb.getPCB("PCB01");
SSAList ssaList = pcb.getSSAList("A1111111");
Path path = ssaList.getPathForRetrieveReplace();

ssaList.addInitialQualification("A1111111", "LASTNAME", SSAList.EQUALS, "insert your last name");

pcb.getUnique(path, ssaList, false);

System.out.println("FIRSTNAME: " + path.getString("FIRSTNAME"));
System.out.println("LASTNAME: " + path.getString("LASTNAME"));

while(pcb.getNext(path, ssaList, false)) {
	System.out.println("FIRSTNAME: " + path.getString("FIRSTNAME"));
}
```

Once you're done coding and validating your output, go back to the `main()` method and comment out the following line:
```java
//readASpecificRecordWithDliGu();
```

### Exercise 10: Updating a specific record with a DL/I GU and REPL call
In this exercise we will update the record that we just retrieved. Uncomment the following lines in the `main()` method and then navigate to the `updateASpecificRecordWithDliGhuRepl()` method.
```java
updateASpecificRecordWithDliGhuRepl();
readASpecificRecordWithDliGu();
```

Here we will be writing the equivalent of the following SQL statement:
`UPDATE PCB01.A1111111 SET FIRSTNAME='FRODO' WHERE LASTNAME='insert your last name'`

Feel free to run your specific SQL through the translator but in general the DL/I equivalent should look like the following:
```
GHU  A1111111(A1111111EQ*YOUR LAST NAME*   )
REPL
```

The first call is very similar to the GU statement we issued in Exercise 9. The main difference is now we're doing a GHU instead of GU which if you remember is toggled through the boolean parameter of the `PCB.getUnique()` method. This will save our position in the database so we can perform our update.

Here's the code for the GHU call, don't forget to insert your last name:
```java
PCB pcb = psb.getPCB("PCB01");
SSAList ssaList = pcb.getSSAList("A1111111");
ssaList.addInitialQualification("A1111111", "LASTNAME", SSAList.EQUALS, "insert your last name");
Path path = ssaList.getPathForRetrieveReplace();
pcb.getUnique(path, ssaList, true);
```

Now the Path object here contains the current row data. We'll want to update the values in the path object and then push it back to the database with the `PCB.replace()` method. In the following example we're updating the **FIRSTNAME**.
```java
path.setString("FIRSTNAME", "FRODO");
pcb.replace(path);
```

Run the application and validate that your record was indeed updated. Once you're satisfied go back to the `main()` method and comment the following lines:
```java
//updateASpecificRecordWithDliGhuRepl();
//readASpecificRecordWithDliGu();
```


## Writing a native Java application
The IMS Transaction Manager supports writing native IMS applications in Java within an IMS Java dependent region. IMS supports several different types of [dependent regions](https://www.ibm.com/support/knowledgecenter/en/SSEPH2_15.1.0/com.ibm.ims15.doc.sag/system_intro/ims_depend-regions.htm):
* Batch Message Processing (BMP)
* Message Processing Program (MPP)
* IMS Fast Path (IFP)
* Java Message Processing (JMP)
* Java Batch Processing (JBP)

Typically when referring to the IMS Java dependent regions, we will be talking about both JMP and JBP regions. Also the JMP and JBP regions are the respective Java equivalents for MPP and BMP regions.

IMS actually supports Java in every dependent region for [language interoperability](https://developer.ibm.com/zsystems/documentation/java/ims/). The region that gets chosen will depend on what language is called first. If your application entry point is written in COBOL which then calls Java, you should be using a non-Java dependent region. If your application entry point is Java, then a Java dependent region should be used. This lab will not cover language interoperability.

### Exercise 11: Writing a JBP application

We will be writing a simple JBP application here that will access the same database we just looked at. Let's start by uncommenting the following line in the `main()` method.
```java
executeNativeApplication();
```

Within the `executeNativeApplication()` method, you'll see that the first thing we do is create a Type-2 JDBC connection. This code hasn't been implemented yet so let's go back to the `createAnImsConnection()` method and add the code for that within the following else block:
```java
} else if (driverType == 2) {
  // A Type-2 JDBC connection is used for local access on the mainframe
  // Exercise 7: Retrieve a Type-2 JDBC connection and set it to the connection object

}
```

The implementation is remarkably similar to what we did for our Type-4 connection. The only differences is that we no longer need to a host or port as we're not connecting through TCP/IP and we specify the Driver Type to be 2.

After you have finished implementing the code to create a Type-2 connection, let's go back to the `executeNativeApplication()` method. You'll see that in addition to the `Connection` object, we will be working with an `Application` and a `Transaction` object. These will be used to define our unit of work. You can think of the unit of work as the scope for what actions will be committed or rollbacked.

Let's add some work within the unit of work by processing a simple SQL SELECT query to retrieve the record you had inserted and updated earlier in Exercises 5 and 6. Make sure to display it as well with `System.out.println()`. The main difference with what we did earlier is this output will not be displayed within the Eclipse console but instead to the job log for our native application.

While the unit of work for this application is very small, you can have much longer running batch jobs. For long jobs you'll want to issue checkpoints in case an error occurs and you don't want to start from the beginning. This can be done by using the `Transaction.checkpoint()` and `Transaction.restart()` methods. We won't be covering this in the lab but a code sample can be found [here](https://www.ibm.com/support/knowledgecenter/en/SSEPH2_15.1.0/com.ibm.ims15.doc.apg/ims_developingjbpapps.htm).

#### Exporting your native application
Now because this is meant to be a native application, it will need to be deployed to the mainframe. In order to deploy this let's go ahead and export the project as a Java archive (JAR) file.

1. Right click on the **ims-java-lab** project in the **Project Explorer** view on the left.
2. Select **Export...** from the context pop-up menu.
3. Choose the **JAR File** option and click **Next**
4. In the **Select the export destination:** input box, choose an easy to navigate to directory and provide the following file name for the jar file, *ImsJavaLab.jar*

Now you have a JAR file containing your native IMS transaction that's ready for deployment

#### Deploying your native application
Deploying a native application typically requires an IMS System Programmer to setup your IMS dependent region. Some of this work can be automated using some DevOps solution such as Jenkins like the one described [here](https://github.com/imsdev/ims-devops-java-jenkins).

This lab is currently designed such that you do not have System Programmer privileges. Please notify the lab instructor you are ready to deploy your native application and they will deploy and run the JBP application for you and display your custom output in the job log.
