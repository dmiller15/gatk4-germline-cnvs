# gatk4-germline-cnvs

- Cohort WDL: Calling a cohort of samples and building a model for denoising further case samples: ``cnv_germline_cohort_workflow.wdl``
- Case WDL: Calling case samples using a previously built model for denoising: ``cnv_germline_case_workflow.wdl``
- Scattered case WDL (recommended): Functionally equivalent to case WDL, written for reducing cloud compute cost (see below) and wall-clock time ``cnv_germline_case_scattered_workflow.wdl``

#### Setting up parameter json file for a run

To get started, create the json template (using ``java -jar wdltool.jar inputs <workflow>``) for the workflow you wish to run and adjust parameters accordingly.

*Please note that there are optional workflow-level and task-level parameters that do not appear in the template file.  These are set to reasonable values by default, but can also be adjusted if desired.*

#### Required parameters in the germline cohort workflow

The reference used must be the same between PoN and case samples.

- ``CNVGermlineCohortWorkflow.cohort_entity_id`` -- Name of the cohort.  Will be used as a prefix for output filenames.
- ``CNVGermlineCohortWorkflow.contig_ploidy_priors`` -- TSV file containing prior probabilities for the ploidy of each contig, with column headers: CONTIG_NAME, PLOIDY_PRIOR_0, PLOIDY_PRIOR_1, ...
- ``CNVGermlineCohortWorkflow.gatk_docker`` -- GATK Docker image (e.g., ``broadinstitute/gatk:latest``).
- ``CNVGermlineCohortWorkflow.intervals`` -- Picard or GATK-style interval list.  For WGS, this should typically only include the chromosomes of interest.
- ``CNVGermlineCohortWorkflow.normal_bais`` -- List of BAI files.  This list must correspond to `normal_bams`.  For example, `["Sample1.bai", "Sample2.bai"]`.
- ``CNVGermlineCohortWorkflow.normal_bams`` -- List of BAM files.  This list must correspond to `normal_bais`.  For example, `["Sample1.bam", "Sample2.bam"]`.
- ``CNVGermlineCohortWorkflow.num_intervals_per_scatter`` -- Number of intervals (i.e., targets or bins) in each scatter for GermlineCNVCaller.  If total number of intervals is not divisible by the value provided, the last scatter will contain the remainder.
- ``CNVGermlineCohortWorkflow.ref_fasta_dict`` -- Path to reference dict file.
- ``CNVGermlineCohortWorkflow.ref_fasta_fai`` -- Path to reference fasta fai file.
- ``CNVGermlineCohortWorkflow.ref_fasta`` -- Path to reference fasta file.
- ``CNVGermlineCohortWorkflow.maximum_number_events_per_sample`` -- Maximum number of events threshold for doing sample QC (recommended for WES is ~100)

In additional, there are optional workflow-level and task-level parameters that may be set by advanced users; for example:

- ``CNVGermlineCohortWorkflow.do_explicit_gc_correction`` -- (optional) If true, perform explicit GC-bias correction when creating PoN and in subsequent denoising of case samples.  If false, rely on PCA-based denoising to correct for GC bias.
- ``CNVGermlineCohortWorkflow.PreprocessIntervals.bin_length`` -- Size of bins (in bp) for coverage collection.  *This must be the same value used for all case samples.*
- ``CNVGermlineCohortWorkflow.PreprocessIntervals.padding`` -- Amount of padding (in bp) to add to both sides of targets for WES coverage collection.  *This must be the same value used for all case samples.*

Further explanation of other task-level parameters may be found by invoking the ``--help`` documentation available in the gatk.jar for each tool.

#### Required parameters in the germline case workflow

The reference, number of intervals per scatter, and bins (if specified) must be the same between cohort and case samples.

- ``CNVGermlineCohortWorkflow.normal_bais`` -- List of BAI files.  This list must correspond to `normal_bams`.  For example, `["Sample1.bai", "Sample2.bai"]`.
- ``CNVGermlineCohortWorkflow.normal_bams`` -- List of BAM files.  This list must correspond to `normal_bais`.  For example, `["Sample1.bam", "Sample2.bam"]`.
- ``CNVGermlineCaseWorkflow.contig_ploidy_model_tar`` -- Path to tar of the contig-ploidy model directory generated by the DetermineGermlineContigPloidyCohortMode task. 
- ``CNVGermlineCaseWorkflow.gatk_docker`` -- GATK Docker image (e.g., ``broadinstitute/gatk:latest``).
- ``CNVGermlineCaseWorkflow.gcnv_model_tars`` -- Array of paths to tars of the contig-ploidy model directories generated by the GermlineCNVCallerCohortMode tasks.
- ``CNVGermlineCaseWorkflow.intervals`` -- Picard or GATK-style interval list.  For WGS, this should typically only include the chromosomes of interest.
- ``CNVGermlineCaseWorkflow.num_intervals_per_scatter`` -- Number of intervals (i.e., targets or bins) in each scatter for GermlineCNVCaller.  If total number of intervals is not divisible by the value provided, the last scatter will contain the remainder.
- ``CNVGermlineCaseWorkflow.ref_fasta_dict`` -- Path to reference dict file.
- ``CNVGermlineCaseWorkflow.ref_fasta_fai`` -- Path to reference fasta fai file.
- ``CNVGermlineCaseWorkflow.ref_fasta`` -- Path to reference fasta file.
- ``CNVGermlineCohortWorkflow.maximum_number_events_per_sample`` -- Maximum number of events threshold for doing sample QC (recommended for WES is ~100)

In additional, there are several task-level parameters that may be set by advanced users as above.

Further explanation of these task-level parameters may be found by invoking the ``--help`` documentation available in the gatk.jar for each tool.

#### Required parameters in the scattered germline case workflow

Same required parameters as in the germline case workflow. However, in order to reduce wall-clock time and compute cost, it is recommended to optimize for the following parameters:

mportant Notes :
- Runtime parameters are optimized for Broad's Google Cloud Platform implementation.
- For help running workflows on the Google Cloud Platform or locally please
view the following tutorial [(How to) Execute Workflows from the gatk-workflows Git Organization](https://gatk.broadinstitute.org/hc/en-us/articles/360035530952).
- Please visit the [User Guide](https://gatk.broadinstitute.org/hc/en-us/categories/360002310591) site for further documentation on our workflows and tools.
- Relevant reference and resources bundles can be accessed in [Resource Bundle](https://gatk.broadinstitute.org/hc/en-us/articles/360036212652).

### Contact Us :
- The following material is provided by the Data Science Platforum group at the Broad Institute. Pleas- ``CNVGermlineCaseScatteredWorkflow.num_samples_per_scatter_block`` -- (recommended WES value=25) number of samples to process in a single block; blocks of this size will be sent to the germline case workflow and processed in a batch; 
- ``CNVGermlineCaseScatteredWorkflow.preemptible_attempts`` -- (recommended value=5) this reduces cost by using preemptible instances
- ``CNVGermlineCaseScatteredWorkflow.mem_gb_for_determine_germline_contig_ploidy`` -- amount of memory allotted for ploidy determination tasks (the lower the cheaper)
- ``CNVGermlineCaseScatteredWorkflow.cpu_for_determine_germline_contig_ploidy`` -- number of CPU cores allotted for ploidy determination tasks (the lower the cheaper)
- ``CNVGermlineCaseScatteredWorkflow.disk_for_determine_germline_contig_ploidy`` -- amount of storage allotted for ploidy determination tasks (the lower the cheaper) 
- ``CNVGermlineCaseScatteredWorkflow.mem_gb_for_germline_cnv_caller`` -- amount of memory allotted for gCNV caller tasks (the lower the cheaper)
- ``CNVGermlineCaseScatteredWorkflow.cpu_for_germline_cnv_caller`` -- number of CPU cores allotted for gCNV caller tasks (the lower the cheaper)
- ``CNVGermlineCaseScatteredWorkflow.disk_for_germline_cnv_caller`` -- amount of storage allotted for gCNV caller tasks (the lower the cheaper)

Note that lowering disk and memory too much will eventually lead to the workflow failing. Lowering thee direct any questions or concerns to one of our forum sites : [GATK](https://gatk.broadinstitute.org/hc/en-us/community/topics) or [Terra](https://support.terra.bio/hc/en-us/community/topics/360000500432). number of CPU cores could increase the wall-clock times.

### Output :
 (Example: - A BAM file and its index)
  
### Software version notes :
- GATK 4.1.5.0
- Cromwell version support 
  - Successfully tested on v47 
  - Does not work on versions < v23 due to output syntax
  
### Important Notes :
- Runtime parameters are optimized for Broad's Google Cloud Platform implementation.
- For help running workflows on the Google Cloud Platform or locally please
view the following tutorial [(How to) Execute Workflows from the gatk-workflows Git Organization](https://gatk.broadinstitute.org/hc/en-us/articles/360035530952).
- Please visit the [User Guide](https://gatk.broadinstitute.org/hc/en-us/categories/360002310591) site for further documentation on our workflows and tools.
- Relevant reference and resources bundles can be accessed in [Resource Bundle](https://gatk.broadinstitute.org/hc/en-us/articles/360036212652).

### Contact Us :
- The following material is provided by the Data Science Platforum group at the Broad Institute. Please direct any questions or concerns to one of our forum sites : [GATK](https://gatk.broadinstitute.org/hc/en-us/community/topics) or [Terra](https://support.terra.bio/hc/en-us/community/topics/360000500432).

### LICENSING :
Copyright Broad Institute, 2020 | BSD-3
This script is released under the WDL open source code license (BSD-3) (full license text at https://github.com/openwdl/wdl/blob/master/LICENSE). Note however that the programs it calls may be subject to different licenses. Users are responsible for checking that they are authorized to run all programs before running this script.

