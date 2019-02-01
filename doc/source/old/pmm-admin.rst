.. _pmm-admin:

Managing |pmm-client|
********************************************************************************

Use the |pmm-admin| tool to manage |pmm-client|.

|chapter.toc|

.. contents::
   :local:
   :depth: 2

.. _pmm-admin.usage:

.. rubric:: USAGE

.. code-block:: text

   pmm-admin [OPTIONS] [COMMAND]

.. note:: The |pmm-admin| tool requires root access
   (you should either be logged in as a user with root privileges
   or be able to run commands with |sudo|).

To view all available commands and options,
run |pmm-admin| without any commands or options:

.. code-block:: bash

   $ sudo pmm-admin

.. _pmm-admin.options:

.. rubric:: OPTIONS

The following options can be used with any command:

``-c``, |opt.config-file|
  Specify the location of |pmm| configuration file
  (default :file:`/usr/local/percona/pmm-client/pmm.yml`).

``-h``, |opt.help|
  Print help for any command and exit.

``-v``, |opt.version|
  Print version of |pmm-client|.

|opt.verbose|
  Print verbose output.

.. _pmm-admin.commands:

.. rubric:: COMMANDS

|pmm-admin.add|_
  Add a monitoring service.

:ref:`pmm-admin.annotate`
  Add an annotation

|pmm-admin.check-network|_
  Check network connection between |pmm-client| and |pmm-server|.

|pmm-admin.config|_
  Configure how |pmm-client| communicates with |pmm-server|.

|pmm-admin.help|_
  Print help for any command and exit.

|pmm-admin.info|_
  Print information about |pmm-client|.

|pmm-admin.list|_
  List all monitoring services added for this |pmm-client|.

|pmm-admin.ping|_
  Check if |pmm-server| is alive.

|pmm-admin.purge|_
  Purge metrics data on |pmm-server|.

|pmm-admin.remove|_, |pmm-admin.rm|_
  Remove monitoring services.

|pmm-admin.repair|_
  Remove orphaned services.

|pmm-admin.restart|_
  Restart monitoring services.

|pmm-admin.show-passwords|_
  Print passwords used by |pmm-client| (stored in the configuration file).

|pmm-admin.start|_
  Start monitoring service.

|pmm-admin.stop|_
  Stop monitoring service.

|pmm-admin.uninstall|_
  Clean up |pmm-client| before uninstalling it.



.. contents::
   :local:

.. _pmm.pmm-admin.external-monitoring-service.adding:

:ref:`Adding external monitoring services <pmm.pmm-admin.external-monitoring-service.adding>`
---------------------------------------------------------------------------------------------

The |pmm-admin.add| command is also used to add external :term:`monitoring
services <External Monitoring Service>`. This command adds an external
monitoring service assuming that the underlying |prometheus| exporter is already
set up and accessible. The default scrape timeout is 10 seconds, and the
interval equals to 1 minute.

To add an external monitoring service use the |opt.external-service| monitoring
service followed by the port number, name of a |prometheus| job. These options
are required. To specify the port number the |opt.service-port| option.

.. _pmm-admin.add.external-service.service-port.postgresql:

.. include:: .res/code/pmm-admin.add.external-service.service-port.postgresql.txt

By default, the |pmm-admin.add| command automatically creates the name of the
host to be displayed in the |gui.host| field of the
|dbd.advanced-data-exploration| dashboard where the metrics of the newly added
external monitoring service will be displayed. This name matches the name of the
host where |pmm-admin| is installed. You may choose another display name when
adding the |opt.external-service| monitoring service giving it explicitly after
the |prometheus| exporter name.
		
You may also use the |opt.external-metrics| monitoring service. When using this
option, you refer to the exporter by using a URL and a port number. The
following example adds an external monitoring service which monitors a
|postgresql| instance at 192.168.200.1, port 9187. After the command completes,
the |pmm-admin.list| command shows the newly added external exporter at the
bottom of the command's output:

|tip.run-this.root|

.. _pmm-admin.add.external-metrics.postgresql:

.. include:: .res/code/pmm-admin.add.external-metrics.postresql.txt

.. seealso::

   View all added monitoring services
      See :ref:`pmm-admin.list`

   Use the external monitoring service to add |postgresql| running on an |amazon-rds| instance
      See :ref:`use-case.external-monitoring-service.postgresql.rds`
		
.. _pmm.pmm-admin.monitoring-service.pass-parameter:

:ref:`Passing options to the exporter <pmm.pmm-admin.monitoring-service.pass-parameter>`
----------------------------------------------------------------------------------------

|pmm-admin.add| sends all options which follow :option:`--` (two consecutive
dashes delimited by whitespace) to the |prometheus| exporter that the given
monitoring services uses. Each exporter has its own set of options.

|tip.run-all.root|.

.. include:: .res/code/pmm.pmm-admin.monitoring-service.pass-parameter.example.txt

.. include:: .res/code/pmm.pmm-admin.monitoring-service.pass-parameter.example2.txt

The section :ref:`pmm.list.exporter` contains all option
grouped by exporters.
   
.. _pmm.pmm-admin.mongodb.pass-ssl-parameter:

:ref:`Passing SSL parameters to the mongodb monitoring service <pmm.pmm-admin.mongodb.pass-ssl-parameter>`
----------------------------------------------------------------------------------------------------------

SSL/TLS related parameters are passed to an SSL enabled |mongodb| server as
monitoring service parameters along with the |pmm-admin.add| command when adding
the |opt.mongodb-metrics| monitoring service.

|tip.run-this.root|

.. include:: .res/code/pmm-admin.add.mongodb-metrics.mongodb-tls.txt
   
.. list-table:: Supported SSL/TLS Parameters
   :widths: 25 75
   :header-rows: 1

   * - Parameter
     - Description
   * - |opt.mongodb-tls|
     - Enable a TLS connection with mongo server
   * - |opt.mongodb-tls-ca|  *string*
     - A path to a PEM file that contains the CAs that are trusted for server connections.
       *If provided*: MongoDB servers connecting to should present a certificate signed by one of these CAs.
       *If not provided*: System default CAs are used.
   * - |opt.mongodb-tls-cert| *string*
     - A path to a PEM file that contains the certificate and, optionally, the private key in the PEM format.
       This should include the whole certificate chain.
       *If provided*: The connection will be opened via TLS to the |mongodb| server.
   * - |opt.mongodb-tls-disable-hostname-validation|
     - Do hostname validation for the server connection.
   * - |opt.mongodb-tls-private-key| *string*
     - A path to a PEM file that contains the private key (if not contained in the :option:`mongodb.tls-cert` file).

.. include:: .res/contents/note.option.mongodb-queries.txt
       
.. include:: .res/code/mongod.dbpath.profile.slowms.ratelimit.txt


.. _pmm-admin-textfile-collector:

:ref:`Extending metrics with textfile collector <pmm-admin-textfile-collector>`
--------------------------------------------------------------------------------

.. versionadded:: 1.16.0

While |pmm| provides an excellent solution for system monitoring, sometimes you
may have the need for a metric that’s not present in the list of
``node_exporter`` metrics out of the box. There is a simple method to extend the
list of available metrics without modifying the ``node_exporter`` code. It is
based on the textfile collector.

Starting from version 1.16.0, this collector is enabled for the
``linux:metrics`` in |pmm-client| by default.

The default directory for reading text files with the metrics is
``/usr/local/percona/pmm-client/textfile-collector``, and the exporter reads
files from it with the ``.prom`` extension. By default it contains an example
file  ``example.prom`` which has commented contents and can be used as a
template.

You are responsible for running a cronjob or other regular process to generate
the metric series data and write it to this directory.

Example - collecting docker container information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example will show you how to collect the number of running and stopped
docker containers on a host. It uses a ``crontab`` task, set with the following
lines in the cron configuration file (e.g. in ``/etc/crontab``)::

  */1 * * * *     root   echo -n "" > /tmp/docker_all.prom; /usr/bin/docker ps -a | sed -n '1!p'| /usr/bin/wc -l | sed -ne 's/^/node_docker_containers_total /p' >> /usr/local/percona/pmm-client/docker_all.prom;
  */1 * * * *     root   echo -n "" > /tmp/docker_running.prom; /usr/bin/docker ps | sed -n '1!p'| /usr/bin/wc -l | sed -ne 's/^/node_docker_containers_running_total /p' >>/usr/local/percona/pmm-client/docker_running.prom;

The result of the commands is placed into the ``docker_all.prom`` and
``docker_running.prom`` files and read by exporter.

The first command executed by cron is rather simple: the destination text file
is cleared by executing ``echo -n ""``, then a list of running and closed
containers is generated with ``docker ps -a``, and finally ``sed`` and ``wc``
tools are used to count the number of containers in this list and to form the
output file which looks like follows::

  node_docker_containers_total 2

The second command is similar, but it counts only running containers.






.. _pmm-admin.annotate:

:ref:`Adding annotations <pmm-admin.annotate>`
================================================================================

Use the |pmm-admin.annotate| command to set notifications about important
application events and display them on all dashboards. By using annotations, you
can conveniently analyze the impact of application events on your database.

.. _pmm-admin.annotate.usage:

.. rubric:: USAGE

|tip.run-this.root|

.. include:: .res/code/pmm-admin.annotate.tags.txt

.. _pmm-admin.annotate.options:

.. rubric:: OPTIONS

The |pmm-admin.annotate| supports the following options:

|opt.tags|

   Specify one or more tags applicable to the annotation that you are
   creating. Enclose your tags in quotes and separate individual tags by a
   comma, such as "tag 1,tag 2".

You can also use
:ref:`global options that apply to any other command <pmm-admin.options>`.

.. _pmm-admin.check-network:

:ref:`Checking network connectivity <pmm-admin.check-network>`
================================================================================

Use the |pmm-admin.check-network| command to run tests
that verify connectivity between |pmm-client| and |pmm-server|.

.. _pmm-admin.check-network.usage:

.. rubric:: USAGE

|tip.run-this.root|

.. _code.pmm-admin.check-network.options:

.. include:: .res/code/pmm-admin.check-network.options.txt
		
.. _pmm-admin.check-network.options:

.. rubric:: OPTIONS

The |pmm-admin.check-network| command does not have its own options,
but you can use :ref:`global options that apply to any other command
<pmm-admin.options>`

.. _pmm-admin.check-network.detailed-description:

.. rubric:: DETAILED DESCRIPTION

Connection tests are performed both ways, with results separated accordingly:

* ``Client --> Server``

  Pings |consul| API, |qan.name| API, and |prometheus| API
  to make sure they are alive and reachable.

  Performs a connection performance test to see the latency
  from |pmm-client| to |pmm-server|.

* ``Client <-- Server``

  Checks the status of |prometheus| endpoints
  and makes sure it can scrape metrics from corresponding exporters.

  Successful pings of |pmm-server| from |pmm-client|
  do not mean that Prometheus is able to scrape from exporters.
  If the output shows some endpoints in problem state,
  make sure that the corresponding service is running
  (see |pmm-admin.list|_).
  If the services that correspond to problematic endpoints are running,
  make sure that firewall settings on the |pmm-client| host
  allow incoming connections for corresponding ports.

.. _pmm-admin.check-network.output-example:

.. rubric:: OUTPUT EXAMPLE

.. _code.pmm-admin.check-network.output:

.. include:: .res/code/pmm-admin.check-network.output.txt

For more information, run
|pmm-admin.check-network|
|opt.help|.

.. _pmm-admin.diagnostics-for-support:

:ref:`Obtaining Diagnostics Data for Support <pmm-admin.diagnostics-for-support>`
=================================================================================

|pmm-client| is able to generate a set of files for enhanced diagnostics, which
is designed to be shared with Percona Support to solve an issue faster. This
feature fetches logs, network, and the Percona Toolkit output. To perform data
collection by |pmm-client|, execute::

   pmm-admin summary

The output will be a tarball you can examine and/or attach to your Support
ticket in the Percona's `issue tracking system <https://jira.percona.com/projects/PMM/issues>`_. The single file will look like this::

   summary__2018_10_10_16_20_00.tar.gz


.. _pmm-admin.help:

:ref:`Getting help for any command <pmm-admin.help>`
================================================================================

Use the |pmm-admin.help| command to print help for any command.

.. _pmm-admin.help.usage:

.. rubric:: USAGE

|tip.run-this.root|

.. _code.pmm-admin.help.command:

.. include:: .res/code/pmm-admin.help.command.txt

This will print help information and exit.  The actual command is not run
and options are ignored.

.. note:: You can also use the global |opt.h| or |opt.help| option after any
   command to get the same help information.

.. _pmm-admin.help.commands:

.. rubric:: COMMANDS

You can print help information for any :ref:`command <pmm-admin.commands>`
or :ref:`service alias <pmm-admin.service-aliases>`.

.. _pmm-admin.ping:

:ref:`Pinging PMM Server <pmm-admin.ping>`
================================================================================

Use the |pmm-admin.ping| command to verify connectivity with |pmm-server|.

.. _pmm-admin.ping.usage:

.. rubric:: USAGE

|tip.run-this.root|

.. _code.pmm-admin.ping.options:

.. include:: .res/code/pmm-admin.ping.options.txt

If the ping is successful, it returns ``OK``.

.. _code.pmm-admin.ping:

.. include:: .res/code/pmm-admin.ping.txt

.. _pmm-admin.ping.options:

.. rubric:: OPTIONS

The |pmm-admin.ping| command does not have its own options,
but you can use :ref:`global options that apply to any other command
<pmm-admin.options>`.

For more information, run
|pmm-admin.ping|
|opt.help|.






.. include:: .res/replace.txt
