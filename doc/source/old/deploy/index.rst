.. _deploy-pmm:

Deploying |pmm.name|
********************************************************************************

|abbr.pmm| is designed to be scalable for various environments.  If you have
just one |mysql| or |mongodb| server, you can install and run both |abbr.pmm|
server and |abbr.pmm| clients on one database host.

It is more typical to have several |mysql| and |mongodb| server instances
distributed over different hosts. In this case, you need to install the
|abbr.pmm| client package on each database host that you want to monitor. In
this scenario, the |abbr.pmm| server is set up on a dedicated monitoring host.

|chapter.toc|

.. contents::
   :local:
   :depth: 1

.. _deploy-pmm.server.installing:

:ref:`Installing PMM Server <deploy-pmm.server.installing>`
================================================================================

To install and set up the |pmm-server|, use one of the following options:

.. -  :ref:`run-server-docker`
.. -  :ref:`pmm.deploying.server.virtual-appliance`
.. -  :ref:`run-server-ami`

.. toctree::
   :maxdepth: 1

   server/docker
   server/virtual-appliance
   server/ami

.. include:: ../.res/contents/important.port.txt

.. _deploy-pmm.server.verifying:

:ref:`Verifying PMM Server <deploy-pmm.server.verifying>`
--------------------------------------------------------------------------------

In your browser, go to the server by its IP address. If you run your server as a
virtual appliance or by using an |amazon| machine image, you will need to setup
the user name, password and your public key if you intend to connect to the
server by using ssh. This step is not needed if you run |pmm-server| using
|docker|.

In the given example, you would need to direct your browser to
*http://192.168.100.1*. Since you have not added any monitoring services yet,
the site will not show any data.

.. _deploy-pmm.table.web-interface.component.access:

.. table:: Accessing the Components of the Web Interface

   ==================================== ======================================
   Component                            URL
   ==================================== ======================================
   :term:`PMM Home Page`                ``http://192.168.100.1``
   :term:`Metrics Monitor (MM)`         | ``http://192.168.100.1/graph/``
                                        | User name: ``admin``
                                        | Password: ``admin``
   Orchestrator                         ``http://192.168.100.1/orchestrator``
   ==================================== ======================================

You can also check if |pmm-server| is available requesting the /ping
URL as in the following example:

.. include:: ../.res/code/curl.ping.txt

.. _deploy-pmm.data-collecting:

:ref:`Collecting Data from PMM Clients on PMM Server <deploy-pmm.data-collecting>`
==================================================================================

To start collecting data on each |pmm-client| connected to a |abbr.pmm|
server, run the |pmm-admin.add| command along with the name of the selected
monitoring service.

|tip.run-all.root|.

Enable general system metrics, |mysql| metrics, |mysql| query analytics:
   .. include:: ../.res/code/pmm-admin.add.mysql.txt

Enable general system metrics, |mongodb| metrics, and |mongodb| query analytics:
   .. include:: ../.res/code/pmm-admin.add.mongodb.txt

Enable |proxysql| performance metrics:
   .. include:: ../.res/code/pmm-admin.add.proxysql-metrics.txt

To see what is being monitored, run |pmm-admin.list|. For example, if you enable
general OS and |mongodb| metrics monitoring, the output should be similar to the
following:

.. include:: ../.res/code/pmm-admin.list.txt

.. seealso::

   What other monitoring services can I add using the |pmm-admin.add| command?
      Run :program:`pmm-admin add --help` in your terminal

.. _deploy-pmm.diagnostics-for-support:

:ref:`Obtaining Diagnostics Data for Support <deploy-pmm.diagnostics-for-support>`
==================================================================================

|pmm-server| is able to generate a set of files for enhanced diagnostics, which
can be examined and/or shared with Percona Support to solve an issue faster.

Collected data are provided by the ``logs.zip`` service, and cover the following
subjects:

* Prometheus targets
* Consul nodes, QAN API instances
* Amazon RDS and Aurora instances
* Version
* Server configuration
* Percona Toolkit commands

You can retrieve collected data from your |pmm-server| in a single zip archive
using this URL::

   https://<address-of-your-pmm-server>/managed/logs.zip

.. _deploy-pmm.updating:

:ref:`Updating <deploy-pmm.updating>`
================================================================================

When changing to a new version of |pmm|, you update the |pmm-server| and each
|pmm-client| separately.

.. rubric:: Updating the |pmm-server|

The updating procedure of your |pmm-server| depends on the option that you
selected for installing it.

If you are running |pmm-server| as a :ref:`virtual appliance <pmm.deploying.server.virtual-appliance>` or using an :ref:`Amazon Machine Image
<run-server-ami>`, use the |gui.check-for-updates-manually| button on the Home
dashboard (see :term:`PMM Home Page`).

.. figure:: ../.res/graphics/png/pmm.home-page.1.png

   Click |gui.check-for-updates-manually| to updating the |pmm-server| from the
   |pmm| home page.

.. seealso::

   How to update |pmm-server| installed using |docker|?
      :ref:`update-server.docker`

.. rubric:: Updating a |pmm-client|

When a newer version of |pmm-client| becomes available, you can update to it
from  the |percona| software repositories:

|debian| or |ubuntu|
   .. include:: ../.res/code/apt-get.update.install.pmm-client.txt

|red-hat| or |centos|
   .. include:: ../.res/code/yum.update.pmm-client.txt

If you installed your |abbr.pmm| client manually, :ref:`remove it
<deploy-pmm.removing>` and then :ref:`download and install a newer version
<deploy-pmm.client.installing>`.

.. _deploy-pmm.removing:

:ref:`Uninstalling PMM Components <deploy-pmm.removing>`
================================================================================

Each |pmm-client| and the |pmm-server| are removed separately. First, remove all
monitored services by using the |pmm-admin.remove| command (see
:ref:`pmm-admin.rm`). Then you can remove each |pmm-client| and the
|pmm-server|.

.. _deploy.pmm-client.removing:

:ref:`Removing the PMM Client <deploy.pmm-client.removing>`
--------------------------------------------------------------------------------

Remove all monitored instances as described in :ref:`pmm-admin.rm`. Then,
uninstall the |pmm-admin| package. The exact procedure of removing the
|pmm-client| depends on the method of installation.

|tip.run-all.root|

Using YUM
   .. include:: ../.res/code/yum.remove.pmm-client.txt
		  
Using APT
   .. include:: ../.res/code/apt-get.remove.pmm-client.txt
		  
Manually installed RPM package
   .. include:: ../.res/code/rpm.e.pmm-client.txt

Manually installed DEB package
   .. include:: ../.res/code/dpkg.r.pmm-client.txt

Using the generic |pmm-client| tarball.
  |cd| into the directory where you extracted the tarball
  contents. Then, run the :file:`unistall` script:
  
  .. include:: ../.res/code/uninstall.txt

.. _deploy.pmm-server.removing:

:ref:`Removing the PMM Server <deploy.pmm-server.removing>`
--------------------------------------------------------------------------------

If you run your |pmm-server| using |docker|, stop the container as follows:

.. include:: ../.res/code/docker.stop.pmm-server.rm.txt

To discard all collected data (if you do not plan to use
|pmm-server| in the future), remove the ``pmm-data``
container:

.. include:: ../.res/code/docker.rm.pmm-data.txt

If you run your |pmm-server| using a virtual appliance, just stop and
remove it.

To terminate the |pmm-server| running from an |amazon| machine image, run
the following command in your terminal:

.. include:: ../.res/code/aws.ec2.terminate-instances.txt

.. seealso::

   |pmm| Building Blocks
      :ref:`pmm.architecture`
   About using the |pmm-admin.add| command
      :ref:`pmm-admin.add`

.. include:: ../.res/replace.txt
