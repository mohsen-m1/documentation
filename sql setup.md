# SQL and queries

Nodes in the network maintain data in a local database. The network protocol is able to present the distributed data 
as a single collection of data and users and application can view and query the data without knowing the physical location of the data, 
as if the data is hosted in a single machine.  
In addition, users and application can request to query data from a single particular node or from a list of nodes.

This chapter covers the following topics:
* [Configuring a local database](#configuring-a-local-database) on a node
* Issuing a [SQL command](#sql-commands)  
* The metadata layer
* Query of data using the network protocol

# Configuring a local database

Anylog nodes host data. It is up to the administrator to determine the physical database to use. Examples of supported databases 
are [PosrgreSQL](https://www.postgresql.org/) and [SQLite](https://www.sqlite.org/).   
Users determine which database to use - a node operates indifferently regardless of the physical database selected.  
When a logical database is created, users name the physical database that is assigned to the logical database. The association
will host the database tables in the physical database associated with the logical database.  
Usually, users will leverage SQLite with nodes that are low in compute power and PostgreSQL with stronger nodes.  
Users can leverage multiple physical databases for different logical databases within the same nodes.
In addition, different nodes in the network can use different physical databases for the same logical database.

## Connecting to a local database
The command ***connect dbms*** associate a logical database to a physical database. In addition, the command parameters 
provide the connection information to the physical database. Note that different databases require different connection info.

Usage:
<pre> 
connect dbms [db name] where type = [db type] and user = [db user] and password = [db passwd] and ip = [db ip] and port = [db port] and memory = [true/false] and connection = [db string]
</pre>  

Explanation:

* [db name] - The logical name of the database.
* [db_type] - The physical database - One of the supported databases such as psql, sqlite, pi.
* [db user] - A username recognized by the database.
* [db passwd] - The user dbms password. 
* [db port] - The database port.
* [memory] - a bool value to determine memory resident data (if supported by the database).
* [connection] - Database connection string.

***Note 1***: For SQLite, the logical name of the database can include the path to maintain the data. Otherwise, 
the database data is maintained, for each table of the database, in the default location defined by !dbms_dir.    
***Note 2***: If 'memory' is set to ***true***, the database tables are maintained in RAM (this option is supported by SQLite but not with PostgreSQL).

Examples:
<pre> 
connect dbms test where type = sqlite
connect dbms sensor_data where type = psql and user = anylog and password = demo and ip = 127.0.0.1 and port = 5432
</pre>

## The get databases command

The ***get databases*** command lists the declared databases.  
Usage:
<pre> 
get databases
</pre>  

The command provides the list of logical databases and the physical database supporting each logical database.  
Example output:
<pre>
List of DBMS connections
Logical DBMS         Database Type IP:Port                        Storage
-------------------- ------------- ------------------------------ -------------------------
aiops                psql          127.0.0.1:5432                 Persistent
al_smoke_test        psql          127.0.0.1:5432                 Persistent
almgm                psql          127.0.0.1:5432                 Persistent
blockchain           psql          127.0.0.1:5432                 Persistent
dmci                 sqlite        Local                          D:\Node\AnyLog-Network\data\dbms\dmci.dbms
edgex                psql          127.0.0.1:5432                 Persistent
lsl_demo             psql          127.0.0.1:5432                 Persistent
monitor              sqlite        Local                          D:\Node\AnyLog-Network\data\dbms\monitor.dbms
orics                psql          127.0.0.1:5432                 Persistent
system_query         psql          127.0.0.1:5432                 Persistent
</pre>

## The get database size command

The ***get database size*** command returns the size og the database in bytes.  
Usage:
<pre> 
get database size [logical dbms name]
</pre>  

Example:
<pre> 
get database size lsl_demo
</pre>  


# SQL Commands

SQL commands are issued in 2 ways:
1) To the local database of a particular node. This option is relevant to all [SQL DDL commands and DML commands](https://en.wikipedia.org/wiki/Data_manipulation_language).  
2) Issue queries (DML commands) through the network protocol. This approach allows to treat the data hosted on different nodes
of the network as a single collection of data.
   
## Issuing a SQL command to a node in the network
SQL command are issued against a node in the network.
Queries can be executed against data maintained on the local node and on data maintained by nodes in the network.    
The command ***sql*** directs the node to process a sql command on a local node. The command format is detailed below: 
<pre> 
sql [dbms name] [query options] [sql command or select statement]
</pre>  
* ***[dbms name]*** is the logical DBMS containing the data.
* ***[query option]*** include formatting instructions and output directions ([and are detailed below](#query-options)).
* ***[SQL command]*** a SQL command including a SQL query.

Example issuing a Query on a local node:
<pre> 
sql lsl_demo "drop table lsl_demo"
</pre>  

Example issuing a SQL command on a local node:
<pre> 
sql dmci "select count(*) from cos_data"
</pre>  

## Issuing a query to the network

This option allows to treat the multiple nodes as a single machine that host a unified collection of data.  
This option is enabled by adding ***run client ()*** to the command prefix. The added prefix means that the node issuing
the command only serves as a client to the network nodes that host the relevant data.     
If the parenthesis are left empty - the network protocol identifies the destination nodes. Alternatively, users can specify one
or more destination nodes (by listing their IPs and Ports), in this case the listed nodes would be treated as the nodes that host the data.
 
Usage:
<pre> 
run client () sql [dbms name] [query options] [select statement]
</pre>  

Note 1: The SQL queries that are supported by the network protocol are detailed below.
Note 2: The network protocol also supports pre-defined functions.

Details on how to query multiple data from multiple nodes are available in the section [Query nodes in the network](https://github.com/AnyLog-co/documentation/blob/master/queries.md#executing-queries-against-the-nodes-in-the-network).

## The metadata
The data in the network is treated as if it is maintained in a relational database and similarly to a centralized database, 
users and applications can query the metadata to determine the databases, tables and columns for each table.

### The get tables command

The ***get tables*** command lists the tables maintained by the named database.    
Usage:
<pre> 
get tables where dbms = [dbms name] and format = [format type]
</pre>  

Details:  
* [dbms name] - The logical name of the database maintaining the tables.
* [format type] - An optional parameter to specify the format of the reply info. The format options are ***table*** (default) and ***json***.

The output presents every table assigned to the named database and indicates if the table is defined on the local node
(in the physical database) and if defined on the global metadata (i.e. blockchain) platform.

If database name is asterisk  (*) - all tables declared on the node and on the global metadata are listed.

Examples:
<pre> 
get tables where dbms = dmci
get tables where dbms = *
get tables where dbms = aiops and format = json
</pre>  

### The get table command (get table status)

The ***get table*** command provides status info on the named table.  
Usage:
<pre> 
get table [info type] where name = [table name] and dbms = [dbms name]
</pre>  

Details:

| Info Type     | Explanation  |
| ------------- | --------------- |
| exist status  | Returns True/False values indicating if the table is declared on a local database and on the global metadata layer | 
| local status  | Returns True/False value indicating if the table is declared on a local database |
| Blockchain status  | Returns True/False value indicating if the table is declared on the global metadata layer |
| rows count  | Returns the number of rows in the table |
| complete status  | Returns all the available table info |

Examples: 
<pre>
get table local status where dbms = aiops and name = lic1_s
get table partitions names where dbms = aiops and name = lic1_sp
get table complete status where name = ping_sensor and dbms = anylog
</pre>

### The get columns command 

The ***get columns*** command provides the list of columns names and data types for the named table.  
Usage:
<pre> 
get columns where dbms = [dbms name] and table = [table name] and format = [output format]
</pre>  
The format determines the output format. The format options are ***table*** (the default value) and ***json***. 
Examples: 
<pre>
get columns where dbms = aiops and table = ping_sensor
get columns where dbms = aiops and table = ping_sensor and format = json
</pre>


### The get rows count command 
The ***get rows count*** command provides the number of rows in every table on the connected node.  
Note: to determine the number of rows for a particular table in all nodes, issue a ***select count*** command.  
Usage:
<pre> 
get rows count where dbms = [dbms name] and table = [table name] and format = [format type] and group = [group type]
</pre>  

Details:
* [dbms name] - the name of the database that hosts the tablle of interest.
* [table name] - the name of the table of interest.
* [format type] -An optional parameter to specify the format of the reply info. The format options are ***table*** (default) and ***json***.
* [group type] -An optional parameter to specify if rows are returned per partition or aggregated as a single value for each table.
The group options are ***partition*** (default) ***table***.
  
Examples: 
<pre>
get rows count
get rows count where dbms = my_dbms and group = table
get rows count where dbms = my_dbms
get rows count where dbms = my_dbms and table = my_table
</pre>

### The get partitions command 
The ***get partitions*** command details the partition definition for each partitioned table.  
Usage:
<pre> 
get partitions
</pre>  