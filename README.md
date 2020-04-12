Alerta API on Google Cloud
==========================

[Alerta](https://alerta.io) is a monitoring tool used for alert
consolidation, de-duplication and correlation.

To deploy Alerta API to [Google Cloud](https://cloud.google.com) use the
[Google App Engine Flexible Environment](https://cloud.google.com/appengine/docs/flexible/)
with [Cloud SQL for PostgreSQL](https://cloud.google.com/appengine/docs/flexible/python/using-cloud-sql-postgres)
for a fully managed service.

**Note: The following steps will incur costs. See https://cloud.google.com/sql/pricing#pg-pricing**

Installation
------------

Run the Alerta API in Google Cloud by cloning this repo, creating a
Cloud SQL instance and deploying the application to Google App Engine.

To create an PostgreSQL instance either use the Google Cloud Console
or install the `gcloud` SDK and run:
    $ gcloud config set project [PROJECT_NUMBER]

    $ gcloud sql instances create [INSTANCE_NAME] \
    --database-version=POSTGRES_11 --storage-type=SSD --tier=db-f1-micro --region "us-central"

    $ gcloud sql users set-password postgres \
    --instance [INSTANCE_NAME] --password [PASSWORD]

Note: Change the `--storage-type`,  `--tier` and --region as appropriate.

Log in to `psql` to create a database using [GCloud Shell](https://cloud.google.com/shell/docs/)
or `gcloud` (you will be prompted for the password set above):

    $ gcloud sql connect [INSTANCE_NAME] --user postgres

    Password for user postgres:
    *******
    psql (9.6.2, server 9.6.1)
    SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES128-GCM-SHA256, bits: 128, compression: off)
    Type "help" for help.

    postgres=> CREATE DATABASE monitoring;
    CREATE DATABASE

Configuration
-------------

Copy the `app.yaml.example` file to `app.yaml` and set the `DATABASE_URL`
and `beta_settings` for the PostgreSQL instance using the *INSTANCE_CONNECTION_NAME*
generated above:

```
env_variables:
  DATABASE_URL: postgres://USER:PASSWORD@/DATABASE?host=/cloudsql/INSTANCE_CONNECTION_NAME
  CORS_ORIGINS: [
    'http://localhost',
    'http://localhost:8000'
    ]

beta_settings:
    cloud_sql_instances: INSTANCE_CONNECTION_NAME
```

**Example**

```
env_variables:
  DATABASE_URL: postgres://postgres:p0stgr3s@/monitoring?host=/cloudsql/alerta5:europe-west2:monitoring
  CORS_ORIGINS: [
    'http://localhost',
    'http://localhost:8000',
    'https://myaltertagui.com'
    ]

beta_settings:
    cloud_sql_instances: alerta5:europe-west2:monitoring
```

Deploy
------

To deploy the Alerta API and lauch a browser to view the API index page run:

    $ gcloud app deploy
    $ gcloud app browse

Note: Add `--verbosity=info` to any `gcloud` command to get more verbose logging.

Add the service account of your flex app to the Cloud SQL Client role
https://cloud.google.com/sql/docs/mysql/connect-app-engine

Housekeeping
------------

An [App Engine Cron schedule](https://cloud.google.com/appengine/docs/flexible/python/scheduling-jobs-with-cron-yaml)
can be configured to run a housekeeping job every minute which expires alerts
which have timed-out and deletes expired and closed alerts.

To enable the cron job run:

    $ gcloud app deploy cron.yaml

Scaling Down
------------

To scale down the app deploy using `--version dev` then it can be stopped
and started easily:

    $ gcloud app deploy --version dev
    $ gcloud app versions stop dev
    $ gcloud app versions start dev

Testing
-------

Create a [Service Account]() for remote access and download the Cloud SQL Proxy.

    $ cloud_sql_proxy ...
    
TBC


Troubleshooting
---------------

All `gcloud` commands take `--verbosity` which can be used to output more verbose
logging like so:

    $ gcloud app deploy --verbosity=info

To tail the application logs run:

    $ gcloud app logs tail -s default

```
2017-09-23 10:57:59 default[20170923t115438]  2017/09/23 10:57:59 Ready for new connections
2017-09-23 10:57:59 default[20170923t115438]  2017/09/23 10:57:59 errors parsing config:
2017-09-23 10:57:59 default[20170923t115438]  	googleapi: Error 403: Access Not Configured.
Cloud SQL Administration API has not been used in project 465218335446 before or it is disabled.
Enable it by visiting https://console.developers.google.com/apis/api/sqladmin.googleapis.com/overview?project=465218335446
then retry. If you enabled this API recently, wait a few minutes for the action to propagate
to our systems and retry., accessNotConfigured
```

To connect to the database instance using `psql` run:

    $ gcloud sql connect [INSTANCE_NAME] --user postgres

References
----------

  * App Engine Flexible Environment: https://cloud.google.com/appengine/docs/flexible/
  * Quickstart for Python in the App Engine Flexible Environment: https://cloud.google.com/appengine/docs/flexible/python/quickstart
  * Using Cloud SQL with Python: https://cloud.google.com/python/getting-started/using-cloud-sql
  * Using Cloud SQL for PostgreSQL: https://cloud.google.com/appengine/docs/flexible/python/using-cloud-sql-postgres
  * Connect to Cloud SQL from App Engine https://cloud.google.com/sql/docs/mysql/connect-app-engine

License
-------

Copyright (c) 2017 Nick Satterly. Available under the MIT License.
