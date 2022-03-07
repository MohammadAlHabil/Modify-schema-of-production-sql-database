# Modify-schema-of-production-sql-database
# 1- What Is Altering tables?
## SQL ALTER TABLE Statement
The ALTER TABLE statement is used to add, delete, or modify columns in an existing table.


###  ADD Column
To add a column in a table, use the following syntax:
```
ALTER TABLE table_name
ADD column_name datatype;
```
#### Example
The following SQL adds an "Email" column to the "Customers" table:
```
ALTER TABLE Customers
ADD Email varchar(255);

```
### DROP COLUMN
To delete a column in a table, use the following syntax (notice that some database systems don't allow deleting a column):
```
ALTER TABLE table_name
DROP COLUMN column_name;
```
#### Example
The following SQL deletes the "Email" column from the "Customers" table:

```
ALTER TABLE Customers
DROP COLUMN Email;

```
### ALTER/MODIFY COLUMN
To change the data type of a column in a table, use the following syntax:
```
ALTER TABLE table_name
ALTER COLUMN column_name datatype;
```

# 2- What is downtime in this case?
### What is downtime in this case?
![](https://i.imgur.com/Q1N4g7J.png)

### Downtime
Downtime is a term for the time during which a computer or IT system is unavailable, offline or not operational. Downtime has many causes, including shutdowns for maintenance, human errors, software or hardware malfunctions, system crashes, hacker attacks, system reboots, operating system and/or software updates, lack of network connectivity and environmental disasters such as power outages, fires, ...
Downtime duration is the period of time when a system fails to perform its primary function. Communications failures, for example, may cause network downtime.

### Database downtime
There are generally common causes of database failure. Let’s look at each one of them.
1. File Corruption
If one or more files in the database are damaged, they can cause the database to fail at the file level, causing corruption. Files can be corrupted due to several reasons. Primary files, which can corrupt the entire database, may be corrupted due to changes in the SQL Server account, accidental data deletion, and file header corruption, among others.
2. File System Damage
If a server or computer is shut down incorrectly, or if it experiences a power surge, or something happens that interrupts the process while data is being written to the files, the files of the operating system can be damaged or corrupted.
3. Software and Hardware Failure
Hardware failures may include memory errors, disk crashes and others. Hardware failures can also be attributed to design errors and wear out of mechanical parts. On the other hand, software failures may include failures related to software such as operating system, DBMS software, application programs and so on.
4. SQL table alterations
SQL table alterations can interrupt production traffic causing bad customer experience or in worst cases, loss of revenue. DBAs usually encounter these kinds of production interruptions when working with upgrade scripts that touch both application and database or if an inexperienced admin/dev engineer perform the schema change without knowing how SQL operates internally.

# 3- How to Update your Database Schema Without Downtime?
Your application must support update of individual layers, without breaking other layers. That is achieved by making some layers backwards compatible to other layers.

## The Challenge:

We are upgrading application to 2.0, we are going to upgrade our Db schema first. DB schema 2.0 (Data Layer) should be backwards compatible to API Layer 1.0.

That means nothing that API Layer 1.0 is using should change. Also, new additions should not break the API 1.0.

Following are examples of type of changes may break API 1.0
(Dropping, Renaming, Adding, Changes in data types or structure of data).

You do not want to any of the above changes, still you want to make your DB ready for API Layer 2.0.

## The Solution:

Managing Compatible and incompatible DB Schema Updates
Prerequisite for this is that you Split your DB schema update in two parts- Compatible updates and Incompatible updates.

Compatible updates contain changes that do not break API 1.0 (such as adding new Objects, tables or columns needed by API 2.0 but API 1.0 is not even aware of these).

Of course, the Db Schema updates should be well tested before you start doing it on production.

![](https://qph.fs.quoracdn.net/main-qimg-5ef44a7404f0e6cd5e292208d616eff0)

1. Your application 1.0 (blue version) is running. You have tested all the following steps in your dev, QA, staging environments. Now you are ready to go.

2. On a separate instance you make your API layer 2.0 ready. But it does not connect to DB Layer.

3. Now you run your compatible updates on DB. Once your compatible updates are complete, Your DB is still serving API 1.0 but is ready for API 2.0. Your DB layer is now Cyan (compatible with both Blue and Green) At this point live user are using UI 1.0 , API 1.0 and DB 2.0 which is still compatible with API 1.0

4. Connect your API 2.0 to DB and UI 2.0. Your green API and UI layer are ready, having versions 2.0

5. You can now smoke your application 2.0. We want to minimise transition time so test should quite but should conver important areas that are known to break after configuration changes or the areas that changed recently.

Let’s move on to another diagram

![](https://qph.fs.quoracdn.net/main-qimg-ad06d7999c7e0f192c13f420df60bd5b)

6. Assuming your API 2.0 is compatible with UI 1.0 you are ready to move your production URLs from API1.0 and UI1.0 to API 2.0 and UI 2.0 instances. Connections to 1.0 servers start draining. 1.0 servers are still up to complete ongoing requests. New users access full 2.0 stack. Live users are still on UI 1.0 but UI 1.0 is now using API 2.0

7. Connections to API 1.0 are drained. we may still keep server running to allow long running processes to complete. Whatever time the longest process takes.

8. API 1.0 is not in use now. Turn this instance off.

9. At this point you may run Incompatible updates to DB. Objects not needed by 2.0 are dropped. DB is fully Green (2.0 ) now.