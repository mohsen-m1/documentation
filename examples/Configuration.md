# Configuration Examples

This document provides configuration examples.

## Starting a node with a configuration file

When an AnyLog node is initiated, it can be called with a command line parameters. The command line parameters 
are one or more AnyLog commands (with multiple commands, each command is enclosed by quotation mark and seperated by the keyword ***and***).  
Usage:
<pre>
python3 user_cmd.py "command 1" and "command 2" ... and "command n"
</pre>


The command ***process*** followed by a path and a file name will process all the commands in the specified file.  
The following example starts an AnyLog node and configures the node according to the commands listed in a file called ***autoexec.al***.

<pre>
python3 user_cmd.py "process !local_scripts\autoexec.al"
</pre>

## Updating the configuration file

Users can update the configuration file using an editor.

Alternatively, users can update the configuration file from the ***Remore CLI***.

### Configuring a node from the Remote CLI

#### Prerequisite: 
* An AnyLog node running.
* The node is configured with a REST connection (configuring a REST connection is detailed in the [Rest Requests](https://github.com/AnyLog-co/documentation/blob/master/background%20processes.md#rest-requests) section).

#### Updating the config file
* In the Remote CLI, select the config section.  
* In tne config section update the REST IP and Port of the destination node (the REST IP and Port assigned by the node).
* If authentication is enabled, add username and password in the appropriate fields.
* Select one of the following files from the pull-down menu:
  <pre>
  Autoexec
  Operator
  Publisher
  Query
  Master
  Standalone
  </pre>
* Select ***Load*** to retrieve the config file associated with the selected option.
* Note: Autoexec is the config file currently used. The other options are default options for a target role.
* Update the config file as needed.
* To update the changes, select ***Save***.
* Note: ***Changes are saved to the Autoexec file*** regardless the file selected with the ***Load***.

Restart the AnyLog Node - if the node is initiated as in the [example above](#starting-a-node-with-a-configuration-file), the updated ***Autoexec*** file will determine the configuration.

  
## Configuring data removal and archival

### Configuring Backup, Archive and Removal of data

Multiple options are available to backup, archive and remove old data.

#### Setting a standby node

Declare a second operator node associated with an existing cluster. The second node will be dynamically updated with the
data assigned to the cluster.  
This process is detailed in the [High Availability (HA)](../high%20availability.md#high-availability-ha) section.

#### Archival of data

If an Opertaor node is configured with archive option enabled, data that is streaming to the local database is organized in 
files, compressed, and stored in the archival directory by ingestion date.  
The default archival directory is ```AnyLog-Network\data\archive```  
If needed, these files can be copied to an AnyLog ***watch*** directory to be ingested to a new database.
Details are availabel in [Placing data in the WATCH directory](https://github.com/AnyLog-co/documentation/blob/master/adding%20data.md#placing-data-in-the-watch-directory) section.

#### Partitioning of data

A table that is managed by AnyLog can be partitioned by time.  
The ***Partition Command*** id detailed [here](../anylog%20commands.md#partition-command).  
Partitions can be dropped by naming the partitions, or by requesting to drop the oldest partition, or by a request to
keep N number of partition, or to drop old partitions as long as disk space is lower than threshold.  
The ***Drop Partition*** command is detailed [here](https://github.com/AnyLog-co/documentation/blob/master/anylog%20commands.md#drop-partition-command)

These processes can be placed on the AnyLog scheduler to be repeated periodically.  
For example, a table is partitioned by day and the scheduler is executed daily to remove the oldest partition if disk space 
is under a threshold.

Configuring the scheduler is detailed in the [Monitoring Nodes](../monitoring%20nodes.md#monitoring-nodes) section.

#### Backup Partition

Users can leverage the [archival directory](#archival-of-data) for the data backup.  
Alternatively, uses can actively archive a partition using the [backup table](../anylog%20commands.md#backup-command) 
command (and specify the needed partition).


# Example Configuration File

This is an example configuration file with commonly used configuration options and explanations to the selected options.     
Note that the Hash Sign indicates start of a comment and the text that follows is ignored.

#### Disable authentication
If the nodes are trusted, behind a firewall, authentication can be disabled.  
If authentication is enabled, there are different layers that can be leveraged: passwords, signature of messages, and certificates.  
Details are available in the [Users Authentication](../authentication.md#users-authentication) section.

<pre>
set authentication off
</pre>


#### Assign values to variable names
Every node maintains a local dictionary with key value names. Users can associate values to variable names as needed.  
The command ```get dictionary``` shows the variables and their assigned values.  
The command get ```get ![variable name]``` returns the variable value (or, on the CLI the variable name prefixed by exclamation point returns the variable value, i.e. !ip).
<pre>

# Generic Variables

anylog_root_dir=C:\                 # associate the path to the root folder to a variable (NOTE C:\ is the windows version). 
hostname = get hostname             # assign the hostname of the local machine to the key hostname
node_name = anylog-node             # provide a name (preferably unique) to the node. Note, the name would appear on the CLI (i.e.: AL anylog-node > ) 
company_name = "New Company"        # The node owner (company name)

# IP / Port Variables

#external_ip=<external_ip>          # The node may be able to identify the external IP. Otherwise define the external IP. 
#ip=<local_ip>                      # The node may be able to identify the local IP. Otherwise define the local IP. 
anylog_server_port=2148             # The port to use for messages from nodes members of the network.
anylog_rest_port=2149               # The port to use for messages from 3rd parties applications.
master_node = !ip + ":" + !anylog_server_port  # This is declaration for a STANDALONE configuration. Otherwise assign the IP and Port of the master node.
sync_time="30 seconds"              # Synchronize the metadata (from a master node or blockchain) every 30 seconds.

# DBMS Variables

db_user=postgres                    # Use PostgreSQL as a local database (users can use SQLite or both)
db_passwd=postgres
db_ip=!ip
db_port=5432
default_dbms=test



</pre>


#### Declare the root folder for the AnyLog files
AnyLog maintains scripts, configurations and data in different folders.  
The default folders structure is detailed in the a [local directory structure](../getting%20started.md#local-directory-structure) section.
The command below determines the path to the root folder (to the AnyLog-Network folder).  
  
<pre>
set anylog home !anylog_root_dir    # Declare the location of the AnyLog root folder. Note that anylog_root_dir was associated with a value above (anylog_root_dir=C:\).
</pre>

#### Create the AnyLog Directories
The command below will create the AnyLog folders (under the AnyLog root folder) if the folders do not exists.
<pre>
create work directories
</pre>


#### Making the node a member of the AnyLog Network
The nodes is configured to initiate a listener on a dedicated IP and Port to receive messages from peer nodes.  
Details are available in the [TCP Server process](../background%20processes.md#rest-requests) section.  

<pre>

</pre>
  
#### Enabling REST request
3rd party applications communicate with members of the network using REST requests.  
The nodes is configured to initiate a listener on a dedicated IP and Port to receive REST requests from 3rd parties applications.  
Details are available in the [REST requests](../background%20processes.md#rest-requests) section.

<pre>

</pre>


#### Configuring the local database
The local database is used to store the user data and in some cases system data.  
Each line associates a logical database with a physical database.  
Details are available in the [configuring a local database](../sql%20setup.md#configuring-a-local-database) section.
  

<pre>

</pre>

#### Metadata
The nodes are configured to periodically retrieve the metadata (from a blockchain platform or a master node) and host it locally.   
Details are available in the [Blockchain Synchronizer](../background%20processes.md#blockchain-synchronizer) section.

<pre>

</pre>


#### Initiating the scheduler
AnyLog commands can be placed on the scheduler and be executed periodically.  
The command below initiates a scheduler. Additional information is available in the [Alrts and Monitoring](../alerts%20and%20monitoring.md#alerts-and-monitoring) section.

<pre>
run scheduler 1
</pre>



#### Data Partitioning
Data that is hosted in the local database can be partioned by date.     
Details are available in the [Partition Command](../anylog%20commands.md#partition-command) section.

<pre>

</pre>

#### Removal of old data
Using the scheduler, a process is triggered periodically and removes old partitions.
The [Drop Partition Command](../anylog%20commands.md#drop-partition-command) is used to remove old partitions.  
In the example below, the command is placed on the scheduler to be executed daily.
 
<pre>

</pre>

