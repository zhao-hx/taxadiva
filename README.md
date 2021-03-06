# TaxADivA

TaxADivA - TAXonomy Assignment and DIVersity Assessment

TaxADivA is a wrapper script written in Perl to facilitate the analysis of nifH amplicon sequences.

This script was published as part of the paper: http://aem.asm.org/content/84/4/e01512-17.long
(PubMed: https://www.ncbi.nlm.nih.gov/pubmed/29180374;
PDF: http://jordan.biology.gatech.edu/pubs/Gaby-AEM-2017.pdf)

The script uses threading to parallelize the processing of sequences and thereby reduce run time. This wrapper pipeline performs the following steps in order (including the optional steps):
1. Sequences are merged with PEAR (Zhang J et al 2014. Bioinformatics 30:614–620)
2. Primers are trimmed using PrinSeq
3. Chimeras are removed and sequences clustered with VSEARCH (Rognes T et al. 2016. PeerJ. 4:e2584)
4. Taxonomy is assigned with BLAST (Altschul SF. 1990. J Mol Biol. 215:403–410.) by reference to a nifH taxonomy database, cluster IV/V sequences are removed,
5. Numerous outputs for taxonomy exploration are produced including a BIOM text table (for QIIME; Caporaso JG et al. 2010. Nat Methods 7:335–336), Krona (Ondov BD et al. 2011. BMC Bioinformatics. 12:385.), STAMP (Parks DH et al. 2014. Bioinformatics. 30:3123-4.) and
6. An optional oligotyping analysis by Minimum Entropy Decomposition (Eren M et al. 2014. ISME J 9:968–979.) which produces taxonomically-labeled oligotype networks explorable with the network visualization tool Gephi (Bastian M et al. 2009. International AAAI Conference on Weblogs and Social Media.).

The taxonomy assignment step relies on similarity between the input clustered reads to the reference database.  Taxonomy of the reference database is transferred over to the clustered read based on a set of decision rules as shown in the figure below:

![Decision Chart Used in TaxADiva](/DecisionChart.png?raw=true "TaxADiva's decision chart for assigned OTU names and numbers")

The percent identity parameters described in the chart above are derived by emperically comparing all the sequences in the database and estimating the cutoffs that maximizes the number of correctly placed sequence within the same species, genus and family.  These parameters can be changed from inside the script.  These are declared as follows:

```
# These are emperically calculated threshold values
	my $family  = 75;
	my $genus   = 88.1;
	my $species = 91.9;
```

The code has been lightly commented and going forward, if time permits, I will add more comments to help anyone read, modify or update the script.  This README will be continuously updated, as required, going forward.

Citations: Gaby, J.C., Rishishwar, L., Valderrama-Aguirre, L., Green, S.J., Valderrama-Aguirre, A., Jordan, I.K., Kostka , J.E., 2017. Diazotroph community characterization via a high-throughput nifH amplicon sequencing and analysis pipeline.
Applied and Environmental Microbiology. 84: e01512-17.

## Dependencies

The script is written specifically for Linux operating system and has been tested on Ubuntu 14.04 and RedHat systems.  Certain components of the script may throw error on other *nix systems.  The script utilizes basic Linux commands and thus may work on Cygwin but not on MS-DOS.  All the required dependencies will hopefully not require further dependency installation and may simply require placement of respective binaries in a folder in the $PATH variable in the best case scenario.

- Perl (comes installed with Linux)
- nifH sequence database (*Comes with the script*, files: fDb.fasta and fTax.db.tsv; source: http://www.css.cornell.edu/faculty/buckley/nifh.htm ; 
Gaby JC, Buckley DH. 2013. A comprehensive aligned nifH gene database: a multipurpose tool for studies of nitrogen-fixing bacteria. Database. doi: 10.1093/database/bau001)
- PRINSEQ: http://prinseq.sourceforge.net/
  - Installation of the script simply requires placing the Perl script in a folder that is in the $PATH.  See below.
- Pear: http://sco.h-its.org/exelixis/web/software/pear/  
Zhang J, Kobert K, Flouri T, Stamatakis A. 2014. PEAR: a fast and accurate Illumina Paired-End reAd mergeR. Bioinformatics 30:614–620
  - Please note that the PEAR binaries may be on the bottom-right of the page!  Download the binaries, extract them and place them in a folder that is in the $PATH variable.  See below.
- VSEARCH: https://github.com/torognes/vsearch
Rognes T, Flouri T, Nichols B, Quince C, Mahé F. 2016. VSEARCH: a versatile open source tool for metagenomics. PeerJ 4:e2584.
  - The binary can then be placed in a folder that is in the $PATH variable.
- BLAST+: ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/
Altschul SF, Gish W, Miller W, Myers EW, Lipman DJ. 1990. Basic local alignment search tool. J Mol Biol 215:403–410.
  - Binaries for the relevant system can directly be obtained and placed in a folder that is in the $PATH variable.  See below.
- KRONA: https://github.com/marbl/Krona/wiki
Ondov BD, Bergman NH, Phillippy AM. 2011. Interactive metagenomic visualization in a Web browser. BMC Bioinformatics. 12:385. doi: 10.1186/1471-2105-12-385.
  - Download the archive, unzip it and place them in a folder that is in the $PATH variable.  See below.

For users not familiar with the $PATH variable, please follow the following steps to create your own bin directory and add it to your $PATH variable:
```
mkdir ~/bin
echo "export PATH=\$PATH:~/bin" >> ~/.bashrc
source ~/.bashrc
```
This will create a bin directory in your home folder (~) and add this folder to your PATH variable.  All the local installations can be placed inside this folder.


## Installations
Download the dependency, install them and place it in a folder that is in your PATH.  The script can then be run simply as ./taxadiva.pl

In case of an issue with installation, please contact Lava lavanyarishishwar@gmail.com

## General Usage
The argument and the type of argument expected are defined in the help below.
```
taxadiva.pl [-1 <forward read file>] [-2 <reverse read file>]
           [-o <STRING. output dir and PREFIX to store results. All results will be stored as PREFIX inside the directory PREFIX. Default: input filename>]
           [-d <STRING. Database. Default: db.fasta>] [-t <STRING. Taxonomy file. Default: tax.tsv>]
           [-p <INT. Primer length to be trimmed. Default: no trimming>]
           [-r <INT. Primer length to be trimmed from the RIGHT. Default: no trimming>]
           [-l <INT. Primer length to be trimmed from the LEFT. Default: no trimming>]
           [-k <FLAG. If specified, the trimmed primers will be retained as separate files.  Default: Don't retain trimmed primers.>]
           [-y <FLAG. Performs MED analysis>]
           [-j <INT. Number of threads. Default: 1>]
           [-g <INT. Depth cutoff for considering file. Default: 5000>]
           [-u <STRING. VSEARCH program path. Default: VSEARCH>]
           [--pear <STRING. PEAR paramaters in double quotes.  Default: "-v 50 -m 450 -n 350 -p 1.0 -j <threads>". Validity of the arguments not checked.>]
           [--med <STRING. Oligotyping paramaters in double quotes.  Default: "". Validity of the arguments not checked.>]
           [--med-metadata <STRING. MED metadata file to be used for the decompose command.  Specified by the -E option in the decompose command.  Default: decompose_map3.tab.>]
           [--keepc4 <FLAG. TaxADivA will not filter out cluster IV sequences>]
                   
           [-h <FLAG. Prints this help>]
           [-v <FLAG.  Prints the current version of the script.>]
           [--version <FLAG.  Prints the current version of the script.>]
```
```
taxadiva.pl [-s <file with set of forward and reverse files>]
           [-o <STRING. output dir and PREFIX to store results. All results will be stored as PREFIX inside the directory PREFIX. Default: input filename>]
           [-d <STRING. Database. Default: db.fasta>] [-t <STRING. Taxonomy file. Default: tax.tsv>]
           [-p <INT. Primer length to be trimmed. Default: no trimming>]
           [-r <INT. Primer length to be trimmed from the RIGHT. Default: no trimming>]
           [-l <INT. Primer length to be trimmed from the LEFT. Default: no trimming>]
           [-k <FLAG. If specified, the trimmed primers will be retained as separate files.  Default: Don't retain trimmed primers.>]
           [-y <FLAG. Performs MED analysis>]
           [-j <INT. Number of threads. Default: 1>]
           [-g <INT. Depth cutoff for considering file. Default: 5000>]
           [-u <STRING. VSEARCH program path. Default: VSEARCH>]
           [--pear <STRING. PEAR paramaters in double quotes.  Default: "-v 50 -m 450 -n 350 -p 1.0 -j <threads>". Validity of the arguments not checked.>]
           [--med <STRING. Oligotyping paramaters in double quotes.  Default: "". Validity of the arguments not checked.>]
           [--med-metadata <STRING. MED metadata file to be used for the decompose command.  Specified by the -E option in the decompose command.  Default: decompose_map3.tab.>]
           [--keepc4 <FLAG. TaxADivA will not filter out cluster IV sequences>]
                   
           [-h <FLAG. Prints this help>]
           [-v <FLAG.  Prints the current version of the script.>]
           [--version <FLAG.  Prints the current version of the script.>] 
```

Example usage: 
`taxadiva.pl -d fDb.fasta -t fTax.db.tsv -j 18 -s list.txt -o output1 -m "-v 50 -m 450 -n 350 -p 1.0 -j 4"`

The script can work on a single set of files (using the -1 and -2 option) or a set of files provided as a list as shown below:
```
#SampleName	ForwardRead	ReverseRead
FLD0006_S350	FLD0006_S350_L001_R1_001.fastq	FLD0006_S350_L001_R2_001.fastq
FLD0029_S42	FLD0029_S42_L001_R1_001.fastq	FLD0029_S42_L001_R2_001.fastq
FLD0052_S15	FLD0052_S15_L001_R1_001.fastq	FLD0052_S15_L001_R2_001.fastq
FLD0054_S351	FLD0054_S351_L001_R1_001.fastq	FLD0054_S351_L001_R2_001.fastq
FLD0063_S121	FLD0063_S121_L001_R1_001.fastq	FLD0063_S121_L001_R2_001.fastq
```

The columns needs to be **tab** separated.

## Parallelization
The script takes advantage of the embarrasingly parallel nature of the problem.  Basic parallelization is performed in the script using Perl threads (and threading whenever available inside dependencies).

By default, the script assumes that it can run 10 threads which may not be possible on many systems.  This can be changed using the -j command or from the beginning of the script where the variable is defined `my $threads = 10;` (in case the user want to permanently change the default for their machine).


## Known Issues
1.  One of the BLAST version changes from -max_hsps to -max_hsps_per_subject.  This will throw an error if there is a version conflict.  Lava is looking into this.

## Software that can be used for downstream analysis
- STAMP: Parks DH, Tyson GW, Hugenholtz P, Beiko RG. 2014. STAMP: statistical analysis of taxonomic and functional profiles. Bioinformatics. 30(21):3123-4.
- MED: Eren  a M, Morrison HG, Lescault PJ, Reveillaud J, Vineis JH, Sogin ML. 2014. Minimum entropy decomposition: Unsupervised oligotyping for sensitive partitioning of high-throughput marker gene sequences. ISME J 9:968–979.
- QIIME: Caporaso JG, Kuczynski J, Stombaugh J, Bittinger K, Bushman FD, Costello EK, Fierer N, Peña AG, Goodrich JK, Gordon JI, Huttley GA, Kelley ST, Knights D, Koening JE, Ley RE, Lozupone CA, McDonald D, Muegge BD, Pirrung M, Reeder J, Sevinsky JR, Turnbaugh PJ, Walters WA, Widmann J, Yatsunenko T, Zaneveld J, Kinght R. 2010. QIIME allows analysis of high-throughput community sequencing data. Nat Methods 7:335–336
- Gephi: Bastian M, Heymann S, Jacomy M. 2009. Gephi: an open source software for exploring and manipulating networks. International AAAI Conference on Weblogs and Social Media.
- EMPeror: Vázquez-Baeza Y, Pirrung M, Gonzalez A, Knight R. 2013. EMPeror: a tool for visualizing high-throughput microbial community data. Gigascience. 2(1):16. doi: 10.1186/2047-217X-2-16.



## Version Updates
- 0.11 - Stable version.  Alpha tested with following procedure: sequence quality control (prinseq), read merging (PEAR), read clustering (VSEARCH), taxonomy assignment (BLAST + processing), KRONA plot generation, oligotyping (MED; optional).  MED is fully functional.  Cluster IV filtering is now optional.  Instead of creating multiple output files, creates an output directory and places all the output files in it.
- 0.10 - Stable version.  Alpha tested with following procedure: sequence quality control (prinseq), read merging (PEAR), read clustering (USEARCH), taxonomy assignment (BLAST + processing), KRONA plot generation, oligotyping (MED; optional).  MED is fully functional.  Instead of creating multiple output files, creates an output directory and places all the output files in it.
- 0.9 - Last stable version.  Alpha tested the following procedure: sequence quality control (prinseq), read merging (PEAR), read clustering (USEARCH), taxonomy assignment (BLAST + processing), KRONA plot generation, oligotyping (MED; optional).
