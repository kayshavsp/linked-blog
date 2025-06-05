
## How to setup a new Postgresql DB on EC2 Ubuntu

Chosen the Ubuntu instance. The instance type is t2.small. Created it within a private subnet, and the public IP is disabled. Create a new Key-Pair for the Instance to SSH from the local machine.

After launching an Instance SSH into the instance.

1. Install PostgreSQL

	`sudo apt install postgresql postgresql-contrib`

2. Start PostgreSQL Server

	`sudo systemctl restart postgresql`

3. Start the Postgres command-line tool

	`sudo -u postgres psql`

4. Create a User

	`CREATE USER yourusername WITH PASSWORD 'yourpassword';`

5. Create a DataBase

	`CREATE DATABASE yourdatabase;`

6. Grant all access to your user to the created database.

	`GRANT ALL PRIVILEGES ON DATABASE yourdatabase TO yourusername;`

7. Edit the Postgres server configuration file

	`sudo nano /etc/postgresql/16/main/postgresql.conf

And change listen_addresses = ‘*’. By default, Postgres only listens for localhost which means it only listens for local connections. After changing listen_addresses to “*” then it allows remote connections.

8. Edit the pg_hba (Host-Based Authentication) configuration file.

	`sudo nano /etc/postgresql/16/main/pg_hba.conf

Add the following configuration,

	host <your_database> <your_user> 0.0.0.0/0 md5

0.0.0.0/0 means any IP address this not recommended. You can strict this by mentioning specific IP addresses.<your_machine_ip>/32.

9. After all the changes restart your postgreSQL server.

	`sudo systemctl restart postgresql`

I have Downloaded [dvdrental](https://www.postgresqltutorial.com/postgresql-getting-started/postgresql-sample-database/) database to use that database as my database.

10. Copy data to your server

	`scp -i <name_of_the_pem>.pem <path_to_file>/dvdrental.tar ubuntu@<server_ip>:/home/ubuntu

11. Restore the database

	`pg_restore -U postgres -d <your_db_name> dvdrental.tar

12. Navigate to psql command-line,

	`sudo -u postgres psql`



### Check Database

Copy data to your server

	\\l+

Check database status

	sudo systemctl status postgresql


