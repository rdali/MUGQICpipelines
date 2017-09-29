# MUGQIC Pipeline User Manual
<br/>

Bioinformatics pipelines developed at the McGill University and Génome Québec Innovation Centre (MUGQIC), as part of the GenAP project, are available for public use. This document explains how to use the pipelines using Hi-C analysis pipeline hicseq, and the ChIP-Seq pipeline, as examples.

<br/>


## Getting Started:

### Abacus, Guillimin and Mammouth users

<br/>

software and scripts used by MUGQIC are already installed on several Compute Canada servers including guillimin, mammouth and will soon be installed on Cedar and Graham. To access the tools, you will need to add the tool path to your **bash_profile**. The bash profile is a hidden file in your home directory that sets up your environment every time you log in.

```
## open bash_profile:
nano $HOME/.bash_profile
```

paste the following lines of code and save the file:

```
## MUGQIC genomes and modules
export MUGQIC_INSTALL_HOME=/cvmfs/soft.mugqic/CentOS6
module use $MUGQIC_INSTALL_HOME/modulefiles
module load mugqic/python/2.7.13
module load mugqic/mugqic_pipelines/<latest_version>
export JOB_MAIL=<my.name@my.email.ca>
export RAP_ID=<my-rap-id>
```


You will need to replace text within **< >**, with your information.

To find out the latest mugqic_pipelines version use the output of:

```
module avail 2>&1 | grep mugqic/mugqic_pipelines
```

**JOB_MAIL** is the email to which the notifications are sent after each job. We advise you to create a separate email for jobs since you can receive hundreds of emails per pipeline. You can also de-activate the email sending option by removing the **"-M $JOB_MAIL"** option from the .ini files (discussed below).

**RAP_ID** is the Resource Allocation Project ID from Compute Canada. It is usually in the format: abc-123-ab

<br/>

By adding those lines to your bash profile, you are now ready to use our pipelines. This also gives you access to hundreds of bioinformatics tools pre-installed by our team. To explore the available tools, you can type:

```
module avail mugqic/
```

To load a tool, for example samtools, you can use:

```
# module add mugqic/<tool>/<version>
module add mugqic/samtools/1.4.1

# Now samtools 1.4.1 is available to use. To check:
samtools
```



<br/>




You also have access to pre-installed genomes available in:  `$MUGQIC_INSTALL_HOME/genomes/`
To check all the available species, type:

```
ls $MUGQIC_INSTALL_HOME/genomes/species
```

All genome-related files, including indices for different aligners and annotation files can be found in `$MUGQIC_INSTALL_HOME/genomes/species/<species_scientific_name>.<assembly>/` 

<br/>

## Usage:

Now that your variables are set, you can launch any pipeline using:
<br/>
`<pipeline_name>.py`

To check the help information for our hicseq (Hi-C analysis) and our chipseq piplines, try:

```
hicseq.py --help
chipseq.py --help
```

To use most of our pipelines you will need two types of files; a configuration file that stores all the parameters used by the pipeline (extention .ini) and a readset file that stores all the information about your samples.

<br/>


### Configuration File:

MUGQIC pipelines are multi-step pipelines that run several tools, each with its own parameter inputs. All those parameters are stored in configuration files with `.ini` extension. Those files have a structure similar to Microsoft Windows INI files, where parameters are divided within sections.


<br/>
![light](Brain-Light-Bulb_small.png)  

```
What is a "configuration file" or an "ini" file and why do we need it?
An ini file is a file that contains parameters needed to run a pipeline. 
Our genome alignment pipeline contains over 20 steps, each involving over 5 
parameters per step. Imagine having to type all 100 parameters to run a pipeline! 
For simplicity, all the parameters are stored in an "ini" file (extention .ini)
that accompanies the pipeline.
Try opening an ini file in a text editor and look at its content!
```

<br/>


Each pipeline has several configuration/ini files in:<br/>
`$MUGQIC_PIPELINES_HOME/pipelines/<pipeline_name>/<pipeline_name>.*.ini`

For hicseq, that would be:

```
ls $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.*.ini
```

For chipseq, that would be:

```
ls $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.*.ini
```

<br/>
You will find a `<pipeline_name>.base.ini` as well as an ini file for particular servers like Guillimin (`<pipeline_name>.guillimin.ini`). The base.ini file has all the paramters needed by the pipeline but is optimized for usage on our own server, Abacus. To use the pipeline on Guillimin, you will need to use both base.ini and guillimin.ini, as such:

```
hicseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.guillimin.ini ...
```

<br/>
To change different parameters in the ini files, you can create your own ini file and overwrite the required parameters. For example, to change the number of threads for trimmomatic and hicup, I can create my own ini file: `hicseq.test.ini`
and in it I can include the parameters to be changed:

```
[trimmomatic]
threads=2

[hicup_align]
threads=4
```

then add my ini file after the other ini files:

```
hicseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.guillimin.ini hicseq.test.ini...
```

For different species, we have custom ini files stored in `$MUGQIC_PIPELINES_HOME/resources/genomes/config/`

<br/>


### Readset File:



The readset file is a **tab-separated** file that contains the following information:

**Sample**: must contain letters A-Z, numbers 0-9, hyphens (-) or underscores (_) only; BAM files will be merged into a file named after this value; *mandatory*.<br/>
**Readset**): a unique readset name with the same allowed characters as above; *mandatory*.<br/>
**Library**: *optional*.<br/>
**RunType**: PAIRED_END or SINGLE_END; *mandatory*.<br/>
**Run**: *optional*.<br/>
**Lane**: *optional*.<br/>
**Adapter1**: sequence of the forward trimming adapter<br/>
**Adapter2**: sequence of the reverse trimming adapter<br/>
**QualityOffset**: quality score offset integer used for trimming; *optional*.<br/>
**BED**: relative or absolute path to BED file; *optional*.<br/>
**FASTQ1**: relative or absolute path to first FASTQ file for paired-end readset or single FASTQ file for single-end readset; *mandatory if BAM value is missing*.<br/>
**FASTQ2**: relative or absolute path to second FASTQ file for paired-end readset; *mandatory if RunType value is "PAIRED_END"*.<br/>
**BAM**: relative or absolute path to BAM file which will be converted into FASTQ files if they are not available; *mandatory if FASTQ1 value is missing, ignored otherwise*.<br/><br/>

Example:

```
Sample  Readset Library RunType Run Lane    Adapter1    Adapter2    QualityOffset   BED FASTQ1  FASTQ2  BAM
sampleA readset1    lib0001 PAIRED_END  run100  1   AGATCGGAAGAGCACACGTCTGAACTCCAGTCA   AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT   33  path/to/file.bed    path/to/readset1.paired1.fastq.gz   path/to/readset1.paired2.fastq.gz   path/to/readset1.bam
sampleA readset2    lib0001 PAIRED_END  run100  2   AGATCGGAAGAGCACACGTCTGAACTCCAGTCA   AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT   33  path/to/file.bed    path/to/readset2.paired1.fastq.gz   path/to/readset2.paired2.fastq.gz   path/to/readset2.bam
sampleB readset3    lib0002 PAIRED_END  run200  5   AGATCGGAAGAGCACACGTCTGAACTCCAGTCA   AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT   33  path/to/file.bed    path/to/readset3.paired1.fastq.gz   path/to/readset3.paired2.fastq.gz   path/to/readset3.bam
sampleB readset4    lib0002 PAIRED_END  run200  6   AGATCGGAAGAGCACACGTCTGAACTCCAGTCA   AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT   33  path/to/file.bed    path/to/readset4.paired1.fastq.gz   path/to/readset4.paired2.fastq.gz   path/to/readset4.bam
```

If some optional information is missing, leave its position empty.

**sample vs readset:**
<br/>
readsets refer to replicates that belong to a particular sample. If a sample was divided over 3 lanes, each lane output would be a readset of that sample. Most pipelines merge readsets and run the analysis based on samples.


<br/>

<br/>
![light](Brain-Light-Bulb_small.png)  

```
What is a "Readset file" and why do we need it?
A readset file is another file that accompanies our pipelines. 
While the configuration files contains information on the parameters needed by the 
tools in the pipeline, the readset file contains information about the samples to be 
analyzed. In the Readset file, you list each readset used for the analysis, which 
samples are to be merged and where your fastq files or bam files are located. 
```

<br/>

#### Creating a Readset File:

If you have access to Abacus, we provide a script `$MUGQIC_PIPELINES_HOME/utils/nanuq2mugqic_pipelines.py` that creates that can access your Nanuq data, creates symlinks to the data on Abacus and creates the Readset file for you.

If your data is on nanuq but you do not have access to Abacus, there is a helper script `$MUGQIC_PIPELINES_HOME/utils/csvToreadset.R` that takes a csv file downloadable from nanuq and creates the Readset file. However, you will have to download the data from Nanuq yourself.

If your data is not on nanuq, you will have to manually create the Readset file. You can use a template and enter your samples manually. Remember that it is a tab separated file. 

<br/>
<br/>

### Design File:

Certain pipelines where samples are compared against other samples, like chipseq and rnaseq, require a design file that describes which samples are to be compared. We will discuss this later during an example.

<br/>
![light](Brain-Light-Bulb_small.png)  

```
What is a "Design file" and why do we need it?
A Design file is another file that accompanies some of our pipelines,
where sample comparison is part of the pipeline. Unlike the configuration file and the 
Readset file, the Design file is not required by every pipeline. For whether the pipeline 
you are interested in requires a Design file and the format of the file, read the specific
help pages for your pipeline.

```
<br/>

## Example run:

<br/>
### hicseq Test Dataset:



Let's now run the pipeline using a test dataset. We will use the first 2 million reads from HIC010 from Rao et al. 2014 (SRR1658581.sra). This is an in-situ Hi-C experiment of GM12878 using MboI restriction enzyme.

We will start by downloading the dataset from HERE (add URL of zipped file).
In the downloaded zip file, you will find the two fastq read files in folder rawData and will find the readset file (readsets.HiC010.tsv) that describes that dataset.

We will run this analysis on guillimin as follows:

```
hicseq -c $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/hicseq/hicseq.guillimin.ini -r readsets.HiC010.tsv -s 1-15 -e MboI > hicseqScript_SRR1658581.txt
```

**-c** defines the ini configuration files <br/>
**-r** defines the readset file <br/>
**-s** defines the steps of the pipeline to execute. To check pipeline steps use `hicseq --help` <br/>
**-e** defines the restriction enzyme used in the HiC library <br/>

**hicseqScript_SRR1658581.txt** is the script that contains the analysis commands.  <br/>

To run it, use:

```
bash hicseqScript_SRR1658581.txt
```

You will not see anything happen, but the commands will be sent to the queue. **So do not run this more than once per job**. <br/>
To confirm that the commands have been submitted, wait a minute or two depending on the server and type:

```
showq | grep <userID>
```

#### Congratulations! you just ran the hicseq pipeline.
After the processing is complete, you can access quality control plots in the `homer_tag_directory/HomerQcPlots`. You can find the compartment data in the `compartments` folder, TADs in the `TADs` folder and significant interactions in the `peaks` folder. 


For more information about ouput formats please consult the webpage of the third party tool used. 


**Note:** The hicseq pipeline also analyzes capture hic data if the "-t capture" flag is used. For more information on the available steps in that pipeline use: `hicseq --help`

<br/>

### Design File:

Certain pipelines that involve comparing and contrasting samples, need a Design File. 
<br/>

The Design File is a **tab-separated** plain text file with one line per sample and the following columns:

**Sample**: first column; must contain letters A-Z, numbers 0-9, hyphens (-) or underscores (_) only; the sample name must match a sample name in the readset file; *mandatory*.

**contrast**: each of the following columns defines an experimental design contrast; the column name defines the contrast name, and the following values represent the sample group membership for this contrast:

+ **'0' or ''**: the sample does not belong to any group.

+ **'1'**: the sample belongs to the control group.

+ **'2'**: the sample belongs to the treatment test case group.

Example:

```
Sample  Contrast_AB   Contrast_AC
sampleA 1   1
sampleB 2   0
sampleC 0   2
sampleD 0   0
```

where `Contrast_AB` compares treatment sampleB to control sampleA, while `Contrast_AC` compares sampleC to sampleA.

You can add several contrasts per design file.

To see how this works, lets run a ChIP-Seq experiment.

<br/>
### chipseq Test Dataset:

We will use the first 2 million reads from [ENCFF361CSC](https://www.encodeproject.org/experiments/ENCSR828XQV/) and [ENCFF837BCE](https://www.encodeproject.org/experiments/ENCSR236YGF/) from ENCODE. 

We will start by downloading the dataset from HERE (add URL of zipped file).
In the downloaded zip file, you will find the two fastq read files in folder rawData and will find the readset file (readsets.chipseqTest.tsv) that describes that dataset. You will also find the design file (designfile_chipseq.txt) that contains the contrasts of interest.  (EXPAND)

We will run this analysis on guillimin as follows:

```
chipseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/chipseq/chipseq.guillimin.ini -r readsets.chipseqTest.tsv -d designfile_chipseq.txt -s 1-12 > chipseqScript.txt

bash chipseqScript.txt

```
<br/>
<br/>

## For more information:

For more information or help with particular pipelines, you can send us an email to:
info@computationalgenomics.ca

Or drop by during our [Open Door](http://www.computationalgenomics.ca/open-door/) slots. 





