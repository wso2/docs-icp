# Configure Database

ICP is capable of using various databases to store its data. By default, it uses an embedded H2 database for development and testing purposes. 
However, for production environments, it is recommended to use a more robust database system such as MySQL.

This guide will walk you through the steps to configure ICP to use a MySQL database.

## Prerequisites
Before you begin, ensure you have the following:
1. A running MySQL server.
2. Access credentials (username and password) for a MySQL user with permissions to create databases and tables.

## Step 1: Create a MySQL Database
1. Log in to your MySQL server using the MySQL command-line client or a database management tool.
2. Execute the script in the `<ICP_HOME>/dbscripts/mysql_init.sql` path. This script will create the necessary database and tables required by ICP.
   ```sql
   SOURCE /path/to/ICP_HOME/dbscripts/mysql_init.sql;
   ```

## Step 2: Configure ICP to Use MySQL
1. Open the `deployment.toml` file located in the `<ICP_HOME>/conf/` directory.
2. Add the following configuration to specify MySQL as the database:
   ```toml
   [icp_server.storage]
   
   dbHost ="<MYSQL_HOST>"
   dbPort = "<MYSQL_PORT>"
   dbName = "icp_database"
   dbUser = "<MYSQL_USERNAME>"
   dbPassword = "<MYSQL_PASSWORD>"
   ```
   Replace `<MYSQL_HOST>`, `<MYSQL_PORT>`, `<MYSQL_USERNAME>`, and `<MYSQL_PASSWORD>` with your MySQL server's host, port, username, and password, respectively.
3. Save the `deployment.toml` file.
4. Restart the ICP server to apply the changes.
