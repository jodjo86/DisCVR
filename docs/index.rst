.. DisCVR documentation master file, created by
   sphinx-quickstart on Wed May  8 11:31:09 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


Welcome to DisCVR's documentation!
==================================

DisCVR is a viral detection tool which allows the identification of known human viruses in clinical
samples from high-throughput sequencing (HTS) data. It uses the k-mers approach in which the
sample reads are decomposed into *k*\-mers and then matched against a virus k-mers database. The
built-in database is a list consists of all *k*\-mers (*k*\=22) that are not low-complexity and only found in
the viral genomes but not in the human genome. Each *k*-mer is assigned the taxonomic label of all
viral genomes that contain that *k*\-mer in the NCBI taxonomy tree. These assignments are made at the
species and strains taxonomic level.

DisCVR has a user-friendly Graphical User Interface (GUI) which runs the analysis on the sample and
shows the results interactively. It enables the visualisation of the coverage of the virus genomes found
in the sample in order to validate the significance of the results. In addition, DisCVR is a generic tool
which can be used with other non-human viruses by facilitating the build and use of customised kmers
database.

DisCVR is designed to run on machines with low processing capacity and small memory.

.. toctree::
   :maxdepth: 1
   :caption: Contents:
   
   installation
   gui
   classification
   validation
   commandline
   customdb
   faq


