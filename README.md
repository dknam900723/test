# EDScan-S
### EDgc SCAN - Screening
#### Detect Single Nucleotide Variantion(SNV) in a plasma sample using targeted sequence data.

## GETTING HELP
If your problem or question is still unsolved, feel free to post a question or submit a bug report on the e-mail(daekuen.nam@edgc.com, s.katiyar@edgc.com).

## QUICKSTART GUIDE
Obtain the required modules:  
`pip install numpy pandas matplotlib pysam`  
You may need to install pip first, and depending on preferences you may add sudo at the beginning of this command.

## REQUIREMENTS
EDScan-S was developed and tested using Python2.7. Using any other version may cause errors or faulty results.

FastQCv0.11.5   
bwa-0.7.16a     
samtools-1.8    
qualimap_v2.2   

The list of python packages required and tested versions as reported by `pip freeze`:  
numpy==1.16.4.1  
pandas==0.24.2  
matplotlib==2.2.2  
pysam==0.14.1  

### script run
To convert  (R1,R2).fastq.gz files to a .vcf file for analysis, use EDScan-S.py:  
`python EDScan-S.py -idir [input fastq.gz directory] -ibed [input target ROI.bed file] -iseq [input primer sequence file] -odir [output directory]`

The target ROI.bed file specified here is EDGC target ROI. You can take any .bed file here (assuming it fits in memory).    
There are some parameters to adjust trimming, quality filter and variants call.

### quality check results
The script create .log file. This file record each steps quality check.
1. FastQC: fastq read count
2. pTrim(primer trimming script) : after pTrim fastq read count & filtered read ratio
3. BWA : mapped read count & on/off target read ratio
4. ROI clipping : clipped read count
5. Qaulimap : GER
6. EddpClean(UMI based de-duplication script) : ExSL(Exclude Singleton) read count & (Include Singleton) read count
7. UMI distribution : Total/Single UMI count & ratio
8. RODC(Read Overlap Depth Count target ROI depth check script) : mean/target coverage depth count & ratio
9. ROQA(Read Overlap Quality Access SNV caller script) : called variants count 
10. Annovar : annotate SNV

### In general
To improve your results you probably want to change a few parameters. Most of the values used in EDScan-S can be altered using arguments. To find out what arguments can be passed into any script, try running it with -h as argument, for example:  
`python EDScan-S.py -h`  

### Multi-core
EDScan-S provide multi-core process basically.

### Description
    usage: EDScan-S.py -idir INPUT_PATH [-ibed INPUT_BED] [-iseq INPUT_PRIMER]
                       -odir OUT_PATH [-panel {archer,edgc,qiagen}]
                       [-mp1 MULTI_PROCESS1] [-mp2 MULTI_PROCESS2]
                       [-mr MINIMUM_READLENGTH] [-um UMI_MISMATCH]
                       [-ec EXCLUDE_CLUSTERS] [-ecc ERRORCORRECTION_CUTOFF]
                       [-td TARGET_DEPTH] [-pfdb PFDB_LOCATION] [-afpe {yes,no}]
                       [-fpc FALSEPOSITIVE_CUTOFF] [-gc GERMLINE_CUTOFF]
                       [-afc AF_CUTOFF] [-acc ALTCOUNT_CUTOFF] [-dc DEPTH_CUTOFF]
                       [-cpc COSMIC_PRIORITYCUTOFF] [-v] [-h]
    
    Liquid Biopsy pipeline multiple runner.
    
    Required arguments:
      -idir INPUT_PATH, --input-path INPUT_PATH
                            * Required: Please provide the PATH of input FASTQs file directory.
                            
      -ibed INPUT_BED, --input-bed INPUT_BED
                            * Required: Please provide the PATH of input BED file.
                            	ROI bed file: 4 column <chr> <start> <end> <Gene>
                            	Default) /Work/Cancer/LiquidBiopsy/EDScan-S/data/EDGC_v4/target/Reveal_ctDNA_EDGC_v4_dSA12302-v1.1_target_annotated.bed [EDGC Custom v4 Panel]
                            
      -iseq INPUT_PRIMER, --input-primer INPUT_PRIMER
                            * Required: Please provide the PATH of input Primer sequence file
                            	Default) /Work/Cancer/LiquidBiopsy/EDScan-S/data/EDGC_v4/primers/Reveal_ctDNA_EDGC_v4_dSA12302-v1_1_allprimers.txt
                            
      -odir OUT_PATH, --out-path OUT_PATH
                            * Required: Please provide the PATH of output directory.
    
    Optional arguments:
      -panel {archer,edgc,qiagen}, --input-panel {archer,edgc,qiagen}
                            Please provide the panel of input.(default:archer)
      -mp1 MULTI_PROCESS1, --multi-process1 MULTI_PROCESS1
                            Please provide the multiprocessing core for samples.(default:5)
      -mp2 MULTI_PROCESS2, --multi-process2 MULTI_PROCESS2
                            Please provide the multiprocessing core for scripts.(default:10)
      -mr MINIMUM_READLENGTH, --minimum-readlength MINIMUM_READLENGTH
                            Optional but important: Discard reads that are less than -mr parameter length, after all kind of trimming.
                            	Default: 15
                            	Note: This read lengtht is the processed read length means after removing 'UMI','SSE','Primer' sequences.
                            
      -um UMI_MISMATCH, --umi-mismatch UMI_MISMATCH
                            Please provide the number of mismatch allowed in UMI while Clustering.(default:1)
      -ec EXCLUDE_CLUSTERS, --exclude-clusters EXCLUDE_CLUSTERS
                            If user wish to exclude clusters from main cluster data that have UPTO 'n' number of reads.(default:1)
      -ecc ERRORCORRECTION_CUTOFF, --errorcorrection-cutoff ERRORCORRECTION_CUTOFF
                            Please provide the cutoff value of mismatch ratio.(default:0.8)
      -td TARGET_DEPTH, --target-depth TARGET_DEPTH
                            Please provide the target coverage depth.(default:3000)
      -pfdb PFDB_LOCATION, --pfdb-location PFDB_LOCATION
                            * Optional: Please provide path of a population allele frequency database.
                            	Default) /Work/BIO/refs/KRGDB/KRGDB_622+1100.txt
                            
      -afpe {yes,no}, --auto-filterpanelerrors {yes,no}
                            * Optional: exclusive to '-edb': Filter variants by automatically making a panel specific False Positive Database.
                            	Default) yes
                            
      -fpc FALSEPOSITIVE_CUTOFF, --falsepositive-cutoff FALSEPOSITIVE_CUTOFF
                            * Optional: Required but valid only if '-afpe' is 'yes' or -edb: Define the cut-off count for a variant to consider it in false positive database.
                            			A variant is false positive if found more than -fpc cutoff samples.
                            	Default) 3
                            
      -gc GERMLINE_CUTOFF, --germline-cutoff GERMLINE_CUTOFF
                            * Optional: Required but valid only if '-afpe' is 'yes' or -edb: Define the cut-off count for a variant to consider it in germline database.
                            			A variant is false positive if found more than -gc cutoff samples.
                            	Default) 3
                            
      -afc AF_CUTOFF, --af-cutoff AF_CUTOFF
                            * Optional: Allele Frequency based Cut-off: Please provide a cut-off of Allele Frequency below which all the variants will be filtered out (dropped).
                            	Default) 0.1
                            
      -acc ALTCOUNT_CUTOFF, --altcount-cutoff ALTCOUNT_CUTOFF
                            * Optional: Any variant that has euqls to or lower than this alt count, will be dropped.
                            	Default) 3
                            
      -dc DEPTH_CUTOFF, --depth-cutoff DEPTH_CUTOFF
                            * Optional: Turn on the Depth-Based filter by providing a cutoff value of depth below which all the variants will be filtered out.
                            	Default) 0
                            
      -cpc COSMIC_PRIORITYCUTOFF, --cosmic-prioritycutoff COSMIC_PRIORITYCUTOFF
                            Optional: Use this option to give cosmic count highest priority at the time of filteration.
                            		  If the cosmic count for a variable is equals or more than this cut-off, it will always be kept irrespective of any other filter (even False Positive database filter will not drop such variant).
                            		  However, It may be labelled either as Somatic / Germline.
                            Choices:
                            		50 (Default) --> Means do not use this filter
                            		Any other number --> cosmic priority cutoff threshold.
      -v, --version         show program's version number and exit
      -h, --help            show this help message and exit

