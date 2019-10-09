# uconn-gcp-6
UCONN - GCP Lab 6 - Cloud Spanner

This ReadMe contains the supplumentary instructions for the Cloud Spanner Lab. The [Original Instructions can be found here.](https://googleclouduconn.blogspot.com/p/chapter-6.html)

# Instructions
## Interacting with Cloud Spanner

In this Lab, we will enable the Cloud Spanner API, Create a new instance of Cloud Spanner, Create a Database, then enter and query data in the database.

### 1) Enable the Cloud Spanner API

Find the Cloud Spanner API by searching "Cloud Spanner API" in the top search bar of Google Cloud Console. Alternatively, you can find "Spanner" in the left main-menu. 

By Default, the Cloud Spanner API is disabled. You will be required to enable the API.

![Cloud Spanner API](https://1.bp.blogspot.com/-AVGtwcdXvUo/XZpO555bM8I/AAAAAAAALkM/JILllX5QxlkjUcJxicg4lAY-Qvr5YxRBQCLcBGAsYHQ/s1600/1.JPG)

![Cloud Spanner Enabled](https://1.bp.blogspot.com/-GFMPS0m7k9g/XZpPNi_5reI/AAAAAAAALkU/Ra7PW0K9uBYlXW4hM9EORRwQcemKdjLygCLcBGAsYHQ/s1600/2.JPG)


### 2) Create a New Instance (Named "test instance")

- Click on "Create Instance"
- Choose a configuration geographically near you, have the VMs accessing Spanner in the same region as the instance itself.
- leave the number of nodes set to one.

![Create Spanner Instance](https://1.bp.blogspot.com/-UV5pL5syEIQ/XZpQluDwTnI/AAAAAAAALkg/R3jKI-orlIwCAe3CDPvSAwutXrpJms1KgCLcBGAsYHQ/s1600/3.JPG)


### 3) Create a database in the instance


 you have to create a new database
 
![New Instance](https://1.bp.blogspot.com/-ZCNKsX16bwA/XZpRGm3ZuII/AAAAAAAALko/heGA69-M7_8WWsHz4jyv6RBQ8r8h8C_KACLcBGAsYHQ/s1600/4.JPG)

![New Instance Details](https://1.bp.blogspot.com/-4MsjkUNyzNc/XZpRcCFBE-I/AAAAAAAALk0/eqnOWolQGfoim5FBAMdzHVPAPB2dJWvfwCLcBGAsYHQ/s1600/5.JPG)

 choose a name for the database

create some tables for your database.
6.4.2. Creating a table
create a simple employee information

two fields  a unique ID (primary key) for the employee, and the employee’s name.

```DDL
CREATE TABLE employees ( 
    employee_id INT64 NOT NULL, 
    name STRING(100) NOT NULL,
    start_date DATE 
) PRIMARY KEY (employee_id);
```


![Create Table](https://dpzbhybb2pdcj.cloudfront.net/geewax2/Figures/06fig10_alt.jpg)




Click Create, a page opens where you can see the details of your table, such as the schema

![Table Schema](https://1.bp.blogspot.com/-fhYlG0wgdOY/XZp5Gb1tQII/AAAAAAAALlc/rqUlUO0fHiE_SkShabnrdeb54GQs-sXSgCLcBGAsYHQ/s1600/10.JPG)

 created an instance, a database belonging to the instance, and a table belonging to the database



### 4) Add Data to your Database

6.4.3. Adding data

differences between Spanner and other relational databases is the way you modify data

MySQL, you use an INSERT SQL query to add new data and an UPDATE SQL query to update

write to Cloud Spanner via a separate API, which is more similar to a nonrelational key-value system

demonstrate, use the @google-cloud/spanner Node.js

npm install @google-cloud/spanner@0.7.0



```javascript
const spanner = require('@google-cloud/spanner')({
  projectId: 'your-project-id'
});

const instance = spanner.instance('test-instance');
const database = instance.database('test-database');
const employees = database.table('employees');

employees.insert([
  {employee_id: 1, name: 'Steve Jobs', start_date: '1976-04-01'},
  {employee_id: 2, name: 'Bill Gates', start_date: '1975-04-04'},
  {employee_id: 3, name: 'Larry Page', start_date: '1998-09-04'}
]).then((data) => {
  console.log('Saved data!', data);
});
```


### 5) Query Data from Your Database

6.4.4. Querying data

use Spanner’s Read API to query a single table.

execute a SQL query on the database

Start by using the Read API, by calling table.read() in the Node.js

#### Input
```javascript
const spanner = require('@google-cloud/spanner')({
  projectId: 'your-project-id'
});
const instance = spanner.instance('test-instance');
const database = instance.database('test-database');
const employees = database.table('employees');
const query = {
  columns: ['employee_id', 'name', 'start_date'],
  keys: ['1']
};

employees.read(query).then((data) => {
  const rows = data[0];
  rows.forEach((row) => {
    console.log('Found row:');
    row.forEach((column) => {
      console.log(' - ' + column.name + ': ' + column.value);
    });
  });
});
```
#### Response
```
Found row:
 - employee_id: 1
 - name: Steve Jobs
 - start_date: Wed Mar 31 1976 19:00:00 GMT-0500 (EST)
```

you can use a special all flag on the query

#### Input
```javascript
const spanner = require('@google-cloud/spanner')({
  projectId: 'your-project-id'
});

const instance = spanner.instance('test-instance');
const database = instance.database('test-database');
const employees = database.table('employees');
const query = {
  columns: ['employee_id', 'name', 'start_date'],
  keySet: {all: true}
};

employees.read(query).then((data) => {
  const rows = data[0];
  rows.forEach((row) => {
    console.log('Found row:');
    row.forEach((column) => {
      console.log(' - ' + column.name + ': ' + column.value);
    });
  });
});
```

#### Response
```
Found row:
 - employee_id: 1
 - name: Steve Jobs
 - start_date: Wed Mar 31 1976 19:00:00 GMT-0500 (EST)
Found row:
 - employee_id: 2
 - name: Bill Gates
 - start_date: Thu Apr 03 1975 20:00:00 GMT-0400 (EDT)
Found row:
 - employee_id: 3
 - name: Larry Page
 - start_date: Thu Sep 03 1998 20:00:00 GMT-0400 (EDT)
```

generic SQL-querying

you query a database rather than a specific table because the query might involve other tables (for instance, if you JOIN two tables together).

you send a string containing your SQL query.

Start by sending a simple query to retrieve all of the employees with a SQL query

#### Input
```javascript
const spanner = require('@google-cloud/spanner')({
  projectId: 'your-project-id'
});

const instance = spanner.instance('test-instance');
const database = instance.database('test-database');
const query = 'SELECT employee_id, name, start_date FROM employees';

database.run(query).then((data) => {
  const rows = data[0];
  rows.forEach((row) => {
    console.log('Found row:');
    row.forEach((column) => {
      console.log(' - ' + column.name + ': ' + column.value);
    });
  });
});
```
#### Response
```
Found row:
 - employee_id: 1
 - name: Steve Jobs
 - start_date: Wed Mar 31 1976 19:00:00 GMT-0500 (EST)
Found row:
 - employee_id: 2
 - name: Bill Gates
 - start_date: Thu Apr 03 1975 20:00:00 GMT-0400 (EDT)
Found row:
 - employee_id: 3
 - name: Larry Page
 - start_date: Thu Sep 03 1998 20:00:00 GMT-0400 (EDT)
```

Now, filter this down to only Bill Gates. To do that, you need to add a WHERE clause in your SQL statement

#### Input
```javascript
const spanner = require('@google-cloud/spanner')({
  projectId: 'your-project-id'
});

const database = spanner.instance('test-instance').database('test-database');
const query = {
  sql: 'SELECT employee_id, name, start_date FROM employees
 WHERE employee_id = @id',
  params: {
    id: 2
  }
};

database.run(query).then((data) => {
  const rows = data[0];
  rows.forEach((row) => {
    console.log('Found row:');
    row.forEach((column) => {
      console.log(' - ' + column.name + ': ' + column.value);
    });
  });
});
```
#### Response
```
Found row:
 - employee_id: 2
 - name: Bill Gates
 - start_date: Thu Apr 03 1975 20:00:00 GMT-0400 (EDT)
 ```
