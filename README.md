# Setup Postgresql Ubuntu

# 1. Installation
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql.service

# 2.Using PostgreSQL Roles and Databases
Another way to connect to the Postgres prompt is to run the psql command as the postgres account directly with sudo:
sudo -u postgres psql

# 3.Creating a New Role
If, instead, you prefer to use sudo for each command without switching from your normal account, run:
sudo -u postgres createuser --interactive

Either way, the script will prompt you with some choices and, based on your responses, execute the correct Postgres commands to create a user to your specifications.
Output
Enter name of role to add: sammy
Shall the new role be a superuser? (y/n) y

# 4.Change Password User 
sudo -u user_name psql db_name
ALTER USER user_name WITH PASSWORD 'new_password';

# 5. Connect remote pgAdmin4 window with postgresql Ubuntu
Setup postgresql ubuntu
------------------

cd /etc/postgresql/15/main

nano pg_hba.conf

insert this on the last row : host    all             all             0.0.0.0/0               md5

exit and save

nano postgresql.conf

change listen_addresses = '*'                  # what IP address(es) to listen on;

exit and save




Setup pgAdmin windows
--------------------

Once you have logged into the pgAdmin client, you can connect to your database servers using the Create Server option:
Change Localhost with your db name

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/974fda18-a602-4905-a2df-5498e980dc4f)

In the first window you only need to provide an identifiable name for the server. The connection details for that server are in the connection tab below:
Change localhost with your ip address and username password with your user

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/68013b7b-80b8-44f4-a521-8a41c01dff0c)

# 6. Setup pgAgent Ubuntu and pgAdmin Windows
Installing pgAgent
------
As mentioned previously, pgAgent is not configured automatically when you install pgAdmin. You can install pgAgent from your terminal by running apt install and the package name pgagent as in the following command:

sudo apt install pgagent

After you’ve installed pgAgent, move on to the next step to configure your database to use pgAgent in pgAdmin.

Configuring your Database for pgAgent
--------------

Having followed the prerequisites, pgAdmin is set up and ready to use. You can configure your database for pgAgent use through pgAdmin. Open your web browser and navigate to the pgAdmin application at http://your_domain. Once you are logged into your account, navigate to the tree control on the left-hand side panel. Locate the database you created called sammy and expand the list. From this list, there will be an option called Extensions. Once you’ve located it, right-click on it and choose the option Query Tool:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/286bbf34-d0bc-4151-9e51-1936af5a3e77)

pgAgent requires an extension to be loaded into your database before it can be used in pgAdmin. To do so, write the following query and click on the sideways arrow ▶ signifying Execute to run the command:

CREATE EXTENSION pgagent;

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/cc45547f-5672-4e07-89a3-9b591c64de57)

Under the Messages tab there will be an output that returns Query returned successfully in 300 msec. This confirms that the pgAgent extension was created successfully.

Note: If you do not have the appropriate plpgsql language loaded to your database you will receive the following error message:

Output
ERROR:  language "plpgsql" does not exist
HINT:  Use CREATE EXTENSION to load the language into the database.
SQL state: 42704
If this happens, you need to run CREATE LANGUAGE to install the pl/pgsql procedural language required. You can install this by running the following command:

CREATE LANGUAGE plpgsql;
Once you’ve installed the pl/pgsql language, a message at the bottom will state something like Query returned successfully in 231 msec. After this, run the previous CREATE EXTENSION pgagent query again.

After you’ve run these queries, under Extensions, there will be two items listed for pgagent and plpgsql:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/ee00991c-097e-4246-adaa-f158aa365be6)

A new item in the tree control on the left-hand side will appear called pgAgent Jobs. This signifies that pgAgent was successfully installed on your pgAdmin account. Next, you will set up pgAgent as a daemon so that it can run your jobs successfully.

Note: If these items do not show up for you right away, refresh your browser page, and they should appear if your queries were successful.

 Setting Up pgAgent as a Daemon
 -----------

 our connection string will provide the credentials of your hostname, database name, and username. In our example, the host will use a Unix domain socket, the database name is sammy, and the user is sammy. This string will be appended to a pgagent command to initiate the daemon. In your terminal, you’ll run the following code:

pgagent hostaddrs=127.0.0.1 dbname=sammy user=sammy
If nothing returns in your output and you don’t receive a connection error message, then the connection string setup was successful.

After you’ve created the connection string, you’re ready to schedule a job with pgAgent.


Scheduling a Job with pgAgent
-------------

pgAgent serves as a scheduling agent that can run and manage jobs and can create jobs of one or more steps or schedules. For example, a step may consist of several SQL statements on a shell script and is executed consecutively after the other. Overall, you can use pgAgent to schedule, manage, modify, or disable your jobs.

For the purposes of this tutorial, you will use pgAgent to create a job that will back up your sammy database every minute on each day of the week. You can begin by right-clicking on pgAgent Jobs and selecting Create and then pgAgent Job… as in the following:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/914c55f0-da7c-412a-a955-39f597a5c8ad)

Once you do this, a prompt titled Create - pgAgent Job will appear, and you can begin completing the information required in the General tab. In this example, we will use the name sammy_backup and will not specify a Host agent since we want to be able to run this job on any host. Additionally, we will leave the Job class as Routine Maintenance. If you would like to include any other comments, feel free to do so in the Comment section:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/b7226a29-4ea3-4ca7-919d-179d99005a9c)

Next, navigate to the Steps tab. Click on the + symbol in the upper right-hand corner to create a step. In this example, we will name this step step1. Then, to expand your options, click on the pencil on the left-hand side of the trash bin icon. The Enabled? button is defaulted to be switched on and signifies that this step will be included when this job is executed.

For the Kind option you can select either SQL or Batch, here we have selected Batch. The reason you want to choose Batch in this example is because this is what will run the appropriate PostgreSQL commands you’ll set for the backups you want to schedule for your database. The SQL option is available for scheduling a job to execute raw SQL. In this case, we have selected Local for Connection type so that the step is executed on the local server, but if you prefer, you can also choose Remote for a remote host of your choice. If you prefer to do so on a remote host, you need to specify that criteria in the Connection string field. If you followed Step 1, your connection string is already set up and connected.

For the Database field, ensure that you have the correct database selected, here we have specified sammy. With the On error option, you can customize the pgAgent response if there’s an error when executing a step. In this case, we have selected Fail to notify us if there is an error when a step is trying to be processed. Again, if you would like to add additional notes, you can add them in the Comment box:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/73ffecb3-16cc-4c89-961c-26d97f1d8720)

Within the same Steps tab there’s also a Code tab. If you selected Batch as we did in this example, then navigate to that Code tab. Once you’re in this tab, there’s an empty line for you to insert your PostgreSQL command. You can substitute your own backup command here with your custom set of options. Any valid command is acceptable.

This tutorial will use the pg_dump command to back up your Postgres database sammy. In this command, include your specific username, the database name, and the --clean flag, which helps with pg_dump by dropping or “cleaning” the database objects before outputting any commands that are being created. For the --file flag you’re specifying the exact location where the backup files will be saved. The final part of this statement date +%Y-%m-%d-%H-%M-%S is to dynamically generate a date and multiple files for each backup. Otherwise, the backup file will constantly override and save over the existing one. This way, you can keep track of each one of your backup files for any specified time or date you scheduled. Your complete command will be as follows:

pg_dump --username=sammy --dbname=sammy --clean --file=/home/sammy/backup-`date +%Y-%m-%d-%H-%M-%S`.sql

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/efa3f2fd-1ca5-4f79-a71d-68379f938814)

Note: If you choose to save your backup files to a different location, make sure to use an absolute path to your chosen directory. For example, while ~/ normally does point to the home directory of /home/sammy/, pg_dump in this case requires the absolute path of /home/sammy/.

Once you’ve added your backup command, you can navigate to the tab labeled Schedules. Similar to when setting up Steps, click on the + symbol to add a schedule, then provide your preferred name, and click on the pencil icon next to the trash bin icon to expand your options. Under the General tab there will be the Name you wrote, in this example, it’s schedule1. Again, for Enabled this is defaulted to the on switch to ensure the schedule is properly executed. For the Start and End options, specify the starting and ending day and time for your scheduled job. Since you will be testing your scheduled job, make sure the current time is within the range of Start and End. Add a note in Comment if you prefer:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/e2951e4d-adf6-4079-a3e0-ac1469646864)

Now proceed to the Repeat tab. Here you can customize how frequently you want this scheduled job to execute. You can be as specific as possible with the week, month, date, hours, or minutes. Please note, if you do not make a selection, this is the same as choosing Select All. Therefore, if you leave the Week Days blank, your schedule will take into account all the weekdays. Similarly, with Times, you can leave the hours or minutes blank, and this is the same as Selecting All. Keep in mind that times are in cron-style format, so for this example, to generate a backup every minute, you have to select every minute in an hour (00 to 59). To demonstrate this, we’ve chosen Select All for minutes. All minutes get listed out, but leaving it blank will also achieve the same results:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/87da070f-e3e8-4935-a897-1185b919f40e)

If there are days or times you don’t want to execute a job, you can create a more granular time schedule, or you can set this by navigating to the Exceptions tab.

Note: A job is also executed based on the schedule, so anytime it is altered, the scheduled runtime will be re-calculated. When this happens, pgAgent will poll the database for the past scheduled runtime value and, from there, will typically start within a minute of that specified start time. If there are issues, then when pgAgent starts up again, it will return to the regular schedule you’ve set.

When you’re finished setting up and customizing the schedule you want to execute, press the Save button. A new pgAgent job will appear on the tree control on the left-hand side with the name of your job. For this example, sammy_backup appears with the Schedules and Steps listed beneath it:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/dc4a5c33-d793-491d-93f7-0a3784106d5f)

Now that you’ve successfully created a pgAgent job, in the next step, you will learn about how to verify your pgAgent job is running successfully.


Verifying your pgAgent Job
-------

You can check if your scheduled job to create a backup file of your database every minute is working in a couple of ways. In pgAdmin, you can navigate to the tree control on the left-hand side and click on sammy_backup. From there, proceed to the tab labeled Statistics. The Statistics page will list each instance of your scheduled job working as follows:

![image](https://github.com/ganny0219/setup-postgresql-ubuntu/assets/43429378/0db1a0b2-e245-48e4-8a18-f26218f344e7)


Please note that the statistics may not appear or refresh immediately, so you may need to navigate away or refresh the browser. Remember that your job is scheduled to run at a set interval, so keep this in mind if you’re setting a date or time that’s periodic or a longer length.






