# dragonflye-nf
A generic pipeline for creating long-read assemblies. Supports both long (Oxford Nanopore) reads, or hybrid assemblies with both short and long reads.
Optionally annotate genes. Collects quality info on both incoming and outgoing datasets. 

## Analyses

* Read trimming & QC: [fastp](https://github.com/OpenGene/fastp) and [filtlong](https://github.com/rrwick/Filtlong)
* Genome Assembly: [dragonflye](https://github.com/rpetit3/dragonflye) (long reads, or hybrid)
* Gene Annotation: [prokka](https://github.com/tseemann/prokka) or [bakta](https://github.com/oschwengers/bakta)
* Assembly QC: [quast](https://github.com/ablab/quast), [bandage](https://github.com/rrwick/bandage)

## Usage

By default, dragonflye will be used for assembly of long reads, and no gene annotation will be run:
```
nextflow run BCCDC-PHL/dragonflye-nf \
  --long_only \
  --fastq_input_long <long-read fastq input directory> \
  --outdir <output directory>
```

...or a hybrid assembly can be generated by supplying short reads:

```
nextflow run BCCDC-PHL/dragonflye-nf \
  --hybrid \
  --fastq_input <short-read fastq input directory> \
  --fastq_input_long <long-read fastq input directory> \
  --outdir <output directory>
```


Prokka and/or bakta can be used with the `--prokka` and `--bakta` flags:
```
nextflow run BCCDC-PHL/dragonflye-nf \
  --long_only \
  --fastq_input_long <long-read fastq input directory> \
  --prokka \
  --bakta \
  --outdir <output directory>
```


The pipeline also supports a 'samplesheet input' mode. Pass a `samplesheet.csv` file with the headers `ID`, `R1`, `R2`,`LONG`:
```
nextflow run BCCDC-PHL/routine-assembly-nf \
  --samplesheet_input <samplesheet.csv> \
  --outdir <output directory>
```

Eg:
```
ID,R1,R2,LONG
sample-01,/path/to/sample-01_R1.fastq.gz,/path/to/sample-01_R2.fastq.gz,/path/to/sample-01_RL.fastq.gz
sample-02,/path/to/sample-02_R1.fastq.gz,/path/to/sample-02_R2.fastq.gz,/path/to/sample-02_RL.fastq.gz
sample-03,/path/to/sample-03_R1.fastq.gz,/path/to/sample-03_R2.fastq.gz,/path/to/sample-03_RL.fastq.gz
```

## Output
An output directory will be created for each sample under the directory provided with the `--outdir` flag. The directory will be named by sample ID, inferred from
the fastq files (all characters before the first underscore in the fastq filenames), or the `ID` field of the samplesheet, if one is used.

If we have `sample-01_R{1,2}.fastq.gz`, in our `--fastq_input` directory, the output directory will be:

```
sample-01
├── sample-01_20211125165316_provenance.yml
├── sample-01_fastp.csv
├── sample-01_fastp.json
├── sample-01_dragonflye_short.fa
├── sample-01_dragonflye_short.log
└── sample-01_dragonflye_short_quast.csv
```

Including the tool name suffixes on output files allows re-analysis of the same sample with multiple tools without conflicting output filenames:

```
sample-01
├── sample-01_20211125165316_provenance.yml
├── sample-01_20211128122118_provenance.yml
├── sample-01_fastp.csv
├── sample-01_fastp.json
├── sample-01_dragonflye_hybrid_bakta.gbk
├── sample-01_dragonflye_hybrid_bakta.gff
├── sample-01_dragonflye_hybrid_bakta.json
├── sample-01_dragonflye_hybrid_bakta.log
├── sample-01_dragonflye_hybrid_bandage.png
├── sample-01_dragonflye_hybrid_prokka.gbk
├── sample-01_dragonflye_hybrid_prokka.gff
├── sample-01_dragonflye_hybrid_quast.csv
├── sample-01_dragonflye_hybrid_quast.csv
├── sample-01_dragonflye_hybrid.fa
├── sample-01_dragonflye_hybrid.gfa
├── sample-01_dragonflye_hybrid.log
├── sample-01_dragonflye_short_bakta.gbk
├── sample-01_dragonflye_short_bakta.gff
├── sample-01_dragonflye_short_bakta.json
├── sample-01_dragonflye_short_bakta.log
├── sample-01_dragonflye_short_bandage.png
├── sample-01_dragonflye_short_prokka.gbk
├── sample-01_dragonflye_short_prokka.gff
├── sample-01_dragonflye_short_quast.csv
├── sample-01_dragonflye_short_quast.csv
├── sample-01_dragonflye_short.fa
├── sample-01_dragonflye_short.gfa
└── sample-01_dragonflye_short.log
```

If the `--versioned_outdir` flag is used, then a sub-directory will be created below each sample, named with the pipeline name and minor version:

```
sample-01
    └── dragonflye-nf-v0.4-output
        ├── sample-01_20220216172238_provenance.yml
        ├── sample-01_fastp.csv
        ├── sample-01_fastp.json
        ├── sample-01_dragonflye_short.fa
        ├── sample-01_dragonflye_short.log
        ├── sample-01_dragonflye_short_prokka.gbk
        ├── sample-01_dragonflye_short_prokka.gff
        └── sample-01_dragonflye_short_quast.csv
```

This is provided as a way of combining outputs of several different pipelines or re-analysis with future versions of this pipeline:

```
sample-01
    └── dragonflye-nf-v0.4-output
    │   ├── sample-01_20220216172238_provenance.yml
    │   ├── sample-01_fastp.csv
    │   ├── sample-01_fastp.json
    │   ├── sample-01_dragonflye_short.fa
    │   ├── sample-01_dragonflye_short.log
    │   ├── sample-01_dragonflye_short_prokka.gbk
    │   ├── sample-01_dragonflye_short_prokka.gff
    │   └── sample-01_dragonflye_short_quast.csv
    └── dragonflye-nf-v0.5-output
        ├── sample-01_20220612091224_provenance.yml
        ├── sample-01_fastp.csv
        ├── sample-01_fastp.json
        ├── sample-01_dragonflye_short.fa
        ├── sample-01_dragonflye_short.log
        ├── sample-01_dragonflye_short_prokka.gbk
        ├── sample-01_dragonflye_short_prokka.gff
        └── sample-01_dragonflye_short_quast.csv
```

### Provenance files
For each pipeline invocation, each sample will produce a `provenance.yml` file with the following contents:

```yml
- pipeline_name: BCCDC-PHL/dragonflye-nf
  pipeline_version: 0.4.0
- timestamp_analysis_start: 2022-08-16T13:22:11.553143
- input_filename: sample-01_R1.fastq.gz
  sha256: 4ac3055ac5f03114a005aff033e7018ea98486cbebdae669880e3f0511ed21bb
  file_type: fastq-input
- input_filename: sample-01_R2.fastq.gz
  sha256: 8db388f56a51920752319c67b5308c7e99f2a566ca83311037a425f8d6bb1ecc
  file_type: fastq-input
- process_name: fastp
  tools:
    - tool_name: fastp
      tool_version: 0.23.1
- process_name: dragonflye
  tools:
    - tool_name: dragonflye
      tool_version: 1.1.0
- process_name: prokka
  tools:
    - tool_name: prokka
      tool_version: 1.14.5
      parameters:
        - parameter: --compliant
          value: null
- process_name: quast
  tools:
    - tool_name: quast
      tool_version: 5.0.2
      parameters:
        - parameter: --space-efficient
          value: null
        - parameter: --fast
          value: null
```

The filename of the provenance file includes a timestamp with format `YYYYMMDDHHMMSS` to ensure that re-analysis of the same sample will create a unique `provenance.yml` file.
