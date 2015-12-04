# CCS: PacBio Circular Consensus Sequence Analysis

todo

Table of contents
=================

  * [Overview](#overview)
  * [Manual](#manual)
    * [Running with SMRTLink](#running-with-smrtlink)
    * [Running on the Command Line](#running-on-the-command-line)
    * [Running on the Command Line with pbsmrtpipe](#running-on-the-command-line-with-pbsmrtpipe)
  * [Advanced Analysis Options](#advanced-analysis-options)
    * [SMRTLink/pbsmrtpipe CCS Options](#smrtlinkpbsmrtpipe-ccs-options)
    * [CCS Options](#ccs-options)
  * [Files](#files)
    * [CCS Files](#pbalign-files)
  * [Algorithm Module](#algorithm-module)
  * [Glossary](#glossary)

## Overview

[![Circle CI](https://circleci.com/gh/PacificBiosciences/pbccs.svg?style=svg)](https://circleci.com/gh/PacificBiosciences/pbccs)

![Image of SMRTbell](http://www.evolvedmicrobe.com/CCS.png)

The ccs program takes multiple reads of the same SMRTbell sequence and combines them to produce one high quality consensus sequence.

## Manual

### Running with SMRTLink

To run Isoseq using SMRTLink, follow the usual steps for analysing data on SMRTLink. TODO: Link to document explaining SMRTLink.

### Running on the Command Line

On the command line, the analysis is performed with a single call of the function `ccs`:

```
ccs myresults.bam mynewbam.subreads.bam
```

The general format for a CCS command is:

```
 ccs [OPTIONS] OUTPUT FILES...
```

### Running on the Command Line with pbsmrtpipe

####Install pbsmrtpipe
pbsmrtpipe is a part of `smrtanalysis-3.0` package and will be installed
if `smrtanalysis-3.0` has been installed on your system. Or you can [download   pbsmrtpipe](https://github.com/PacificBiosciences/pbsmrtpipe) and [install](http://pbsmrtpipe.readthedocs.org/en/master/).
    
You can verify that pbsmrtpipe is running OK by:

    pbsmrtpipe --help

#### Create a dataset
Now create an XML file from your subreads.

```
dataset create --type SubreadSet my.subreadset.xml subreads1.bam subreads2.bam ...
```
This will create a file called `my.subreadset.xml`. 


#### Create and edit ccs options and global options for `pbsmrtpipe`.
Create a global options XML file which contains SGE related, job chunking and
job distribution options that you may modify by:

```
 pbsmrtpipe show-workflow-options -o global_options.xml
```

Create a basemod options XML file which contains ccs-related options that 
you may modify by:
```
 pbsmrtpipe show-template-details pbsmrtpipe.pipelines.sa3_ds_ccs -o ccs_options.xml
```

The entries in the options XML files have the format:

```
 <option id="pbtranscript.task_options.min_seq_len">
            <value>300</value>
        </option>
```

And you can modify options using your favorite text editor, such as vim.

#### Run BaseMod from pbsmrtpipe
Once you have set your options, you are ready to run basemod via pbsmrtpipe:

```
pbsmrtpipe pipeline-id pbsmrtpipe.pipelines.sa3_ds_ccs -e eid_subread:my.subreadset.xml --preset-xml=ccs_options.xml --preset-xml=global_options.xml
```


## Advanced Analysis Options

### SMRTLink/pbsmrtpipe Basemod Options


### CCS Options

|           Option           |     Example (Defaults)      |                                                                                                                                                                                                                                                                                                    Explanation                                                                                                                                                                                                                                                                                                     |
| -------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Output File Name           | myResult.bam                | This argument is the first argument that comes after the named arguments shown below.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Input Files                | myData.subreads.bam         | The end of the CCS command following the named arguments and the output file name is the name of one, or more, subread.bam files to be processed.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Version Number             | --version                   | Prints the version number                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Report File Name           | --reportFile=ccs_report.csv | The report file contains a result tally of the outcomes for all ZMWs that were processed. If the filename is not given the report is outputted to the CSV file ccs_report.csv  In addition to the count of successfully produced consensus sequences, this file lists how many ZMWs failed various data quality filters (SNR too low, not enough full passes, etc.) and is useful for diagnosing unexpected drops in yield.                                                                                                                                                                                        |
| Minimum SNR                | --minSnr=4.0                | This filter removes data that is likely to contain deletions.  SNR is a measure of the strength of signal for all 4 channels (A, C, G, T) used to detect basepair incorporation.  The SNR can vary depending on where in the ZMW a SMRTbell stochastochastically lands when loading occurs.  SMRTbells that land near the edge and away from the center of the ZMW have a less intense signal, and as a result can contain sequences with more "missed" basepairs.  This value sets the threshold for minimum required SNR for any of the four channels.  Data with SNR < 4 is typically considered lower quality. |
| Minimum Length             | --minLength=10              | Sets a minimum length requirement for the median size of insert reads in order to generate a consensus sequence.  If the targeted template is known to be a particular size range, this can filter out alternative DNA templates.                                                                                                                                                                                                                                                                                                                                                                                  |
| Minimum Number of Passes   | --minPasses=3               | Sets a minimum number of passes for a ZMW to be emitted.  This is the number of full passes.  Full passes must have an adapter hit before and after the insert sequence and so does not include any partial passes at the start and end of the sequencing reaction.  Additionally, the full pass count does not include any reads that were dropped by the Z-Filter.                                                                                                                                                                                                                                                    |
| Minimum Predicted Accuracy | --minPredictedAccuracy=0.9  | The minimum predicted accuracy of a read.  CCS generates an accuracy prediction for each read, defined as the expected percentage of matches in an alignment of the consensus sequence to the true read.  A value of 0.99 indicates that only reads expected to be 99% accurate are emitted.                                                                                                                                                                                                                                                                                                                       |
| Minimum Z Score            | --minZScore=-5              | The minimum Z-Score for a subread to be included in the consensus generating process. For more information, see the "What are Z-Scores" section below.                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ZMWs to Process            | --zmws=0-100000000000       | If the consensus sequence for only a subset of ZMWs is required, they can be specified here.  ZMWs an be specified either by range (--zmws=1-2000) by values (--zmws=5,10,20), or by both (--zmws=5-10,35,1000-2000).  Simply use a comma separated list with no spaces.                                                                                                                                                                                                                                                                                                                                           |
| Number of Threads to Use   | --numThreads=0              | How many threads to use while processing.  By default, ccs will use as many threads as there are available cores to minimize processing time, but fewer threads can be specified here.                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Log File                   | --logFile=mylog.txt         | The name of a log file to use, if none is given the logging information is printed to STDERR.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Log level verbosity        | --logLevel=INFO             | How much log data to produce? By setting --logLevel=DEBUG, you can obtain detailed information on what ZMWs were dropped during processing, as well as any errors which may have appeared.                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Overwrite output file        | --force             | When you don't care it already exists.                                                                                                                                                                                                                                                                                                                        
