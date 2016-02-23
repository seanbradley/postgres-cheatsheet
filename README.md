#POSTGRESQL (ON UBUNTU)

Check out these resources...

https://help.ubuntu.com/community/PostgreSQL

https://www.digitalocean.com/community/tutorials/how-to-use-postgresql-with-your-django-application-on-ubuntu-14-04

Note: my instructions below are old, but you may find them helpful--and a good cheatsheet from time to time. I believe the current stable version is 9.3.x...at the most recent updating these instructions. Minor tweaks to my commands may be necessary to accommodate the most recent version.

Throughout the following instructions... 

- angle brackets < > indicate values you must type or fill-in at the command line
- Ubuntu commands may or may not be prefaced with the $ BASH shell prompt
- PostgreSQL commands may or may not be prefaced pwith the postgres=# shell prompt

Feel free to clean the above indicators up and commit your changes to this doc. I'd appreciate that. :-) 


---
##INSTALLING POSTGRESQL
---

You can install PostgreSQL system wide with sudo apt-get...or you can use pip and manage its installation Pythonically rather than through Ubuntu's package manager.


---
##CREATE (OR RECREATE) A DATABASE FROM SCRATCH
---

###STEP 1: LOGIN TO THE POSTGRES SHELL

    psql -U postgres -h localhost

You will be prompted for a password.  The password is 'postgres'.

An old way: if you set PGHOST=localhost as an environment variable, you don't need to specify the -h option every time. This also works with other pg_* commands such as pg_dump.

Add'l documentation here (again, some of this may be deprecated):

http://help.ubuntu.com/12.04/serverguide/postgresql.html
https://help.ubuntu.com/10.04/serverguide/postgresql.html
http://od-eon.com/blogs/calvin/postgresql-cheat-sheet-beginners

From here on forward, throughout these instructions, this...

    postgres=#

represents the prompt in the postgres shell.


###STEP 2: CREATE A SUPERUSER (IF IT DOES NOT ALREADY EXIST)

In this case, $USER is an environment variable which may need to be defined (probably in your ~/.bashrc file) prior to executing these commands. You can also explicitly enter the user when prompted.

    sudo -u postgres psql postgres

    \password postgres

Enter ctrl+D to exit the PostgreSQL prompt.

You can also create the superuser from the Ubuntu command line...

    sudo -u postgres createuser --superuser $USER
    sudo -u postgres psql

    postgres=# \password $USER
    
Likewise, the database can be created from Ubuntu's command line...

    sudo -u postgres createdb <dbname>

Client programs on Ubuntu, by default, connect to the localhost using your Ubuntu login name and expect to find a database with that name, too. So to make things REALLY easy, use your new superuser privileges granted above to create a database with the same name as your login name:

    createdb $USER

That means, to make things very simple locally, set the db name and the superuser name to the same value as your Ubuntu login name.


###STEP 3: CONNECT TO THE NEW DB LOCALLY

If you do the above, connecting to your own database locally to try out some SQL should now be as easy as:

    psql

The above will get you going, but we need a way to be more explicit about the superuser we're creating, and about the name of the database itself...


###STEP 2 (ALTERNATE): CREATE A NEW USER FROM THE POSTGRESQL SHELL

The above really applies only to local development. This way is more broadly applicable than outlined thus far...

Login to the PostgreSQL as the postgres user--same as above.

Now, we'll create a hypothetical new user named John... John will not have superuser powers at first.

Type the following SQL command to create a user john with myPassword as his password:

    postgres=# CREATE USER john WITH PASSWORD 'myPassword';

More info here on how to add a user can be found here...
http://www.cyberciti.biz/faq/howto-add-postgresql-user-account/

If you use this method, you will only be able to login to PostgreSQL as the postgres user, or the user you specify in the in your CREATE USER command.


###STEP 4: CREATE A NEW DATABASE; ASSIGN AN OWNER OR CHANGE THE OWNER

Type the following command to create a database named 'jane':

    postgres=# CREATE DATABASE jane WITH OWNER john;

    ...or to change owner of existing database...

    postgres=# ALTER DATABASE name OWNER TO new_owner;


##STEP 4: GIVE THE NEW USER SUPER POWERS ON YOUR NEW DATABASE

Now, let's grant all privileges on database jane to user john...thus, making John a superuser.

    postgres=# GRANT ALL PRIVILEGES ON DATABASE jane to john;

So, now we have things set-up like so...

- database name = jane
- superuser name= john
- superuser's password = myPassword

If you're using Django, make adjustments to the appropriate settings.py file accordingly. You will also need to point the settings file to the appropriate database URL or IP if you are not developing locally.

If you are setting up an existing Django project from a private repo, there may be existing database name, database user name, and database user password info in the repo. Best practice suggests these secret config values should be handled by each developer with environment variables. Refer to the Two Scoops book.


##STEP 5: START OR RESTART THE DATABASE SERVER

Exit the postgres shell (ctrl-d), and enter the following at the bash prompt...

    sudo /etc/init.d/postgresql restart


##STEP 6: PUT DATA INTO THE DATABASE

If you're using Django, execute the following command...

    python manage.py syncdb

Sometimes, you may need to include a flag, like...

    python manage.py syncdb --all
   
...or...

    python manage.py syncdb --noinput

See the Two Scoops book and/or the official Django docs for more info on various manage.py commands and flags.

If you're using South for migrations, execute the following command for each Django app...

    python manage.py migrate <appname>
    
Entering just...

    python manage.py migrate
    
...migrates data for _all_ tables.

Finally, you'll also need to load your test data via any fixtures, if they exist, like so...

    python manage.py loaddata <fixture_name>
    
If you do not specify which fixture, the above command will load all fixtures if they are in the appropriate /fixtures directories.

That's it!

---
#ADDITIONAL COMMANDS
---

##DROP A DATABASE

WARNING!!! This will totally dust the database... You will have to recreate the database for your project.

    DROP DATABASE "<database_name>";


##DROP ALL TABLES

First, login to PostgreSQL and tell PostgreSQL which database you want to work on...

...to list all databases:

    \l 
   
...to connect to a specific database:

    \c <database_name> 
    
Sometimes, when working--for instance--on a Django project, you want to blow away all the tables in your project's database, but do not have to go through the drama of recreating that particular database in full. 

You can achieve the same effect by dropping the database's default schema. If you do this, you'll have to immediately recreate that schema.

    drop schema public cascade;
    
    create schema public;

Alternately...

http://blog.impressiver.com/post/37947630374/drop-all-database-tables-for-a-django-project-without


##LIST ALL USERS

    \du


##RESET THE PASSWORD FOR A USER

    sudo -u <username> psql

Here the user is 'postgres'...

    sudo -u postgres psql
    
    \password

    
##RESTART THE DB SERVER

    /etc/init.d/postgresql + stop/start/restart
    

##OTHER MISC COMMANDS

    \? for help with psql commands (in regular SQL, this is \h )
    
    \g or terminate with semicolon to execute query
    
    \q to quit

---
#DATABASE INTROSPECTION: LOOKING UNDER THE HOOD
---

##USE THE POSTGRESQL ADMIN GUI

To get an idea of what PostgreSQL can do, you may want to install the pgAdmin tool, and fire up a graphical client. In a terminal, type:

    pgadmin3

You will be presented with the pgAdmin III interface. Click on the "Add a connection to a server" button (top left). In the new dialog, enter the address 127.0.0.1, a description of the server, the default database ("jane" in the example above), your username ("john") and your password.

With this GUI you may start creating and managing databases, query the database, execute SQl, etc.

The following are commands you would execute at the PostgreSQL prompt... 


Via http://www.linuxscrew.com/2009/07/03/postgresql-show-tables-show-databases-show-columns/

##USING SHORTCUTS FROM THE POSTGRESQL SHELL

PostgreSQL is one of the best database engines for an average web project and many who move to PostgreSQL from SQL, MySQL, or TSQL (for example) often ask the following questions: 

"What is the analog of “show tables” in Postgres?" 

Or: 

"How can I get the list of databases in postgres like 'show databases;' in MySql?" 

The answers are short. Below, for reference, if you are familiar with SQL, I've included the command in SQL, and follow those with the PostgreSQL shortcuts, as well as the full command in PostgreSQL.

- mysql: SHOW TABLES
- postgresql shortcut: \d
- postgresql full: SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';

- mysql: SHOW DATABASES
- postgresql shortcut: \l
- postgresql full command: SELECT datname FROM pg_database;

- mysql: SHOW COLUMNS
- postgresql shortcut: \d table
- postgresql full command: SELECT column_name FROM information_schema.columns WHERE table_name ='table';

- mysql: DESCRIBE TABLE
- postgresql shortcut: \d+ table
- postgresql full command: SELECT column_name FROM information_schema.columns WHERE table_name ='table';

---
#WORKING ON THE DB THROUGH DJANGO
---

It may be easier to do things like add a user, etc., via an interactive Python console rather than the PostgreSQL shell. You can access the db via Django via this command...

    python manage.py dbshell
    
To get the postgres=# prompt, you may need to append the above command with the Django settings flag, depending on the environment in which you are working...like so--for example--for your local environment:

    python manage.py dbshell --settings=<project_name>.settings.local

You can also open up a Python shell through the Django manage.py command, and interact with the database through Django's ORM...

    python manage.py shell --settings=<project_name>.settings.local

...where you can execute Python commands to manipulate the database like so:

    >>> from django.contrib.auth.models import User
    >>> user = User.objects.create_user(username='john',
    ...                                 email='jlennon@beatles.com',
    ...                                 password='glass onion')
    >>> user.is_superuser = True
    >>> user.save()

Or...

    >>> user = User.objects.get(username='john')
    >>> user.set_password('goo goo goo joob')
    >>> user.save()

Or just...

    python manage.py createsuperuser --username=<username> --email=<user_email>

If you're trying to create a superuser in Django, make sure you review the following...

https://docs.djangoproject.com/en/dev/topics/auth/default/#user-objects

---
#MORE ON DJANGO MIGRATIONS
---

Here's a nice resource...

https://realpython.com/blog/python/django-migrations-a-primer/

Migrations have changed in recent versions of Django. You may hear about an old migrations manager named South. You will need to use it in older versions of Django.

From http://andrewingram.net/2012/dec/common-pitfalls-django-south/  ...

##MANAGE YOUR DEPENDENCIES WHEN USING SOUTH

In each migration you can declare a depends_on field which tells South what other migrations must be run beforehand. When you create a relationship to another app (for example from basket to catalogue), it's important that you tell South to first run the migration which creates the database table at the target end of the relation.

Often you'll find you can get by without doing this, it depends on the way your database handles foreign keys (PostgreSQL is nice and strict about this by default, so you'll see the errors). But the order of your INSTALLED_APPS has an impact on this too, when running manage.py migrate your apps will be migrated in the order they're defined in your settings.py file. You may find your dependencies are implicitly taken care of, but my recommendation is to always do it explicitly. You're less likely to have a problem if someone re-orders your apps (for whatever reason) and it's, generally, more Pythonic to be explicit. :-)
