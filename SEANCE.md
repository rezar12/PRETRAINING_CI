
*Ce guide est une version revisitée du workshop **wrangling-genomics**. Voir la référence pour le lien.*

![image]("https://github.com/rezar12/PRETRAINING_CI/blob/main/Pasted%20image%2020241116173846.png?raw=true")

### Prérequis

**Ma configuration : je dispose d'une machine :**

- Core i3
- 16 Go de RAM
- 1 To d'espace de stockage (~ 50 Go suffisent largement)

**Amorçage**:

Mise à jour :

``` bash
sudo apt update
```

Installation de **wget** :

``` bash
sudo apt install wget
```

Création d'un répertoire d'analyse :

```bash
mkdir Analyse && cd Analyse
```

**Séquences :**

- Référence (.fasta.gz) :  
    Après notre déconvenue avec les séquences de _Plasmodium falciparum (qui nécessitent plus de ressources RAM et CPU), j'opte pour une espèce plus digeste, si je peux m'exprimer ainsi :  Escherichia coli_ (E. coli).

> [!WARNING]
> ESPACE UTILISE
> **Le téléchargement des ressources en termes d'espace disque (~ 600 Mo).**

Création du dossier pour le stockage de la référence :

```bash
mkdir reference && cd reference
```

Téléchargement de la référence depuis le NCBI via wget :

``` bash
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/017/985/GCA_000017985.1_ASM1798v1/GCA_000017985.1_ASM1798v1_genomic.fna.gz -O e_coli.fasta.gz
```

Décompression :

```bash
gunzip e_coli.fasta.gz
```


Capture de la sortie :

![[Pasted image 20241116183921.png]]

-  Reads (.fastq.gz)  
	Lien vers ENA browser : [lien vers ENA](https://www.ebi.ac.uk/ena/browser/view/SAMN04096081?show=reads)

Sortez du répertoire **reference** et créez un répertoire **raws**. Puis, téléchargez les séquences.

```bash
cd .. && mkdir raws && cd raws
```

```bash
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/004/SRR2589044/SRR2589044_1.fastq.gz

wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/004/SRR2589044/SRR2589044_2.fastq.gz

wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/001/SRR2584861/SRR2584861_1.fastq.gz

wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/001/SRR2584861/SRR2584861_2.fastq.gz

```

Capture de la sortie :

![[Pasted image 20241117054012.png]]


### Miniconda et installation de packages

Lien vers Miniconda [lien miniconda](https://docs.anaconda.com/miniconda/)

Installation de miniconda :

```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```

Capture de la sortie :

![[Pasted image 20241117063630.png]]

Activation de l'environnement :

```bash
source ~/miniconda3/bin/activate
```

Capture de la sortie :

![[Pasted image 20241117064032.png]]

Ajout de conda à l'ensemble de vos shells :

```bash
conda init --all
```

Capture de la sortie :

![[Pasted image 20241117064929.png]]

Création d'un environnement pour nos analyses, autre que le base. 

Mon environnement de travail pour les analyses du jour s'appellera **analyse** :

```bash
conda create -n analyse -y
```

Capture de la sortie :

![[Pasted image 20241117071335.png]]

Activons notre environnement (**analyse**) :

```bash
conda activate analyse
```

Capture de la sortie :
	Comme vous pouvez le remarquer, on passe de l'environnement (**base**) à l'environnement (**analyse**).
	
![[Pasted image 20241117071518.png]]

Ajout de channels (banque de packages) supplémentaire:

```bash
conda config --add channels defaults 
conda config --add channels bioconda 
conda config --add channels conda-forge
```

Capture de la sortie :

![[Pasted image 20241117072258.png]]

Installation des packages 

> [!WARNING] ESPACE UTILISÉ
> Sachez que l'installation de l'ensemble des outils vous coûtera en termes d'espace disque (~ 457,7 Mo).

```bash
conda install fastqc multiqc trimmomatic bwa samtools freebayes bcftools -y
```

Capture de la sortie :

![[Pasted image 20241117075239.png]]



Vérification via **grep** que mes packages sont installés :

```bash
conda list | grep -E "multiqc|fastqc|samtools|freebayes|trimmomatic|bcftools"
```

Capture de la sortie :

![[Pasted image 20241117080842.png]]

Pour en savoir plus sur conda, vous pouvez consulter le cheatsheet disponible sur le lien suivant  : [Cheatsheet](https://docs.conda.io/projects/conda/en/latest/_downloads/e0795803d81d6d87b74abed339025237/conda-24.4.0.pdf)

### Contrôle Qualité (QC)

**Récapitulatif :**  
Nous disposons de deux sous-dossiers dans notre répertoire **analyse** : **raws** et **reference**.

Capture (commande **tree**) :

![[Pasted image 20241117084838.png]]

> [!ERROR]
> Si tree non installé,  exécutez `sudo apt install tree` 
##### Visualisation (FastQC / MultiQC) :

Création du répertoire **result_fastqc** :

```bash
mkdir result_fastqc
```

Capture de la sortie :
*Le dossier **result_fastqc** est créé, le dossier est surligné en vert.*

![[Pasted image 20241117090120.png]]

Exécution de **FastQC** pour la visualisation de la qualité :

```bash
fastqc raws/*.gz -o result_fastqc/
```

Explication des arguments :

- **`raws/*.gz`** :
    - Indique que tous les fichiers compressés (`*.gz`) situés dans le dossier `raws/` doivent être analysés.
- **`-o result_fastqc/`** :
    - Spécifie le dossier de sortie (`result_fastqc/`) où les rapports générés seront enregistrés.

Capture de la sortie :
- Pour chaque fichier **`fastq.gz`** fastqc génère :
	- Un rapport HTML (consultable dans un navigateur).
	- Un fichier ZIP contenant des données détaillées.

![[Pasted image 20241117094225.png]]

Générons une sortie globale via **MultiQC** :

```bash
multiqc result_fastqc/
```

Explication des arguments:

- **`result_fastqc/`** :
	- Spécifie le répertoire où les rapports **FastQC** générés seront enregistrés.
	- À noter : **MultiQC** ne réalise pas d'analyses des fichiers **FastQC**. Il compile plutôt les résultats pour fournir une vue d'ensemble des rapports **FastQC**.

Capture de la sortie :

- Exécution de **MultiQC** :
	- Génère un fichier HTML **`multiqc_report.html`** et un dossier **`multiqc_data`**.`**

![[Pasted image 20241117092209.png]]
![[Pasted image 20241117093615.png]]

Consultation des rapports FastQC et MultiQC :

Vous pouvez consulter les rapports directement dans votre navigateur en ouvrant le fichier HTML du **read** à consulter pour **FastQC**, ou un visuel global en utilisant **`multiqc_report.html`**.

> [!NOTE]
> Pour ceux qui ont des difficultés à comprendre les différentes sections du rapport généré par **FastQC**, vous pouvez consulter le lien suivant [Babraham FastQC aide](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/). Il y est expliqué en détail chaque composant du rapport, les causes possibles des erreurs, et bien d'autres informations utiles.

Capture FastQC (pour `SRR2584861_1.fastq.gz`) :

![[Pasted image 20241117095704.png]]


Capture MultiQC :

![[Pasted image 20241117100021.png]]

> [!NOTE] **Résumer :**
> 
> la visualisation des reads est une étape clé pour vérifier la qualité de nos données avant de passer à la suite. Grâce à ses rapports, on peut repérer les problèmes comme les bases de mauvaise qualité ou la présence d’adaptateurs. Ces infos sont super utiles pour préparer les étapes suivantes, comme le **trimming**, où on nettoie les reads pour enlever tout ce qui pourrait gêner l’analyse.

### Trimming ( Trimmomatic )

L’amélioration de la qualité des séquences passe souvent par la suppression des portions de reads dont le score Phred est inférieur à 20 ou par l’élimination des adaptateurs présents dans les séquences.

Création d'un dossier pour nos résultats :

```bash
mkdir result_trimming
```

Capture de la sortie :
![[Pasted image 20241117113651.png]]

Présence d'adaptateur dans nos séquences qui sont de types `Nextera`.

Capture de la sortie :

![[Pasted image 20241117114132.png]]

Création d'un fichier pour les séquences d'adaptateurs :

```bash
nano NexteraPE-PE.fa
```

> [!ERROR]
> si nano non installé,  exécutez `sudo apt install nano` 

Collez **le contenu copié** ci-dessous dans l'éditeur nano. Puis  grâce au combinaison de touches <kbd>Ctrl</kbd> + <kbd>X</kbd>  suivis de la touche <kbd>Y</kbd> pour  "yes" .
Appuyez sur **`Enter`** (touche Entrée) pour confirmer le nom du fichier et le sauvegarder.

```text
>PrefixNX/1
AGATGTGTATAAGAGACAG
>PrefixNX/2
AGATGTGTATAAGAGACAG
>Trans1
TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG
>Trans1_rc
CTGTCTCTTATACACATCTGACGCTGCCGACGA
>Trans2
GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG
>Trans2_rc
CTGTCTCTTATACACATCTCCGAGCCCACGAGAC
```

Capture **collez le texte** :

![[Pasted image 20241117120704.png]]

Capture **enregistrement** :

![[Pasted image 20241117120859.png]]

capture **valider l'enregistrement avec la touche `enter`**:

![[Pasted image 20241117120943.png]]


> [!SUCCESS]  Visualisation du contenue
> exécutez la commande `cat NexteraPE.fa`

Capture de la sortie :

![[Pasted image 20241117121413.png]]

Exécution de trimmomatic :

Pour faciliter la tâche, nous exécuterons la commande directement dans le répertoire `raws`. Déplaçons donc la séquence de l'adaptateur dans ce répertoire :

```bash 
mv NexteraPE-PE.fa raws/
cd raws
ls -l
```

Capture de la sortie :

![[Pasted image 20241117123417.png]]

Exécutons Trimmomatic sur les séquences :

- Pour `SRR2589044`

```bash
trimmomatic PE SRR2589044_1.fastq.gz SRR2589044_2.fastq.gz \
SRR2589044_1.trim.fastq.gz SRR2589044_1un.trim.fastq.gz \
SRR2589044_2.trim.fastq.gz SRR2589044_2un.trim.fastq.gz \
SLIDINGWINDOW:4:20 MINLEN:25 HEADCROP:15 ILLUMINACLIP:NexteraPE-PE.fa:2:40:15
```

- Pour `SRR2584861`

```bash
trimmomatic PE  SRR2584861_1.fastq.gz SRR2584861_2.fastq.gz \
SRR2584861_1.trim.fastq.gz SRR2584861_1un.trim.fastq.gz \
SRR2584861_2.trim.fastq.gz SRR2584861_2un.trim.fastq.gz \
SLIDINGWINDOW:4:20 MINLEN:25 HEADCROP:15 ILLUMINACLIP:NexteraPE-PE.fa:2:40:15
```


> [!SUCCESS] Commande parfaitement exécutée
>Lorsque la commande s'exécute dans de bonnes conditions, vous obtiendrez en dernière ligne ce message :
>
>![[Pasted image 20241117124500.png]]
>

Explication des arguments :
- **`PE`** :
	- **`PE` (Paired-End)** : Indique que les reads d'entrée sont en format paire ( 1 et 2).
- **`Fichiers d'entrer`** :
	- **`SRR2589044_1.fastq.gz`** et **`SRR2589044_2.fastq.gz`** les fichiers `fastq.gz` brute.
-  **`Fichiers de sortir`**:
	- Pour chaque reads forward et reverse une sortir appariés `trim.fastq.gz` et non appariés `un.trim.fastq.gz` est générées.    
- **`SLIDINGWINDOW:4:20`**
	- Active un trimming par fenêtre glissante :
	    - **`4`** : Longueur de la fenêtre (en bases).
	    - **`20`** : Score de qualité seuil (Phred). Si la qualité moyenne dans une fenêtre descend en dessous de 20, la lecture est coupée à cet endroit.
- **`MINLEN:25`**
	- Supprime complètement un read si sa longueur tombe en dessous de **25 bases** après le trimming.
- **`HEADCROP:15`**  
	- Est utilisé pour couper un nombre de **15 bases** **au début** de chaque read, quelle que soit leur qualité.
- **`ILLUMINACLIP:NexteraPE-PE.fa:2:40:15`** :
	- Retire les séquences d'adaptateurs `Nextera` présentes dans les reads.
	- **`NexteraPE-PE.fa`** : Fichier contenant les séquences des adaptateurs.
	- **`2`** : Nombre maximal de mismatches autorisés entre l’adaptateur et la séquence.
	- **`40`** : Score seuil pour considérer qu’une correspondance avec un adaptateur est significative.
	- **`15`** : Longueur minimale de la correspondance avec l’adaptateur.


Visualisation des fichiers du trimming :

```bash
ls -l | grep "trim"
```

Capture de la sortie:

![[Pasted image 20241117130136.png]]

Déplaçons les fichiers du trimming dans le répertoire `result_trimming` :

```bash 
mv *.trim* ../result_trimming/ 
cd ../result_trimming/
ls -l
```

Capture de la sortie :

![[Pasted image 20241117131140.png]]

Créons deux sous-dossiers pour la séparation des reads appariés et non appariés :

```bash
mkdir {trim,untrim}
```

Capture de la sortie :

![[Pasted image 20241117131542.png]]

```bash
mv *un.trim* untrim/ && mv *.trim* trim/
```

Capture de la sortie :

![[Pasted image 20241117131915.png]]

> [!NOTE] Pour plus de commodité
> Pour la suite de nos analyses, nous utiliserons les reads appariés `trim`. Pour cela, déplaçons le dossier `trim` vers le répertoire `analyse`.
> ```bash
> mv trim/ ..  && cd ..
> ```

Capture de la sortie :

![[Pasted image 20241117133032.png]]


> [!INFO] Visualisation des fichiers du trimming
> Un nouveau contrôle de qualité est indispensable après le trimming. Cette étape permet de vérifier si le nettoyage a correctement éliminé les bases de faible qualité et les adaptateurs, tout en conservant des reads suffisamment longs et utilisables pour les analyses suivantes.


> [!INFO] EXERCICE
> **La compilation des nouveaux rapports pour les fichiers issus du trimming est laissée en exercice. **

Capture de la sortie :

Avant SRR2584861_2 :

![[Pasted image 20241117134217.png]]

Après trimming SRR2584861_2 :

![[Pasted image 20241117134237.png]]

Avant SRR2589044_1 :

![[Pasted image 20241117134415.png]]

Après trimming SRR2589044_1:

![[Pasted image 20241117134442.png]]

*Avant Multiqc* 
![[fastqc-status-check-heatmap.png]]
*Après  Trimming Multiqc*
![[fastqc-status-check-heatmap (2).png]]



> [!NOTE] **Résumer :**
> 
> A comparer les captures des résultats avant et après le trimming met clairement en évidence une amélioration de la qualité des séquences. En effet, le trimming permet d’éliminer les bases de faible qualité, les adaptateurs résiduels augmentant ainsi  le score de qualité reads.

### Mapping (BWA & SAMTOOLS)

Le **mapping**, ou alignement, est une étape clé en bio-informatique qui consiste à aligner les reads issus du séquençage sur un génome de référence.

#### indexation de la reference 

Indexer une référence a pour but permettre un alignement rapide et efficace des reads.
rendez-vous dans le répertoire de la reference puis exécuter la commande ci-dessous : 

```bash
cd reference
bwa index e_coli.fasta
```

> [!INFO] ALTERNATIVE
> **SAMTOOLS**, via sa sous-commande `faidx`, permet aussi d'indexer une référence.


Explication des arguments :
- **`index e_coli.fasta`**
	- Permet d'indexer la  reference  *e_coli.fasta* produisant ainsi les fichiers évoqués dans le tableau ci-dessous.

capture indexation :

![[Pasted image 20241117174754.png]]

capture contenue du dossier  reference:

![[Pasted image 20241117174935.png]]

---
Tableau descriptif des fichiers générés après l'indexation :

| **Fichier**        | **Description**                                                                                         | **Utilité**                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `e_coli.fasta.amb` | Contient les bases ambiguës (comme N ou d'autres codes IUPAC) dans la séquence de référence.            | Gère les régions où les bases ne sont pas spécifiées de manière unique.                                 |
| `e_coli.fasta.ann` | Contient les métadonnées sur la référence, comme les noms des contigs/chromosomes et leur position.     | Fournit des informations de contexte pour la séquence de référence.                                     |
| `e_coli.fasta.bwt` | Stocke la transformation Burrows-Wheeler de la séquence de référence.                                   | Accélère la recherche d’appariements grâce à cette structure de données compacte.                       |
| `e_coli.fasta.pac` | Représentation compressée de la séquence de référence au format binaire (chaque base codée sur 2 bits). | Réduit la taille des données et facilite leur manipulation.                                             |
| `e_coli.fasta.sa`  | Contient l'array des suffixes pour la séquence de référence.                                            | Permet de retrouver rapidement les positions d’une sous-séquence, accélérant le processus d’alignement. |

#### Alignement  (SAM / BAM)

**Dans la section alignement, deux répertoires entrent en jeu :**
- Le répertoire des fichiers du trimming
- Le répertoire de la référence indexée

![[Pasted image 20241117180451.png]]

**Génération du fichier SAM**  
Créons un dossier **SAM** pour le stockage de nos fichiers SAM. Puis, pour chaque fichier Paire-End du trimming `fastq.gz`, générons son fichier `.sam` via `bwa mem`.

```bash
mkdir SAM
```

- SRR2584861 

```bash
bwa mem -M -R "@RG\tID:SRR2584861\tSM:SRR2584861\tLB:SRR2584861\tPL:ILLUMINA" reference/e_coli.fasta trim/SRR2584861_1.trim.fastq.gz trim/SRR2584861_2.trim.fastq.gz -o SAM/SRR2584861.sam
```

- SRR2589044

```bash
bwa mem -M -R "@RG\tID:SRR2589044\tSM:SRR2589044\tLB:SRR2589044\tPL:ILLUMINA" reference/e_coli.fasta trim/SRR2589044_1.trim.fastq.gz trim/SRR2589044_2.trim.fastq.gz -o SAM/SRR2589044.sam
```

Explication des arguments:

-  **`bwa mem`**
    - Lance l'outil BWA avec l'algorithme **MEM** (Maximal Exact Match), adapté pour aligner des reads longs et courts.

-  **`-M`**
    - Marque les reads alignés secondaires (c’est-à-dire les reads pouvant s'aligner sur plusieurs endroits du génome) comme des alignements secondaires dans le fichier de sortie.
    - ==Cet argument est souvent utilisé pour une compatibilité avec d’autres outils, comme **Picard**.==

-  **`-R "@RG\tID:SRR2589044\tSM:SRR2589044\tLB:SRR2589044\tPL:ILLUMINA"`**
    - Définit un groupe de reads (`Read Group`) avec les informations suivantes :
        - **`ID`** : Identifiant unique du groupe de reads, ici `SRR2589044`.
        - **`SM`** : Nom de l’échantillon, ici `SRR2589044`.
        - **`LB`** : Identifiant de la bibliothèque, ici `SRR2589044`.
        - **`PL`** : Plateforme de séquençage utilisée, ici `ILLUMINA`.

```bash
cat SAM/SRR2584861.sam | head -n 4 | grep '@RG'
```

Capture de la sortie 

![[Pasted image 20241117204917.png]]

- **`reference/e_coli.fasta`**
    - Chemin vers la séquence de référence indexée (le génome sur lequel les reads seront alignés).

- **`trim/SRR2589044_1.trim.fastq.gz` et `trim/SRR2589044_2.trim.fastq.gz`**
    - Fichiers `fastq.gz` des reads **après trimming** :
        - **`SRR2589044_1.trim.fastq.gz`**
        - **`SRR2589044_2.trim.fastq.gz`**

- **`-o SAM/SRR2589044.sam`**
    - Spécifie le fichier de sortie au format **SAM**, ici `SAM/SRR2589044.sam`. Ce fichier contiendra les alignements des reads sur la référence.

Capture du répertoire SAM :

![[Pasted image 20241117182526.png]]

**Génération du fichier BAM**  
Créons un dossier **BAM** pour le stockage de nos fichiers BAM. Puis, pour chaque fichier SAM, générons son fichier BAM.
 
```bash
mkdir BAM
```

- SRR2584861 

```bash
samtools view -S -b SAM/SRR2584861.sam -o BAM/SRR2584861.bam
samtools sort -n -o BAM/SRR2584861_sorted.bam BAM/SRR2584861.bam
samtools fixmate -m BAM/SRR2584861_sorted.bam BAM/SRR2584861_fixed.bam
samtools sort -o BAM/SRR2584861_fixedsorted.bam BAM/SRR2584861_fixed.bam
samtools markdup BAM/SRR2584861_fixedsorted.bam  BAM/SRR2584861_dedup.bam
samtools index BAM/SRR2584861_dedup.bam
```

- SRR2589044

```bash
samtools view -S -b SAM/SRR2589044.sam -o BAM/SRR2589044.bam
samtools sort -n -o BAM/SRR2589044_sorted.bam BAM/SRR2589044.bam
samtools fixmate -m BAM/SRR2589044_sorted.bam BAM/SRR2589044_fixed.bam
samtools sort -o BAM/SRR2589044_fixedsorted.bam BAM/SRR2589044_fixed.bam
samtools markdup BAM/SRR2589044_fixedsorted.bam  BAM/SRR2589044_dedup.bam
samtools index BAM/SRR2589044_dedup.bam
```

Explication des commandes:
- **`samtools view -S -b SAM/SRR2589044.sam -o BAM/SRR2589044.bam`**
    - Convertit le fichier d'alignement **SAM** (texte) en format **BAM** (binaire) pour des manipulations plus rapides.

- **`samtools sort -n -o BAM/SRR2589044_sorted.bam BAM/SRR2589044.bam`**
    - Trie le fichier BAM par nom des reads (nécessaire pour traiter les reads appariés).

- **`samtools fixmate -m BAM/SRR2589044_sorted.bam BAM/SRR2589044_fixed.bam`**
    - Corrige et enrichit les métadonnées des reads appariés (par exemple, ajoute des informations sur les paires).

- **`samtools sort -o BAM/SRR2589044_fixedsorted.bam BAM/SRR2589044_fixed.bam`**
    - Trie à nouveau le fichier BAM, cette fois par position des reads dans le génome (préparation pour la détection des duplications).

- **`samtools markdup BAM/SRR2589044_fixedsorted.bam BAM/SRR2589044_dedup.bam`**
    - Identifie et marque les reads dupliqués.

- **`samtools index BAM/SRR2589044_dedup.bam`**
	- Indexer le fichier.

capture du répertoire BAM :

![[Pasted image 20241117224920.png]]

> [!NOTE] **Résumer :**
> 
> Le mapping est une étape essentielle pour aligner les reads sur une séquence de référence. Grâce à des outils comme BWA et SAMTOOLS, on peut non seulement aligner les séquences,les trier,et éliminer les duplications. Facilitant l’appel de variants ou l’étude des mutations. 

> [!INFO] Usage de Picard
> Il est important de noter qu’en lieu et place des commandes SAMTOOLS pour le tri et le marquage des duplicats, nous aurions pu utiliser les outils proposés par **Picard**, tels que **`SortSam`** pour le tri et **`MarkDuplicates`** pour le marquage des duplicats.

### Variants Calling (Freebayes):

Le variant calling est une étape clé en bio-informatique qui consiste à identifier les variations génétiques (SNPs, indels, etc.) présentes dans un échantillon par rapport à une séquence de référence.

Génération des fichiers **VCF** _(Variant Call Format)_

Créons un dossier **Variant** pour le stockage de nos fichiers VCF :

```bash
mkdir variant
```

Exécutons **Freebayes** pour la génération de fichiers VCF pour chacune des séquences :

- SRR2584861

```bash
freebayes --ploidy 1 -f reference/e_coli.fasta BAM/SRR2584861_dedup.bam > variant/SRR2584861.vcf
```


- SRR2589044

```bash
freebayes --ploidy 1 -f reference/e_coli.fasta BAM/SRR2589044_dedup.bam > variant/SRR2589044.vcf
```

Création d'un VCF **merge** (concaténation dans un seul fichier) :

- Créons un fichier contenant les noms de nos fichiers `dedup.bam`.

```bash
ll BAM/*_dedup.bam | awk '{print $NF}' > bam_list.txt
cat bam_list.txt 
```

Capture de la sortie :

![[Pasted image 20241117234525.png]]

- Génération du fichier "merge" 

```bash
freebayes --ploidy 1 -f reference/e_coli.fasta  --bam-list bam_list.txt > variant/merge.vcf
```

Explication des arguments  :

- **`--ploidy 1`** :
    - Spécifie que le génome à analyser est **haploïde** (une seule copie de chaque chromosome). 

- **`-f reference/e_coli.fasta`** :
    - Indique le fichier `fasta` contenant la séquence de référence du génome (ici, _E. coli_). 

- **`BAM/SRR2589044_dedup.bam`** :
    - Fichier BAM contenant les alignements des reads après suppression des duplicats (produit par `samtools markdup` ou un équivalent).

- **`> variant/SRR2589044.vcf`** :
    - Redirige la sortie vers un fichier **VCF**  


> [!NOTE] "MERGE"
> Dans le cas de la génération d'un fichier "merge", le fichier `.bam` est remplacé par l'argument suivant :
> -   **`--bam-list bam_list.txt`**
> 	- Dans le cas d'une génération de fichier "merge" prend un fichier (`bam_list.tx`) contenant les noms  des fichiers `_dedup.bam`.
> - **`variant/merge.vcf`** 
> 	- Le nom de la sortie du merge.


Capture du répertoire `variant` :

![[Pasted image 20241117234807.png]]

Affichage de quelques lignes de nos fichiers VCF en ignorant les informations d'entête :

Pour le "merge" :

```bash
grep -v '^##' variant/merge.vcf | head -n 3
```

Capture de la sortie :

![[Pasted image 20241118073730.png]]

Pour le SRR2584861 :

```bash
grep -v '^##' variant/SRR2584861.vcf | head -n 3
```

Capture de la sortie :

![[Pasted image 20241118073810.png]]

Pour le SRR2589044 :

```bash
grep -v '^##' variant/SRR2589044.vcf | head -n 3
```

Capture de la sortie :
![[Pasted image 20241118073848.png]]

> [!INFO] REMARQUE
> Dans le fichier "merge", vous trouverez l'ensemble des noms d'accession des séquences utilisées pour sa création. Ce fichier **VCF merge** représente l'union des mutations présentes dans les différents fichiers VCF qui le composent. Cela permet de regrouper et de visualiser en un seul endroit toutes les variations génétiques identifiées à partir des différentes séquences analysées.

##### Initiation aux filtrage de variant (BCFTOOLS)

Le filtrage des variants dans un fichier VCF est une étape cruciale pour cibler des régions spécifiques ou sélectionner des variants en fonction de critères définis. Par exemple, il est possible d'extraire uniquement certaines colonnes du fichier VCF, de se concentrer sur des régions d'intérêt précises, ou encore de filtrer les variants selon des paramètres comme la profondeur de lecture, la qualité, ou leur type (SNPs, INDELs, etc.).

- Afficher quelques lignes, uniquement les colonnes chromosome, position, référence et alternative, d'un fichier VCF :

```bash
bcftools query -f '%CHROM  %POS  %REF  %ALT{0}\n' variant/merge.vcf | head -n 3
```

Capture de la sortie :

![[Pasted image 20241118082037.png]]

- Afficher quelques lignes des mutations de type SNPs d'un fichier VCF :

```bash
bcftools view -v snps variant/merge.vcf | grep -v '^##' | head -n 3
```

Capture de la sortie :

![[Pasted image 20241118083148.png]]

- Réciproquement des indels :

```bash
bcftools view -v snps variant/merge.vcf | grep -v '^##' | head -n 3
```

Capture de la sortie :

![[Pasted image 20241118083720.png]]

> [!INFO]  VCFFILTER ALTERNATIVE AUX OPTIONS DE FILTRE DE BCFTOOLS
> BCFTOOLS est un outil polyvalent et largement utilisé pour le variant calling, au même titre que Freebayes, GATK ou VARSCAN. Si vous recherchez une solution légère, rapide et efficace, VCFFILTER est une excellente option pour filtrer vos variants selon des critères précis.

---
Liste de quelques outils populaires pour le variant calling :

| **Outil**     | **Description**                                                                                                                                        | **Lien officiel**                                          |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| **FreeBayes** | Outil basé sur un modèle bayésien, idéal pour détecter des variants à partir de données NGS, notamment pour des échantillons polyploïdes ou haploïdes. | [FreeBayes GitHub](https://github.com/freebayes/freebayes) |
| **GATK**      | Suite complète pour le traitement des données NGS, avec des outils avancés comme HaplotypeCaller pour le variant calling.                              | [GATK](https://gatk.broadinstitute.org/)                   |
| **BCFTOOLS**  | Outil léger et rapide pour la manipulation et l'analyse des fichiers VCF/BCF, avec des fonctions intégrées de variant calling et filtrage.             | [BCFTOOLS](http://samtools.github.io/bcftools/)            |
| **VARSCAN**   | Adapté pour les données à faible couverture, cet outil détecte efficacement les SNPs et les Indels dans des données appariées ou non.                  | [VARSCAN](http://varscan.sourceforge.net/)                 |

***

> [!NOTE] **Résumer :**
> 
> Le variant calling permet d’identifier les variations génétiques au sein d’un génome étudié. Grâce à des outils comme Freebayes dans ce guide, nous avons pu détecter ces variations sous forme de fichiers VCF, exploitables pour des analyses ultérieures.


> [!INFO] Automatisation du workflow
> Lorsque vous travaillez avec un grand nombre de séquences, exécuter les commandes manuellement pour chaque fichier devient rapidement difficile. Dans de tels cas, l'automatisation devient essentielle. L'idée est de créer des scripts ou des workflows qui peuvent traiter un ensemble de données de manière répétitive , sans intervention manuelle à chaque étape.
> [initiation au workflow datacarpentry](https://datacarpentry.github.io/wrangling-genomics/05-automation.html)


---
### Référence :
[Pour plus de séquence sur E .coli sur  ENA  ](https://www.ebi.ac.uk/ena/browser/view/SRA303743?show=reads)
[datacarpentry wrangling-genomics](https://datacarpentry.github.io/wrangling-genomics/aio.html)
[FASTQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
[MultiQC usage](https://www.youtube.com/watch?v=qPbIlO_KWN0)
[Trimmomatic](https://github.com/usadellab/Trimmomatic)
[SAMTOOLS](https://www.htslib.org/)
[Freebayes](https://github.com/freebayes/freebayes)
[BCFTOOLS](https://samtools.github.io/bcftools/bcftools.html)

Si vous avez des suggestions n'hésitez pas à [m'envoyer un email](mailto:rechristofer@gmail.com).


