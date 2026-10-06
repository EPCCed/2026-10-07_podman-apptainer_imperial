---
title: Using Apptainer to Run BLAST+
teaching: 30
exercises: 30
---

::::::::::::::::::::::::::::::::::::::: objectives

- Show example of using Apptainer with a common bioinformatics tool.

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How can I use Apptainer to run bioinformatics workflows with BLAST+?

::::::::::::::::::::::::::::::::::::::::::::::::



We have now learned enough to be able to use Apptainer to deploy software on a remote HPC system without us needing to install the software itself on the host system.

In this section we will demonstrate the use of an Apptainer container image that provides the BLAST+ bioinformatics software. The BLAST+ suite of software tools is typically complex to install from source by hand on a HPC system. Using containers we are able to avoid this complexity and get up and running with the software quickly. 

:::::::::::::::::::::::::::::::::::::::::  callout

## Source material

This example is based on the example from the official [NCBI BLAST+ Docker
container documentation](https://github.com/ncbi/blast_plus_docs#step-2-import-sequences-and-create-a-blast-database)
Note: the `efetch` parts of the step-by-step guide do not currently work using
Apptainer version of the image so we provide a dataset with the data already
downloaded.

(This is because the NCBI BLAST+ Docker container image has the `efetch` tool
installed in the `/root` directory and this special location gets overwritten
during the conversion to a Apptainer container image.)

::::::::::::::::::::::::::::::::::::::::::::::::::

## Download the required data


Download the [blast_example.tar.gz](https://epcced.github.io/https://github.com/EPCCed/2026-10-07_podman-apptainer_imperial/raw/refs/heads/main/episodes/files/blast_example.tar.gz).

Unpack the archive which contains the downloaded data required for the BLAST+ example:

```bash
remote$ wget https://github.com/EPCCed/2026-10-07_podman-apptainer_imperial/raw/refs/heads/main/episodes/files/blast_example.tar.gz
remote$ tar -xvf blast_example.tar.gz
```

```output
x blast/
x blast/blastdb/
x blast/queries/
x blast/fasta/
x blast/results/
x blast/blastdb_custom/
x blast/fasta/nurse-shark-proteins.fsa
x blast/queries/P01349.fsa
```

Finally, move into the newly created directory:

```bash
remote$ cd blast
remote$ ls  
```

```output
blastdb        blastdb_custom fasta          queries        results
```

## Create the Apptainer container image

NCBI provide official Docker containers with the BLAST+ software hosted on Docker Hub. We can create
an Apptainer container image from the Docker container image with:

```bash
remote$ apptainer pull ncbi-blast.sif docker://docker.io/ncbi/blast
```

```output
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
INFO:    Fetching OCI image...
29.0MiB / 29.0MiB [=========================================================================================================================================================================] 100 % 22.2 MiB/s 0s
14.8MiB / 14.8MiB [=========================================================================================================================================================================] 100 % 22.2 MiB/s 0s
50.4MiB / 50.4MiB [=========================================================================================================================================================================] 100 % 22.2 MiB/s 0s
448.8KiB / 448.8KiB [=======================================================================================================================================================================] 100 % 22.2 MiB/s 0s
40.9MiB / 40.9MiB [=========================================================================================================================================================================] 100 % 22.2 MiB/s 0s
150.7MiB / 150.7MiB [=======================================================================================================================================================================] 100 % 22.2 MiB/s 0s
22.7MiB / 22.7MiB [=========================================================================================================================================================================] 100 % 22.2 MiB/s 0s
49.1MiB / 49.1MiB [=========================================================================================================================================================================] 100 % 22.2 MiB/s 0s
INFO:    Extracting OCI image...
INFO:    Inserting Apptainer configuration...
INFO:    Creating SIF file...
[======================================================================================================================================================================================================] 100 % 0s
```

Now we have a container with the software in, we can use it.

## Build and verify the BLAST database

Our example dataset has already downloaded the query and database sequences. We first
use these downloaded data to create a custom BLAST database by using a container to run
the command `makeblastdb` with the correct options.

```bash
remote$ apptainer exec --cleanenv ncbi-blast.sif \
    makeblastdb -in fasta/nurse-shark-proteins.fsa -dbtype prot \
    -parse_seqids -out nurse-shark-proteins -title "Nurse shark proteins" \
    -taxid 7801 -blastdb_version 5
```

```output
Building a new DB, current time: 10/06/2026 14:05:20
New DB name:   /home/auser/test/blast/blast/nurse-shark-proteins
New DB title:  Nurse shark proteins
Sequence type: Protein
Keep MBits: T
Maximum file size: 3000000000B
Adding sequences from FASTA; added 7 sequences in 0.021101 seconds.

```

:::::::::::::::::::::::::::::::::::::::::  callout

## `--cleanenv` option

By default, Apptainer makes sure most of your host environment settings are available in the running container. Full details of what Apptainer does [are available in the User Guide](https://apptainer.org/docs/user/main/environment_and_metadata.html). However, this can cause compatibility problems so it is often best to start in the running container with a clean environment and only add environment settings as needed. The `--cleanenv` option to Apptainer tells the running container to not take environment settings from the host.

::::::::::::::::::::::::::::::::::::::::::::::::::

To verify the newly created BLAST database above, you can run the `blastdbcmd -entry all -db nurse-shark-proteins -outfmt "%a %l %T"` command to display the accessions, sequence length, and common name of the sequences in the database.

```bash
remote$ apptainer exec --cleanenv ncbi-blast.sif \
    blastdbcmd -entry all -db nurse-shark-proteins -outfmt "%a %l %T"
```

```output
Q90523.1 106 7801
P80049.1 132 7801
P83981.1 53 7801
P83977.1 95 7801
P83984.1 190 7801
P83985.1 195 7801
P27950.1 151 7801
```

Now we have our database we can run queries against it.

## Run a query against the BLAST database

Lets execute a query on our database using the `blastp` command:

```bash
remote$ apptainer exec --cleanenv ncbi-blast.sif \
    blastp -query queries/P01349.fsa -db nurse-shark-proteins \
    -out results/blastp.out
```

At this point, you should see the results of the query in the output file `results/blastp.out`.
To view the content of this output file, use the command `less results/blastp.out`.

```bash
remote$ less results/blastp.out
```

```output
...output trimmed...

Query= sp|P01349.2|RELX_CARTA RecName: Full=Relaxin; Contains: RecName:
Full=Relaxin B chain; Contains: RecName: Full=Relaxin A chain

Length=44
                                                                       Score     E
Sequences producing significant alignments:                          (Bits)  Value

P80049.1 RecName: Full=Fatty acid-binding protein, liver; AltName...  14.2    0.96


>P80049.1 RecName: Full=Fatty acid-binding protein, liver; AltName: Full=Liver-type
fatty acid-binding protein; Short=L-FABP
Length=132

...output trimmed...
```

With your query, BLAST identified the protein sequence P80049.1 as a match with a score
of 14.2 and an E-value of 0.96.

## Accessing online BLAST databases

As well as building your own local database to query, you can also access databases that are
available online. For example, to see which databases are available online in the Google Compute
Platform (GCP):

```bash
remote$ apptainer exec --cleanenv ncbi-blast.sif update_blastdb.pl --showall pretty --source gcp
```

```output
Connected to GCP
BLASTDB                                                      DESCRIPTION                                                                                                              SIZE (GB)      LAST_UPDATED
nr                                                           All non-redundant GenBank CDS translations+PDB+SwissProt+PIR+PRF excluding environmental samples from WGS projects        753.9510      2026-09-27
swissprot                                                    Non-redundant UniProtKB/SwissProt sequences                                                                                 0.3659      2026-09-15
refseq_protein                                               NCBI Protein Reference Sequences                                                                                          257.3614      2026-08-27
landmark                                                     Landmark database for SmartBLAST                                                                                            0.4693      2025-08-06
pdbaa                                                        PDB protein database                                                                                                        0.2615      2026-09-29
nt                                                           Nucleotide collection (nt)                                                                                               1199.4928      2026-09-24
core_nt                                                      Core nucleotide BLAST database                                                                                            301.4805      2026-09-19
pdbnt                                                        PDB nucleotide database                                                                                                     0.0155      2026-09-24
patnt                                                        Nucleotide sequences derived from the Patent division of GenBank                                                           19.2072      2026-09-19
refseq_rna                                                   NCBI Transcript Reference Sequences                                                                                        80.1543      2026-09-16

...output trimmed...
```

Similarly, for databases hosted at NCBI:

```bash
remote$ apptainer exec --cleanenv ncbi-blast.sif update_blastdb.pl --showall pretty --source ncbi
```

```output
Connected to NCBI
BLASTDB                                                      DESCRIPTION                                                                                                              SIZE (GB)      LAST_UPDATED
18S_fungal_sequences                                         18S ribosomal RNA sequences (SSU) from Fungi type and reference material                                                    0.0026      2026-10-03
Betacoronavirus                                              Betacoronavirus                                                                                                            71.0289      2025-06-26
28S_fungal_sequences                                         28S ribosomal RNA sequences (LSU) from Fungi type and reference material                                                    0.0072      2026-10-03
16S_ribosomal_RNA                                            16S ribosomal RNA (Bacteria and Archaea type strains)                                                                       0.0184      2026-10-03
ITS_RefSeq_Fungi                                             Internal transcribed spacer region (ITS) from Fungi type and reference material                                             0.0091      2026-10-05
ITS_eukaryote_sequences                                      ITS eukaryote BLAST                                                                                                         0.0344      2026-10-05
LSU_eukaryote_rRNA                                           Large subunit ribosomal nucleic acid for Eukaryotes                                                                         0.0053      2022-12-05
LSU_prokaryote_rRNA                                          Large subunit ribosomal nucleic acid for Prokaryotes                                                                        0.0041      2022-12-05
SSU_eukaryote_rRNA                                           Small subunit ribosomal nucleic acid for Eukaryotes                                                                         0.0063      2022-12-05
env_nt                                                       environmental samples                                                                                                       0.4413      2026-09-16
env_nr                                                       Proteins from WGS metagenomic projects (env_nr).                                                                            4.4552      2026-09-30

...output trimmed...
```

## Notes

You have now completed a simple example of using a complex piece of bioinformatics software
through Apptainer containers. You may have noticed that some things just worked without
you needing to set them up even though you were running using containers:

1. We did not need to explicitly bind any files/directories in to the container. This worked
   because Apptainer automatically binds the current directory into the running container, so
   any data in the current directory (or its subdirectories) will generally be available in
   running Apptainer containers. (Note, this is different from the behaviour of the Podman 
   containers we were using earlier in this workshop.)
2. Access to the internet is automatically available within the running container in the same
   way as it is on the host system without us needed to specify any additional options.
3. Files and data we create within the container have the right ownership and permissions for
   us to access outside the container.

In addition, we were able to use the tools in the container image provided by NCBI without having
to do any work to install the software irrespective of the computing platform that we are using.
(In fact, the example this is based on runs the pipeline using Docker on a cloud computing platform
rather than on your local system.)

:::::::::::::::::::::::::::::::::::::: keypoints

- We can use containers to run software without having to install it
- The commands we use are very similar to those we would use natively
- Apptainer handles a lot of complexity around data and internet access for us

::::::::::::::::::::::::::::::::::::::::::::::::

{% include links.md %}
