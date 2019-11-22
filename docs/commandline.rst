DisCVR Command Line
===================

DisCVR can be used to classify multiple samples at once using the command line. In this case, a folder is generated containing the 
results for each sample in a separate .csv file. The results consist of information about the reads in the sample and the full analysis 
of matched *k*-mers. 
The jar called DisCVR\_CL.jar is used to run classifications on more than one file. Refer to the Installation section to download and install the jar and ensure that it is on the same path as *DisCVR.jar* and the *lib* folder. 
The following command can be used to run DisCVR_CL.jar to classify multiple files.

.. code-block:: bash

    java –jar full/path/to/DisCVR_CL.jar <samples folder> <k> <file format> <Database name> <database option> <entropy threshold>

<samples folder>: is the full path to the folder which contains sample files to be classified.  
<k>: is the *k*-mer size.  If using the DisCVR built-in database, use 22
<file format>: is the format of the sample files e.g. fastq  
<database name>: is the name of the database to be used in the classification. If DisCVR’s built-in database is to be used then use *HaemorrhagicVirusDB_22*, *RespiratoryVirusDB_22*, or *HSEVirusDB_22*. In case of a customisedDB, the full path to the database file must be provided.  
<database option>: if it is one of DisCVR’s database then use *BuiltInDB* and if it is a customised database file then use *customisedDB*.  
<entropy threshold>: is the threshold to use to remove low- entropy *k*-mers. If using DisCVR’s built-in database, for consistency use 2.5.

Using the command line, DisCVR enables the analysis of multiple clinical samples, however 
validation of the results via reference genome alignment is not available at this stage.

