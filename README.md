# Exemple du workflow actuel pour la detection de virus
**Projet 400128- SHD Virus
2021-02-19**

## Mise en place

### Prérequis

* Conda
    ```
    conda create --name exempleSHD
    conda activate exempleSHD
    ```
* Java
    ```
    sudo apt install default-jre
    java -version 
    ```
    return
    ```
    openjdk version 11.X.X”
    ```
* NCBI tools
    ```
    # SRA toolkit
    wget --output-document sratoolkit.tar.gz http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
    tar -vxzf sratoolkit.tar.gz
    export PATH=$PATH:$PWD/sratoolkit.2.10.9-ubuntu64/bin
    
    # esearch
    sudo apt install ncbi-entrez-direct
    esearch 
    ```
    return
    ```
    Must supply -db database on command line
    ```
* Apache Ant
    ```
    sudo apt install ant
	ant -version 
    ```
    return
    ```
    Apache Ant(TM) version 1.10.6 compiled on May 29 2020
    ```

### Installation
1. Téléchargement de DisCVR à partir de Github
```
mkdir DisCVRApp
mkdir -p DisCVRApp/{bin,customisedDB,importdb,host}
git clone https://github.com/jodjo86/DisCVR DisCVRSource
cd DisCVRSource
```

2. Compilation de l'application
```
ant 
cd ..
```

3. Copie des fichiers dasn l'applications
```
cp -a ./DisCVRSource/dist/lib ./DisCVRApp/
cp -a ./DisCVRSource/build/classes/utilities ./DisCVRApp/bin/
cp -a ./DisCVRSource/build/classes/customdatabase ./DisCVRApp/bin/
cp -a ./DisCVRSource/dist/DisCVR_CL.jar ./DisCVRApp/
cp -a ./DisCVRSource/downloadDataAndRefSeq.sh ./DisCVRApp/
```

4. Récupération des info de taxonomie de NCBI
```
cd ./DisCVRApp/customisedDB
wget ftp.ncbi.nlm.nih.gov/pub/taxonomy/new_taxdump/new_taxdump.tar.gz
tar -xzvf new_taxdump.tar.gz names.dmp
tar -xzvf new_taxdump.tar.gz nodes.dmp
rm new_taxdump.tar.gz
```

5. Génome de référence
Pomme de terre
```
cd ..
wget -O ./host/genomePdt.fna.gz \ ftp.ncbi.nih.gov/genomes/refseq/plant/Solanum_tuberosum/latest_assembly_versions/GCF_000226075.1_SolTub_3.0/GCF_000226075.1_SolTub_3.0_genomic.fna.gz
gzip -d ./host/genomePdt.fna.gz
```

6. Liste de virus
Variant de PVY
```
mkdir ./importdb/dbPdt
wget -O ./importdb/dbPdt/variantPVY.txt \ https://raw.githubusercontent.com/jodjo86/files/main/variantPVY.txt
```

7. Exemple séquençage Pdt Minion
**Attention gros fichiers**
source: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7232445/
```
cd ..
mkdir PRJNA612026

fastq-dump --split-files --fasta --gzip SRR11431605
mv SRR11431605_1.fasta.gz PRJNA612026/P221_MinION_RH.fasta.gz
fastq-dump --split-files --fasta --gzip SRR11431609
mv SRR11431609_1.fasta.gz PRJNA612026/P099_MinION_RH.fasta.gz
fastq-dump --split-files --fasta --gzip SRR11431610
mv SRR11431610_1.fasta.gz PRJNA612026/P097_MinION_RH.fasta.gz
fastq-dump --split-files --fasta --gzip SRR11431611
mv SRR11431611_1.fasta.gz PRJNA612026/P026_MinION_RH.fasta.gz
```

## DB de détection virale 
**exemple détection de variant de PVY dans la pomme de terre**

```
cd DisCVRApp
bash downloadDataAndRefSeq.sh \
	./importdb/dbPdt/variantPVY.txt \
	./importdb/dbPdt/
```

```
mkdir ./importdb/dbPdt/DataSeq_filtered
cp -r ./importdb/dbPdt/DataSeq/* ./importdb/dbPdt/DataSeq_filtered
```

```
java -cp ./bin  customdatabase.GenomesLibrary \
    ./importdb/dbPdt/variantPVY.txt \
    Pdt_variantPVY \
    ./importdb/dbPdt/RefSeq/
```
Génère le fichier "Pdt_variantPVY_referenceGenomesLibrary" ; doit être copié dans "/customisedDB"
```
cp /importdb/dbPdt/RefSeq/Pdt_variantPVY_referenceGenomesLibrary ./customisedDB/
```

```
java -cp ./bin customdatabase.DataSequences \
    ./importdb/dbPdt/DataSeq_filtered \
    ./importdb/dbPdt/RefSeq/Pdt_variantPVY_referenceGenomesLibrary \
    Pdt_variantPVY
```

```
java -cp ./bin customdatabase.KmersDatabaseBuild \
    ./importdb/dbPdt/DataSeq_filtered/ \
    ./host/genomePdt.fna \
    Pdt_variantPVY_25 \
    22 \
    7 \
    2.5 \
    10
```

# Détection de variant de PVY dans un séquençages de pdt
```
java -jar /home/joel/exempleSHD/DisCVRApp/DisCVR_CL.jar \
	/home/joel/exempleSHD/PRJNA612026 \
	22 \
	fasta.gz \
	/home/joel/exempleSHD/DisCVRApp/customisedDB/Pdt_variantPVY_25_22 \
	customisedDB \
	2.5
```
NUMBER_OF_DISTINCT_K-MERS
| Échan. | Diag. | PVY-N | PVY-O | PVY-NTN | PVY-NWi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| P221 | PVY mixed | **525** | **1365** | **1485** | **527** |
| P099 | PVY-NWi | 271 | 1147 | 655 | 589 |
| P097 | NTNa | 278 | 330 | **862** | 61 |
| P026 | PVY-O | 35 | **2817** | 464 | 138 |

TOTAL_COUNTS_K-MERS
| Échan. | Diag. | PVY-N | PVY-O | PVY-NTN | PVY-NWi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| P221 | PVY mixed | **6027** | **31001** | **68520** | **32702** |
| P099 | PVY-NWi | 1547 | 16901 | 18993 | **61889** |
| P097 | NTNa | 1408 | 1951 | **38495** | 246 |
| P026 | PVY-O | 201 | **349745** | 20515 | 933 |


README-Original
DisCVR is a user-friendly tool for detecting known viruses in clinical samples. 
It works by creating a viral database of _k_-mers (22 nucleotide sequences) from a set 
of known sequence. These _k_-mers are taxonomically labelled according to the viruses 
from which they originated. Each read in the sample is then screened for the presence 
of _k_-mers in the viral database, and a list of all viruses with _k_-mers found in the 
sample is shown. DisCVR accurately detects known human viruses from HTS data using 
computers with limited resources, and includes a graphical user interface to make the 
interpretation and validation of results easy. At present, DisCVR is a human viral 
diagnostic tool, but it can be readily extended to include non-viral human pathogens 
and pathogens of other hosts using the scripts provided to build customised databases. 


The source code for DisCVR can be downloaded from [https://github.com/centre-for-virus-research/DisCVR](https://github.com/centre-for-virus-research/DisCVR) 

Compiled versions with the built-in databases are available from [http://bioinformatics.cvr.ac.uk/discvr.php](http://bioinformatics.cvr.ac.uk/discvr.php)
The download includes information for installing and running the tool. 

For more information on how to run DisCVR, here is the [manual](https://centre-for-virus-research.github.io/DisCVR/)

DisCVR was developed by [Maha Maabar](https://github.com/MahaMaabar)

