================================
Installing PredictionIO on Linux
================================


Prerequisites
-------------

The default PredictionIO setup assumes that you have the following environment:

* A recent version of Linux (64-bit is recommended)
* Java 6+
* MongoDB 2.2+ (http://www.mongodb.org/)

The following software are optional:

* Apache Hadoop 1.0+ (or any compatible distributions that supports the
  "hadoop jar" command; see :ref:`hadoop2`)

.. note::

   You may continue the installation process without Hadoop and MongoDB.
   The setup script will install them for you quickly.

* curl
* gzip
* tar
* unzip
* zip

.. note::

   If you intend to use GraphChi algorithms, you may want to make sure your
   system has GNU libgomp installed. On Redhat-based systems, the library can
   be installed by ``yum install libgomp``. On Debian-based systems, the
   library can be installed by ``apt-get install libgomp1``.


Upgrade Notes
-------------


Version 0.7.x
~~~~~~~~~~~~~

To upgrade your installation to the latest patch level release, start by
downloading the latest binary release from the `Download
<http://prediction.io/download>`_ page.

1.  Extract the binary release to a directory separate from your current
    installation.

2.  If you launched PredictionIO by using the EC2 image from the Amazon Web
    Services Marketplace, the installation path is ``/opt/PredictionIO``.

3.  Stop all PredictionIO services by running ``bin/stop-all.sh``.

4.  Make a backup of your current PredictionIO installation.

5.  From the new release, copy the content of ``bin`` and ``lib`` directories to
    the current installation, and overwrite any files if necessary.

6.  From the new release, copy ``vendors/mahout-distribution-0.9`` to the
    current installation.

7.  From the new release, copy ``conf/init.json`` to the current installation
    and replace the existing file.

8.  From the new release, open ``conf/predictionio.conf`` and compare it to your
    existing copy. Transfer any new configuration keys to the current
    configuration file.

9.  From the new release, open ``conf/quartz.properties`` and compare it to your
    existing copy. Transfer any new configuration keys to the current file. If
    the current file does not exist, just copy it over.

10. From the new release, copy ``conf/graphchi.cnf`` to the current
    installation.

11. The 0.7.x series uses a new convention for its internal meta data and they
    must be converted when you upgrade from a previous release. To do so,
    simply run ``bin/standardized-info-ids``.

12. By now, your current installation should have all the latest files and
    patches applied. Finish the upgrade process by running ``bin/setup.sh`` and
    ``bin/setup-vendors.sh``.

13. Start PredictionIO services by running ``bin/start-all.sh``.

Congratulations! You have successfully upgraded your PredictionIO release.


Version 0.6.6+ Specific Notes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Due to significant changes in the prediction model format from previous
releases, if you are upgrading from a previous release, you will need to allow
your model to be re-trained before any prediction output becomes available.

Prediction models in the previous format will not be automatically removed. To
reclaim space, please manually delete the ``itemRecScores`` and
``itemSimScores`` collections from the model database.


Installation
------------

To start using PredictionIO, please follow the steps below.


Downloading PredictionIO Binaries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Start by downloading a `binary release <http://prediction.io/download>`_ of
PredictionIO. Unzip the binary release and change your working directory to
the extraction location.

.. code-block:: console

    $ unzip PredictionIO-<version>.zip
    $ cd PredictionIO-<version>


Preparing Your System for Hadoop (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

    This section is optional unless you plan to run Hadoop based algorithms.

If you do not have Hadoop installed, the ``bin/setup-vendors.sh`` script described
in the next step will set up one for you. Before that, there are a few steps
that must be performed manually.

#.  Please check that you can ssh to localhost without a passphrase:

    .. code-block:: console

        $ ssh localhost

    If you see any errors similar to "connection refused", it means that your
    machine's SSH service has not been enabled yet. Please enable it before you
    continue.

    If you cannot ssh to localhost without a passphrase, execute the following
    commands:

    .. code-block:: console

        $ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
        $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

    When asked whether the host key should be saved, make sure it is answered
    yes to avoid the same interactive prompt in the future.

#.  By default, Hadoop uses `/tmp` as NameNode and DataNode storage. Many
    PredictionIO users have experienced problems due to this default setting,
    thus we highly recommend this setting be changed for a smooth installation
    experience. Edit ``conf/hadoop/hdfs-site.xml`` and add:

    .. code-block:: xml

        <property>
            <name>dfs.name.dir</name>
            <value>/path_to_big_storage_for_namenode</value>
        </property>
        <property>
            <name>dfs.data.dir</name>
            <value>/path_to_big_storage_for_datanode</value>
        </property>

    Create these directories and make sure they are owned by the user that will
    start PredictionIO, and their permissions must be 0755. These directories
    must be different locations to avoid any locking errors.


Setting Up PredictionIO
~~~~~~~~~~~~~~~~~~~~~~~

Run the 3rd-party software setup script:

.. code-block:: console

    $ bin/setup-vendors.sh

If you are asked to provide your Java installation path, please type in the
*JAVA_HOME* path of a Java 6+ installation in your system.

Afterwards, run the main setup script:

.. code-block:: console

    $ bin/setup.sh


Configuring GraphChi
~~~~~~~~~~~~~~~~~~~~

If you plan to run single machine GraphChi algorithms, please adjust its
configuration according to your available hardware resource.

#.  Open and edit ``conf/graphchi.cnf``.
#.  Pick the set of configuration that match closely to your hardware resource.


Starting PredictionIO
~~~~~~~~~~~~~~~~~~~~~

.. note::

    PredictionIO depends on **MongoDB** be running to work properly. If you did
    not depend on ``bin/setup-vendors.sh`` to install it, make sure it is set
    up properly and running.

To start all PredictionIO services:

.. code-block:: console

    $ bin/start-all.sh

Now, you should be able to access PredictionIO at http://localhost:9000/!
Please proceed to the next step and create an account to access the web-based
administration panel.


Creating a User Account
~~~~~~~~~~~~~~~~~~~~~~~

.. note::

    Please make sure that **MongoDB** is running before you run this tool.

You must add at least one user to be able to log in the web panel:

.. code-block:: console

    $ bin/users


Stopping PredictionIO
~~~~~~~~~~~~~~~~~~~~~

To stop all PredictionIO services:

.. code-block:: console

    $ bin/stop-all.sh

If you are running the local Hadoop that comes with PredictionIO, you can stop Hadoop with:

.. code-block:: console

    $ vendors/hadoop-{current version}/bin/stop-all.sh


Troubleshooting
---------------

If you cannot run PredictionIO properly, please refer to
:doc:`install-predictionio-troubleshooting`.


Advanced Notes
--------------

.. _hadoop2:


Hadoop 0.22+ / 2+
~~~~~~~~~~~~~~~~~

If you are using one of these next generation Hadoop versions, distributed
Mahout jobs may not work as expected as the job JAR from the Apache Mahout
project is built against Hadoop 0.20+ / 1+. You may either compile a custom
Apache Mahout job JAR against your Hadoop distribution, or use the one that
comes with your distribution. For the latter case, it is perfectly fine to use
Apache Mahout 0.7 job JAR that comes with your distribution.

To change the location of the Apache Mahout job JAR to a non-default one,
modify the following in ``conf/predictionio.conf``.

    io.prediction.algorithms.mahout-core-job.jar=your_custom_mahout_job_jar


MongoDB at a Non-local Host
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please refer to :ref:`remote-mongodb`


Specify the Temporary Space
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default temporary space is system-specific. Under Linux, it is usually
``/tmp``. Algorithms packaged with PredictionIO generate temporary files and can
sometimes be too large for the default temporary space. To use a different
temporary space, update the configuration in ``conf/predictionio.conf``.

    io.prediction.commons.settings.local.temp.root=/a_big_temp_space/
