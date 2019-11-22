Requiremenents
==============

* **Disk Space**\: DisCVR.jar requires **~700MB space** for installation with the built-in databases. It is
  recommended to have space for 2x sample size when using DisCVR for classification as the
  process involves writing temporary files to disk. When building a customised database, the
  amount of space depends on the size of the viral sequences and the *k* size. For example,
  extracting *k*\-mers of size 32 from the human genomes generates a file that is 80GB in size. If
  the viral data sequences are 3GB in size, then the minimum disk space needed to build a
  customised database is 200GB.
* **Memory**\: DisCVR runs efficiently on a machine with **4GB RAM**, which is the current standard
  for PCs. It is much faster on machines with higher RAM such as **8GB RAM**. However, the
  amount of RAM depends on the number of the sequences used and the size of the k-mer. The
  larger the dataset and the k-mer, the larger the amount of RAM needed. Therefore, in the
  case of “out of memory” errors, the Java heap space should be increased.


Installation
============

* Operating System: DisCVR runs on both Windows and Linux platforms. To use DisCVR, the
  users need first to download the appropriate folder for their operating system.
  
  * `Windows OS (771Mb) <http://bioinformatics.cvr.ac.uk/DisCVR/Downloads/DisCVR_Windows.zip>`_
  * `Unix (818Mb) - tested on MacOS and Ubuntu <http://bioinformatics.cvr.ac.uk/DisCVR/Downloads/DisCVR_Unix.zip>`_ 
  * `Command-line for multiple classifications (645Mb) <http://bioinformatics.cvr.ac.uk/DisCVR/Downloads/DisCVR_CL.jar>`_
  * `View source codes on GitHub <https://github.com/centre-for-virus-research/DisCVR>`_ 
  
* Java: **64-bit Java (1.8 or above)** must be installed and the full path to the jre\/bin folder should be
  included in the system variables. Java can be downloaded from:
  `Link to Oracle <http://www.oracle.com/technetwork/java/javase/downloads/jre8-
  downloads-2133155.html>`_
* DisCVR.jar: After downloading the DisCVR zipped folder, it is recommended to use a tool, such
  as 7-zip, to unzip the Windows OS version and extract all files to a local directory. For Linux
  and Mac version, open a command prompt and move to the location of the zipped folder.
  Type the following commands to unzip the folder:
  ``tar -xzvf DisCVR_Linux.tar.gz``
  
  This creates a folder, called DisCVR. The contents of DisCVR consists of one jar: DisCVR.jar and
  a lib folder which are used to run the classification. The script file: downloadDataAndRefSeq.sh
  and the folders: bin, customisedDB, and TestData which are needed to build a customised
  database.
  

* Dependencies: DisCVR uses external libraries such as KAnalyze, for k-mers counting, and
  JFreechart packages, for graphs plotting. It makes use of Tanoti, a Blast-based tool for
  reference assembly. These are 10 files in total and they are in the lib folder. It is important
  not to alter the lib folder or its contents and to ensure that it is in the same path as the jar file.

* Compiling the source code: The easiest way to compile the source codes is to use the Ant Apache build tool 
  which is a free software and easy to install (`https://ant.apache.org <https://ant.apache.org/>`_). The build.xml file is included with the source codes 
  which is used to compile all the files before creating the jar. Once the Ant tool is installed you need to add the build.xml 
  file to the same path as the source code and use the command:
  ``ant```
  Ant will take care of compiling all the files and sorting out dependencies on external libraries. 
  In that sense, you only need to modify the build.xml file to achieve your goals from compiling the source codes. 
  The `Customised Database section <customdb.rst>`_ in the operating manual has a detailed example of how to compile and 
  create the GUI using your own database.

Testing
=======

To test if the tools are installed properly, open a command prompt and type the following:

* To know what Java version is installed: ``java –version``. This should state ``java version 1.8.0_<some number>``
  
* To see if Java is added to the path: ``java``.   If the jre\/bin is not added to the path, you will see the following message:
  ``java is not recognized as an internal or external command, operable program or batch file``.

* Download the test samples `here <http://bioinformatics.cvr.ac.uk/DisCVR/Downloads/TestSamples.zip>`_


