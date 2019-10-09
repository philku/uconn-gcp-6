# uconn-gcp-6
UCONN - GCP Lab 6 - Cloud Spanner

This ReadMe contains the supplumentary instructions for the Cloud Spanner Lab. The [Original Instructions can be found here.](https://googleclouduconn.blogspot.com/p/chapter-6.html)

# Instructions
## Interacting with Cloud Spanner

In this Lab, we will enable the Cloud Spanner API, Create a new instance of Cloud Spanner, Create a Database, then enter and query data in the database.

### A) Enable the Cloud Spanner API

Find the Cloud Spanner API by searching "Cloud Spanner API" in the top search bar of Google Cloud Console. Alternatively, you can find "Spanner" in the left main-menu. 

By Default, the Cloud Spanner API is disabled. You will be required to enable the API.

![Cloud Spanner API](https://1.bp.blogspot.com/-AVGtwcdXvUo/XZpO555bM8I/AAAAAAAALkM/JILllX5QxlkjUcJxicg4lAY-Qvr5YxRBQCLcBGAsYHQ/s1600/1.JPG)

![Cloud Spanner Enabled](https://1.bp.blogspot.com/-GFMPS0m7k9g/XZpPNi_5reI/AAAAAAAALkU/Ra7PW0K9uBYlXW4hM9EORRwQcemKdjLygCLcBGAsYHQ/s1600/2.JPG)


### B) Create a New Instance

1) Click on "Create Instance"
2) Name your instance "test instance" or another memorable name.
3) Choose a configuration geographically near you, aand remember the reigon as you'll need the VMs accessing Spanner in the same region as the instance itself.
4) leave the number of nodes set to one.

![Create Spanner Instance](https://1.bp.blogspot.com/-UV5pL5syEIQ/XZpQluDwTnI/AAAAAAAALkg/R3jKI-orlIwCAe3CDPvSAwutXrpJms1KgCLcBGAsYHQ/s1600/3.JPG)

5) Click Create.


### C) Create a database in the instance

1) Enter your instance page by clicking on the instance Name.
 
![New Instance](https://1.bp.blogspot.com/-ZCNKsX16bwA/XZpRGm3ZuII/AAAAAAAALko/heGA69-M7_8WWsHz4jyv6RBQ8r8h8C_KACLcBGAsYHQ/s1600/4.JPG)

2) Start creation of a database by selecting "Create Database"

![New Instance Details](https://1.bp.blogspot.com/-4MsjkUNyzNc/XZpRcCFBE-I/AAAAAAAALk0/eqnOWolQGfoim5FBAMdzHVPAPB2dJWvfwCLcBGAsYHQ/s1600/5.JPG)

3) Choose a name for the database such as "test-database", hit continue.
4) Create some tables for your database by clicking the "edit as text" toggle, and entereing the following DDL (Database Definition Language) entry. This will create two fields: a unique ID (primary key) for the employee, and the employee’s name.

```DDL
CREATE TABLE employees ( 
    employee_id INT64 NOT NULL, 
    name STRING(100) NOT NULL,
    start_date DATE 
) PRIMARY KEY (employee_id);
```

![Create Table](https://dpzbhybb2pdcj.cloudfront.net/geewax2/Figures/06fig10_alt.jpg)

5) Click Create, then you may browse your table details by clicking on the name of your table.

![Table Schema](https://1.bp.blogspot.com/-fhYlG0wgdOY/XZp5Gb1tQII/AAAAAAAALlc/rqUlUO0fHiE_SkShabnrdeb54GQs-sXSgCLcBGAsYHQ/s1600/10.JPG)

You have now created an instance, a database belonging to the instance, and a table belonging to the database.


### D) Add Data to your Database

Note the differences between Spanner and other relational databases. In spanner, you modify data via a separate API, which is more similar to a nonrelational key-value system. Whereas in MySQL, you use an INSERT SQL query to add new data and an UPDATE SQL query to update.

Now we'll prep our environment by installing the client SDK for cloud spanner, through NPM.

1) Enter into your Cloud Shell, and run the following command:
'''
npm install @google-cloud/spanner@0.7.0
'''

2) Create a new file with nano:
```
nano enterData.js
```
This will open up the file in an editor.

3) Copy and Paste the following into the editor:
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
4) Replace "your-project-id", "test-instance", "test-database", and "employees" with your values if they are different. 
5) Save the file by pressing Ctrl+O, and agreeing to any following prompts for filename (press y, then enter).
6) Exit nano by pressing Ctrl+X
7) Run the file to input your data into the database. You should see "Saved data!" printed in the console. If it's not there, or if you see errors, double check your work.
```
node enterData.js
```
Result:
```
Saved data! [ { commitTimestamp: { seconds: '1570637936', nanos: 519760000 } } ]
```
** You may get a Depreciation Warning. It's ok.


### E) Query Data from Your Database

Now that we have data in our datavbase, we'll use Spanner’s Read API to query some data, starting with a single table.

We're going to follow the same process we executed in the last step for creating files and executing them.

Simply:
- Open a new file or edit an existing one with: nano FILENAME.js
- Enter, Edit, and Save your code. Then exit nano.
- Run the file with: node FILENAME.js

1) Start by using the Read API, by calling table.read() in the Node.js

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

<br>


2) Try it again, this time use a special "all" flag on the query. You can copy and paste into a new file, the old file, or you can just modify the new line, changing "keys: ['1']" to "keySet: {all: true}".

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

<br>


3) Generic SQL-querying: you can query a database rather than a specific table, because the query might involve other tables (for instance, if you JOIN two tables together). For this, you can send a string containing your SQL query. Start by sending a simple query to retrieve all of the employees with a SQL query.

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

<br>

4) Now, filter this down to only Bill Gates. To do that, you need to add a WHERE clause in your SQL statement

#### Input
```javascript
const spanner = require('@google-cloud/spanner')({
  projectId: 'your-project-id'
});

const database = spanner.instance('test-instance').database('test-database');
const query = {
  sql: 'SELECT employee_id, name, start_date FROM employees WHERE employee_id = @id',
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

Good Work! Ask you want to know more about Cloud Spanner or anything else.
