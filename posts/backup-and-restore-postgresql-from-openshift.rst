.. title: Backup and Restore PostgreSQL from Openshift
.. slug: backup-and-restore-postgresql-from-openshift
.. date: 2015-04-15 22:31:49 UTC+02:00
.. tags:
.. category:
.. link: 
.. description:
.. type: text

I have my blog hosted on openshift using django.
This is how i back up a the postgres database, copy it to my local computer, and restore into a local database. The backup uses this snippet which u can use as a cron job. I ssh'ed into my app:

::

	rhc ssh myappname

executed the lines from this `snippet <https://gist.github.com/skyebook/3407646>`_:


.. code:: bash

	#!/bin/bash
	# Backs up the OpenShift PostgreSQL database for this application
	# by Skye Book
	NOW="$(date +"%Y-%m-%d")"
	FILENAME="$OPENSHIFT_DATA_DIR/$OPENSHIFT_APP_NAME.$NOW.backup.sql.gz"
	pg_dump $OPENSHIFT_APP_NAME | gzip > $FILENAME

Now you can copy the database to your local folder:

::

	rhc scp myappname download ./ app-root/data/myappname.2015-03-27.backup.sql.gz

``./`` copies into the current working directory.
Now create a new database in pgadmin and restore the data to it:

::

	psql -d nameofnewdb -U username -f myappname.2015-03-27.backup.sql

Now you can connect the dhango app to your local database.
