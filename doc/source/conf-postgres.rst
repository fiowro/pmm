.. _pmm.qan.postgres.conf:

===============================================================================
Configuring |postgresql| for Monitoring
===============================================================================

Monitoring |postgresql| metrics with the ``postgres_exporter`` can be turned on
with ``pmm-admin add postgresql`` command. The ``postgresql`` alias will set up
``postgresql:metrics`` and also ``linux:metrics`` on a host (for more
information, see `Adding monitoring services <https://www.percona.com/doc/percona-monitoring-and-management/pmm-admin.html#pmm-admin-add>`_).

Several parameters are expected by postgres_exporter to make things work. 

The 1st parameter is an example URI, which will be built up based on the correct flags being passed to pmm-admin. An example of the URI is::

   DATA_SOURCE_NAME=postgresql://postgres_exporter:password@172.17.0.2:5432/postgres?sslmode=disable

``pmm-admin`` supports passing |postgresql| connection information via following flags:

=============                 =================================================
Flag                          Description 
=============                 =================================================
``--create-user``             create a new |postgresql| user
``--create-user-password``    optional password for a new PostgreSQL user
``--force``                   force user creation
``--host``                    |postgresql| host
``--password``                |postgresql| password
``--port``                    |postgresql| port
``--user``                    |postgresql| user
=============                 =================================================

An example command line would look like this::

  pmm-admin add postgresql --create-user --host=172.17.0.2 --password=ABC123 --port=5432 --user=postgres_exporter

Supported versions of PostgreSQL
--------------------------------

PMM  supports monitoring PostgreSQL version 10 or higher.


.. _pmm.qan.mongodb.conf.essential-permission.setting-up:

Setting Up the Essential Permissions
================================================================================

User creation should follow these permissions)::

      CREATE USER postgres_exporter PASSWORD 'password';
      ALTER USER postgres_exporter SET SEARCH_PATH TO postgres_exporter,pg_catalog; -- If deploying as non-superuser (for example in AWS RDS)
      -- GRANT postgres_exporter TO :MASTER_USER;
      CREATE SCHEMA postgres_exporter AUTHORIZATION postgres_exporter; CREATE VIEW postgres_exporter.pg_stat_activity
      AS
      SELECT * from pg_catalog.pg_stat_activity; GRANT SELECT ON postgres_exporter.pg_stat_activity TO postgres_exporter; CREATE VIEW postgres_exporter.pg_stat_replication AS
      SELECT * from pg_catalog.pg_stat_replication; GRANT SELECT ON postgres_exporter.pg_stat_replication TO postgres_exporter;

.. include:: .res/replace.txt
