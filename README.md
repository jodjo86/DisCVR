# Exemple du workflow actuel pour la detection de virus
**Projet 400128- SHD Virus
2021-02-19**

Voici le procédure pour mettre en place un environnement pour faire la détection de virus ainsi qu'un exemple de détection de souche de PVY dans des pommes de terre.

## Mise en place

### Prérequis

* Conda
    J'ai pris pour aquis que conda est déjà installé.
    ```
    conda create --name exempleSHD
    conda activate exempleSHD
    ```
* Java
    ```
    sudo apt install default-jre
    java -version 
    ```
    return: "openjdk version 11.X.X"

* NCBI tools
    ```
    1. SRA toolkit
    wget --output-document sratoolkit.tar.gz \
        http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
    tar -vxzf sratoolkit.tar.gz
    export PATH=$PATH:$PWD/sratoolkit.2.10.9-ubuntu64/bin
    
    2. esearch
    sudo apt install ncbi-entrez-direct
    esearch 
    ```
    return: "Must supply -db database on command line"

* Apache Ant
    ```
    sudo apt install ant
	ant -version 
    ```
    return: "Apache Ant(TM) version 1.10.6 compiled on May 29 2020"

* FastQ Screen (optionel)
    Est optionel mais permet de réduire considérablement la taille des fichiers fastq et le temps d'analyse.
    ```
    conda install -c bioconda fastq-screen
    ```

### Installation de DisCVR

DisCVR est l'application centrale pour la détection des virus et est disponible sous licence licence publique générale GNU. 
* Code source: [https://github.com/centre-for-virus-research/DisCVR](https://github.com/centre-for-virus-research/DisCVR) 
* Site web: [http://bioinformatics.cvr.ac.uk/discvr.php](http://bioinformatics.cvr.ac.uk/discvr.php)
* Développé par [Maha Maabar](https://github.com/MahaMaabar)

Cet exemple a été fait dans un dossier nommé "exempleSHD". À la fin de l'installation, la structure de fichier devrait être semblable à ceci.
```
.
└── exempleSHD/
    ├── DisCVRApp/
    │   ├── bin/
    │   ├── lib/
    │   ├── customisedDB/
    │   ├── host/
    │   ├── importdb/
    │   ├── DisCVR_CL.jar
    │   └── downloadDataAndRefSeq.sh
    ├── DisCVRSource/
    │   └── ...
    └── PRJNA612026/
        └── XXX.fastq.gz
```
Description des dossiers:
* DisCVRApp: Application DisCVR
* customisedDB: Base de données virale finale
* host: Génome de l'hôte
* importdb: Création de la base de données virale
* DisCVRSource: Code source de DisCVR
* PRJNA612026: Exemple de résultats de séquençage

1. Téléchargement du code source de DisCVR à partir de Github
    ```
    mkdir DisCVRApp
    mkdir -p DisCVRApp/{bin,customisedDB,importdb,host}
    git clone https://github.com/jodjo86/DisCVR DisCVRSource
    cd DisCVRSource
    ```

2. Compilation avec apache ant
Simplement exécuter la commande "ant" dasn le dossier "DisCVRSource". La compilation est rapide et ne devrait causer d'erreur.
    ```
    ant 
    cd ..
    ```

3. Copie des fichiers dans l'applications
Construction de l'application après la compilation.
    ```
    cp -a ./DisCVRSource/dist/lib ./DisCVRApp/
    cp -a ./DisCVRSource/build/classes/utilities ./DisCVRApp/bin/
    cp -a ./DisCVRSource/build/classes/customdatabase ./DisCVRApp/bin/
    cp -a ./DisCVRSource/dist/DisCVR_CL.jar ./DisCVRApp/
    cp -a ./DisCVRSource/downloadDataAndRefSeq.sh ./DisCVRApp/
    ```

4. Récupération des info de taxonomie de NCBI
Les fichiers "names.dmp" et "nodes.dmp" doivent se retrouver dans le dossier "customisedDB".
    ```
    cd ./DisCVRApp/customisedDB
    wget ftp.ncbi.nlm.nih.gov/pub/taxonomy/new_taxdump/new_taxdump.tar.gz
    tar -xzvf new_taxdump.tar.gz names.dmp
    tar -xzvf new_taxdump.tar.gz nodes.dmp
    rm new_taxdump.tar.gz
    ```

5. Génome de référence - DisCVR
Pour cet exexple nous avons besoin du génome de la pomme de terre.
    ```
    cd ..
    wget -O ./host/genomePdt.fna.gz ftp.ncbi.nih.gov/genomes/refseq/plant/Solanum_tuberosum/latest_assembly_versions/GCF_000226075.1_SolTub_3.0/GCF_000226075.1_SolTub_3.0_genomic.fna.gz
    gzip -d ./host/genomePdt.fna.gz
    ```

6. Génome de référence - FastqScreen (optionel)
Cet étape est optionel et ne peut être réalisé seulement si FastQScreen a été installé. Il faut éditer le fichier "fastq_screen.conf" pour indiquer l'emplacement des fichiers généré par bowtie.
    ```
    bowtie2-build ./host/genomePdt.fna ./host/genomePdt
    rm ./host/genomePdt.fna
    wget -O ./host/fastq_screen.conf \ 
        https://raw.githubusercontent.com/jodjo86/files/main/fastq_screen.conf
    ```

7. Liste de virus
Pour cet exemple, la liste de variant de PVY est : PVY-N, PVY-O, PVY-NTN et PVY-NWi. Le format du sichier est le suivant: un variant/virus par ligne, commencer par le taxID de NCBI, suivi des no d'accession des séquences de références séparé par des vigule. Le taxID et les no d'acession sont séparé par une tabulation.
    ```
    mkdir ./importdb/dbPdt
    wget -O ./importdb/dbPdt/variantPVY.txt \ 
        https://raw.githubusercontent.com/jodjo86/files/main/variantPVY.txt
    ```

8. Exemple séquençage Pdt Minion
Pour cet exemple, nous allons utliser 4 séquençages fait sur MinION de pomme de terre avec un diagnostique PVY connu.

    source: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7232445/

    **Attention gros fichiers** 3 Go au total
    ```
    cd ..
    mkdir PRJNA612026

    fastq-dump --split-files --fastq --gzip SRR11431605
    mv SRR11431605_1.fastq.gz PRJNA612026/P221_MinION_RH.fastq.gz
    fastq-dump --split-files --fastq --gzip SRR11431609
    mv SRR11431609_1.fastq.gz PRJNA612026/P099_MinION_RH.fastq.gz
    fastq-dump --split-files --fastq --gzip SRR11431610
    mv SRR11431610_1.fastq.gz PRJNA612026/P097_MinION_RH.fastq.gz
    fastq-dump --split-files --fastq --gzip SRR11431611
    mv SRR11431611_1.fastq.gz PRJNA612026/P026_MinION_RH.fastq.gz
    ```

## Entrainement d'une BD de détection virale 
**exemple de détection de variant de PVY dans la pomme de terre**

1. Recherche des données Seq sur les variants.

```
cd DisCVRApp
bash downloadDataAndRefSeq.sh \
    ./importdb/dbPdt/variantPVY.txt \
    ./importdb/dbPdt/
```
```
bash downloadDataAndRefSeq.sh \
    <fichier liste des virus> \
    <dossier d'entrainement de la BD>
```

2. Transfert pour filtration
```
mkdir ./importdb/dbPdt/DataSeq_filtered
cp -r ./importdb/dbPdt/DataSeq/* ./importdb/dbPdt/DataSeq_filtered
```

3. Création du fichier de référence du génome des variants
```
java -cp ./bin  customdatabase.GenomesLibrary \
    ./importdb/dbPdt/variantPVY.txt \
    Pdt_variantPVY \
    ./importdb/dbPdt/RefSeq/
```
```
java -cp ./bin  customdatabase.GenomesLibrary \
    <fichier liste des virus> \
    <nom de la BD> \
    <dossier Seq de référence>
```

Le fichier "Pdt_variantPVY_referenceGenomesLibrary" ; doit être copié dans "/customisedDB"

```
cp ./importdb/dbPdt/RefSeq/Pdt_variantPVY_referenceGenomesLibrary ./customisedDB/
```

4. Préparation des fichiers nécessaires à l'entrainement de la BD
```
java -cp ./bin customdatabase.DataSequences \
    ./importdb/dbPdt/DataSeq_filtered \
    ./importdb/dbPdt/RefSeq/Pdt_variantPVY_referenceGenomesLibrary \
    Pdt_variantPVY
```
```
java -cp ./bin customdatabase.DataSequences \
    <dossier "DataSeq_filtered"> \
    <fichier NOM_BD_referenceGenomesLibrary> \ 
    <NOM_BD>
```
5. Entrainement de la BD par identification des k-mers uniques à chaque virus et abscent du génome de l'hôte.
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
```
java -cp ./bin customdatabase.KmersDatabaseBuild \
    <dossier "DataSeq_filtered"> \
    <fichier genome hôte> \
    <Nom BD entrainé> \
    <longueur des k-mers (minimum 18)> \
    <Nombre de threads (CPU)> \
    <Entropy (0 à 3)>\
    <nb fichier à traiter en même temps>
```

## Détection de variant de PVY dans un séquençages de pdt
```
java -jar /home/joel/exempleSHD/DisCVRApp/DisCVR_CL.jar \
	/home/joel/exempleSHD/PRJNA612026 \
	22 \
	fastq.gz \
	/home/joel/exempleSHD/DisCVRApp/customisedDB/Pdt_variantPVY_25_22 \
	customisedDB \
	2.5
```
```
java -jar /home/joel/exempleSHD/DisCVRApp/DisCVR_CL.jar \
	<dossier fichier à analyser> \
	<longueur des k-mers de la BD> \
	<type de fichier à analyser> \
	<fichier BD entrainé> \
	<BD entrainé, les BD préchargées ont été retirées de l'app> \
	<Entropy de la BD>
```

### Résultats
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

## Traitement optionel des fichier fastq

Comme dit précédamment, il est possible de réduire la taille des fichiers fastq et d'accélérer la détection virale. La figure 1 présente les différentes approche de prétraitement des fichiers fastq. La taille des fichiers est proportionnel au temps d'analyse de DisCVR. À noter, que tous les fichiers (fastq et fasta) sont toujours compressés sans perte avec gzip (fichier gz; les archives tgz ne sont pas compatible). De plus, chaque étape de réduction de la taille (Figure 1, en orange) entraine la perte d'information (qualité des séquences ou séquence du génome de l'hôte).

![alt text](https://github.com/jodjo86/files/blob/main/diagram.svg?raw=true)
*Figure 1. Diagrame du traitement des fichiers FastQ avec FastQ Screen et SeqTk (tous les fichiers sont compressés en gz).*

1. FastQ Screen 
```
fastq_screen --nohits --conf ./fastq_screen.conf XXX.fastq.gz
```

2. SeqTk
```
seqtk seq -a XXX.fastq.gz > XXX.fasta
gzip XXX.fasta
```

