.. index:: eclipse

Using Eclipse
=============

In this cookbook, we describe how to start working on the S4 codebase using Eclipse.

Using Eclipse to edit S4 codebase
----------------------------------

Formatting settings
^^^^^^^^^^^^^^^^^^^

When editing the S4 codebase, use the following formatting settings:

  * Spaces for indentation
  * Tab width = 4

Importing into eclipse
^^^^^^^^^^^^^^^^^^^^^^

  1. Clone the S4 repository and build according to :ref:`Set Up S4 <getting_started_set_up>`.
  2. Create eclipse configuration files: ``./gradlew eclipse``.
  3. From Eclipse, select file->import->Existing Projects into Workspace. Select the base of the newly cloned repository (e.g., :file:`<source base>/s4`) as the root directory.

This will import several subprojects into your workspace.

Using The S4 Image to Build an Application
------------------------------------------

If you want to build an application using the stock S4 core library, you can `download the image <http://docs.s4.io/tutorials/getting_started.html#download>`_ and use that in the eclipse project.

1. Suppose the image directory is called ``S4IMAGE``. All the required jars are contained in ``S4IMAGE/s4_core/lib``. They have to be installed to your maven repository.

  * ``cd S4IMAGE/s4_core/lib``
  * ``mvn install:install-file -DgroupId=com.esotericsoftware -DartifactId=kryo -Dversion=1.01 -Dpackaging=jar -Dfile=kryo-1.01.jar``
  * ``mvn install:install-file -DgroupId=com.esotericsoftware -DartifactId=reflectasm -Dversion=0.8 -Dpackaging=jar -Dfile=reflectasm-0.8.jar``
  * ``mvn install:install-file -DgroupId=com.esotericsoftware -DartifactId=minlog -Dversion=1.2 -Dpackaging=jar -Dfile=minlog-1.2.jar``
  * ``mvn install:install-file -DgroupId=org.apache.hadoop -DartifactId=zookeeper -Dversion=3.1.1 -Dpackaging=jar -Dfile=zookeeper-3.1.1.jar``
  * ``mvn install:install-file -DgroupId=io.s4 -DartifactId=comm -Dversion=0.2.1.0 -Dpackaging=jar -Dfile=comm-0.2.1.0.jar``
  * ``mvn install:install-file -DgroupId=io.s4 -DartifactId=s4_core -Dversion=0.2.1.0 -Dpackaging=jar -Dfile=s4_core-0.2.1.0.jar``

2. Let us assume the application is located in the directory ``APPDIR``.

   * For an example app, refer to the `twittertopiccount <https://github.com/s4/examples/tree/master/twittertopiccount>`_ application.

2. Create Eclipse configurations in ``APPDIR``

  * ``cd APPDIR``
  * ``mvn eclipse:eclipse``

3. Import project into Eclipse

  * :menuselection:`File --> Import --> General --> Existing Projects into Workspace`
  * Select ``APPDIR`` from above as the "root directory" of the project.

4. Set variable ``M2_REPO`` to point to the local Maven repository

  * This is where Maven stores its local repository, e.g. ``~/.m2/repository``
  * Adding a variable in Eclipse: :menuselection:`Properties --> Java Build Path --> Libraries --> Add Variable`

5. Set up formatting:

  * Spaces for indentation
  * Tab width = 4