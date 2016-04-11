This project helps you to configure Nagios in order to monitor your DB2 databases with the following two mechanisms:

* [Monitor DB2 with Nagios](https://github.com/angoca/monitor-db2-with-nagios)
* [db2-jnrpe](https://github.com/angoca/db2-jnrpe)

This project consists of a set of scripts that have the structure to define the necessary objects to monitor DB2.

The objects are defined in a non-traditional way, and for this reason it is important to understand the Nagios structure (hosts, services and commands) and DB2 structure (server, instance, database).

# Hosts

The following elements are defined as hosts:

* A server where DB2 resides is considered as a host.
* A DB2 instance (db2sysc process) running on a server is also considered as a host.
* A DB2 database inside an instance is also considered a host.

You need to modify this file according to your own environment.
The name of the objects are generic, and it is highly advisible to modify them with a Replace All function, in order to prevent errors.
There is an example at the nd of this file that points to a sample database in a db2inst1 instance.

**NOTE**: If you are going to define only stand-alone databases, you need to preserve the `hosts_databases_db2_normal.cfg` file and delete the `hosts_databases_db2_cluster.cfg`.
If you are going to define OS clustered databases, you need to preserve the `hosts_databases_db2_cluster.cfg` file and delete the `hosts_databases_db2_normal.cfg`.
Finally, if you are going to define both types of databases you preserve both files.

Normally, the last two elements could not be represented as an object in Nagios, they are often treated as services;
however, this structure is more flexible and allows keep track of each of these elements, even if they change the location.

For example, when a database is defined as a host, the monitoring can continue its history even if the database changes its instance or server.

# Services

This is the complete set of scripts in the previously defined two projects.
Each of these projects is defined in a different way here.
Monitor-DB2-with-Nagios is a set of scripts run from command line, thus, they need the notion of server to connect to (via SSH or NRPE).
On the other hand, db2-jnrpe has the notion of database, thus, they need the properties to connect to the database.

**NOTE**: If you are going to define only stand-alone databases, you need to preserve the `services_databases_db2_normal.cfg` file and delete the `services_databases_db2_cluster.cfg`.
If you are going to define OS clustered databases, you need to preserve the `services_databases_db2_cluster.cfg` file and delete the `services_databases_db2_normal.cfg`.
Finally, if you are going to define both types of databases you preserve both files.


# Commands

The commands are configured to run on the server or the database.
For database connections, they connect directly to the database.
Instead, for the commands run from command line, they have to connect to the server and then run the command.
This execution can be cluster-aware, that means when the execution is run from a passive node, all commands return OK.

You need to define the following elements for this file:

* The location of the `check_by_ssh` command in the `USER5` macro.
* The location of the SSH private key of the id_rsa/id_dsa certificate in the `USER6` macro.
* The location of the `check_nrpe` command in the `USER7` macro

When using OMD, the previous variables are defined in the resources.cfg file, thus you do not need to modify this file.

# Cluster awareness

There are few installation where DB2 run on a active-passive node are OS level without using HADR or similar.
However, they exist, even if this passive node can be considered as a waste of machine because is ON but without doing anything.
Even, this passive machine does not have the file systems where the database resides, neither the virtual IP.
With these last elements, one can identify if a server is active or passive.

With the previous description, there is a script that always returns OK if it is executed from the passive node.
If that node becomes the active node, the script will execute the whole process and returns the corresponding value.

# Common errors

When plugins say "(Return code of 255 is out of bounds)", make sure the remote host has been added in the known_hosts.
You can force this by executing the following command for each host:

    ssh -o StrictHostKeyChecking=no hostname
