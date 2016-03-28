This project helps you to configure Nagios in order to monitor your DB2 databases with the following two mechanisms:

* [Monitor DB2 with Nagios](https://github.com/angoca/monitor-db2-with-nagios)
* [db2-jnrpe](https://github.com/angoca/db2-jnrpe)

This project consists of a set of scripts that have the structure to define the necessary objects to monitor DB2.

The objects are defined in a non-traditional way, and for this reason it is important to understand the Nagios structure and DB2 structure.

# Hosts

The following elements are defined as hosts:

* A server where DB2 resides is considered as a host.
* A DB2 instance (db2sysc process) running on a server is also considered as a host.
* A DB2 database inside an instance is also considered a host.

The last two elements could normally not be represented as an object in Nagios, or they can be treated as services;
however, this structure is more flexible and allows keep track of each of these elements, even if they change the location.

For example, when a database is defined as a host, the monitoring can continue its history even if the database changes its instance or server.

# Services

This is the complete set of scripts in the previously defined two project.
Each of these project is defined in a different way here.
Monitor-DB2-with-Nagios is a set of scripts run from command line, thus, they need the notion of server to connect to (via SSH or NRPE).
On the other hand, db2-jnrpe has the notion of database, thus, they need the properties to connect to the database.

# Commands

The commands are configured to run on the server or the database.
For database connections, they connect directly to the database.
Instead, for the commands run from command line, they have to connect to the server and then run the command.
This execution can be cluster-aware, that means when the execution is run from a passive node, all commands return OK.

# Cluster awareness

There are few installation where DB2 run on a active-passive node are OS level without using HADR or similar.
However, they exist, even if this passive node can be considered as a waste of machine because is ON but without doing anything.
Even, this passive machine does not have the file systems where the database resides, neither the virtual IP.
With these last elements, one can identify if a server is active or passive.

With the previous description, there is a script that always returns OK if it is executed from the passive node.
If that node becomes the active node, the script with execute the whole process and returns the corresponding value.
