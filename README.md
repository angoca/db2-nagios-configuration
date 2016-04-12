This project helps you to configure Nagios in order to monitor your DB2 databases with the following two mechanisms:

* [Monitor DB2 with Nagios](https://github.com/angoca/monitor-db2-with-nagios)
* [db2-jnrpe](https://github.com/angoca/db2-jnrpe)

This project consists of a set of scripts that have the structure to define the necessary objects to monitor DB2.

The objects are defined in a non-traditional way, and for this reason it is important to understand the Nagios structure (hosts, services and commands) and DB2 structure (servers, instances, databases).

It is recommended to create a directory dedicated for these files in your Nagios configuration directory.
This prevents to be modified for other things different from DB2.

# Prerequisites

You need to configure both monitor mechanisms to monitor DB2.

## Monitor DB2 with Nagios

You need to download and extract the scripts in a directory in each server where the scripts are executed.
This directory should be indicated in the Nagios configuration.
Make sure the scripts are executable (`chmod u+x *`).

If you use SSH to run the remote scripts, make sure each server is in the known_hosts file of the .ssh directory.
Port 22 should be opened between Nagios server and database server.

If you use nrpe to run the remove scripts, make sure the remote configuration is according your needs.
Port 5666 should be opened between Nagios server and database server.

More information in the [Wiki of this project.](https://github.com/angoca/monitor-db2-with-nagios/wiki)

## db2-jnrpe

You need to configure jnrpe in order to run the scripts from the Nagios server, or another server designated to use jnrpe.
jNRPE needs java to run.
It is not necessary to run jNRPE in the database servers; these scripts can run remotely via JDBC connection.

If jnrpe is not local to Nagios, the port 5666 should be opened between the Nagios server and the jNRPE server.
If db2-jnrpe is not local to DB2, the instance port should be opened between the jNRPE server and the database server;
in many configurations the port is 50000, but it could change or there could be many instances to monitor.

More information in the [Wiki of this project.](https://github.com/angoca/db2-jnrpe/wiki)

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

## Custom configuration

You can create your own services configuration file to define your own values inheriting from this configuration.
This allows to separate templates and global values from specific configuration.

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
