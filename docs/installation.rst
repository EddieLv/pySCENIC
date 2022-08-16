Installation and Usage
======================

Installation
------------

Stable
~~~~~~

The latest stable release of the **package** itself can be installed via 

.. code-block:: bash

    pip install pyscenic


Note that pySCENIC needs some prerequisites installed before running ``pip install`` in a new conda environment.
For example:

.. code-block:: bash

    conda create -y -n pyscenic python=3.10
    conda activate pyscenic

    pip install pyscenic


.. caution::
    pySCENIC needs a python 3.7 or greater interpreter.


Development
~~~~~~~~~~~

You can also install the bleeding edge (i.e. less stable) version of the package directly from the source:
 
.. code-block:: bash

    git clone https://github.com/aertslab/pySCENIC.git
    cd pySCENIC/
    pip install .


Containers
~~~~~~~~~~

**pySCENIC containers** are also available for download and immediate use. In this case, no compiling or installation is required, provided either Docker or Singularity software is installed on the user's system.  Images are available from `Docker Hub`_. Usage of the containers is shown below (`Docker and Singularity Images`_).
To pull the docker images, for example:

.. code-block:: bash

    docker pull aertslab/pyscenic:0.12.0

Auxiliary datasets
------------------
To successfully use this pipeline you also need **auxilliary datasets**:

1. *Databases ranking the whole genome* of your species of interest based on regulatory features (i.e. transcription factors). Ranking databases are typically stored in the feather_ format and can be downloaded from cisTargetDBs_.
2. *Motif annotation* database providing the missing link between an enriched motif and the transcription factor that binds this motif. This pipeline needs a TSV text file where every line represents a particular annotation.

=======================  ==========================
  Annotations             Species
=======================  ==========================
`HGNC annotations`_       Homo sapiens
`MGI annotations`_        Mus musculus
`Flybase annotations`_    Drosophila melanogaster
=======================  ==========================

.. _`HGNC annotations`: https://resources.aertslab.org/cistarget/motif2tf/motifs-v9-nr.hgnc-m0.001-o0.0.tbl
.. _`MGI annotations`: https://resources.aertslab.org/cistarget/motif2tf/motifs-v9-nr.mgi-m0.001-o0.0.tbl
.. _`Flybase annotations`: https://resources.aertslab.org/cistarget/motif2tf/motifs-v9-nr.flybase-m0.001-o0.0.tbl


.. caution::
    These ranking databases are 1.1 Gb each so downloading them might take a while. An annotations file is typically 100Mb in size.

A list of transcription factors is required for the network inference step (GENIE3/GRNBoost2).
These lists can be downloaded from `resources section on GitHub <https://github.com/aertslab/pySCENIC/tree/master/resources>`_.


Command Line Interface
----------------------

A command line version of the tool is included. This tool is available after proper installation of the package via :code:`pip`.

.. code-block:: bash

    $ pyscenic -h
    usage: pyscenic [-h] {grn,add_cor,ctx,aucell} ...

    Single-CEll regulatory Network Inference and Clustering (0.12.0)

    positional arguments:
      {grn,add_cor,ctx,aucell}
                            sub-command help
        grn                 Derive co-expression modules from expression matrix.
        add_cor             [Optional] Add Pearson correlations based on TF-gene
                            expression to the network adjacencies output from the
                            GRN step, and output these to a new adjacencies file.
                            This will normally be done during the "ctx" step.
        ctx                 Find enriched motifs for a gene signature and
                            optionally prune targets from this signature based on
                            cis-regulatory cues.
        aucell              Quantify activity of gene signatures across single
                            cells.

    optional arguments:
      -h, --help            show this help message and exit

    Arguments can be read from file using a @args.txt construct. For more
    information on loom file format see http://loompy.org . For more information
    on gmt file format see https://software.broadinstitute.org/cancer/software/gse
    a/wiki/index.php/Data_formats .


Docker and Singularity Images
-----------------------------

pySCENIC is available to use with both Docker and Singularity, and tool usage from a container is similar to that of the command line interface.
Note that the feather databases, transcription factors, and motif annotation databases need to be accessible to the container via a mounted volume.
In the below examples, a single volume mount is used for simplicity, which will contains the input, output, and databases files.

For additional usage examples, see the documentation associated with the `SCENIC protocol <https://github.com/aertslab/SCENICprotocol/blob/master/docs/installation.md>`_ Nextflow implementation.

Docker
~~~~~~

Docker images are available from `Docker Hub`_, and can be obtained by running :code:`docker pull aertslab/pyscenic:[version]`, with the version tag as the latest release.

To run pySCENIC using Docker, use the following three steps.
A mount point (or more than one) needs to be specified, which contains the input data and necessary resources).

.. code-block:: bash

    docker run -it --rm \
        -v /data:/data \
        aertslab/pyscenic:0.12.0 pyscenic grn \
            --num_workers 6 \
            -o /data/expr_mat.adjacencies.tsv \
            /data/expr_mat.tsv \
            /data/allTFs_hg38.txt

    docker run -it --rm \
        -v /data:/data \
        aertslab/pyscenic:0.12.0 pyscenic ctx \
            /data/expr_mat.adjacencies.tsv \
            /data/hg19-tss-centered-5kb-7species.mc9nr.genes_vs_motifs.rankings.feather \
            /data/hg19-tss-centered-10kb-7species.mc9nr.genes_vs_motifs.rankings.feather \
            --annotations_fname /data/motifs-v9-nr.hgnc-m0.001-o0.0.tbl \
            --expression_mtx_fname /data/expr_mat.tsv \
            --mode "dask_multiprocessing" \
            --output /data/regulons.csv \
            --num_workers 6

    docker run -it --rm \
        -v /data:/data \
        aertslab/pyscenic:0.12.0 pyscenic aucell \
            /data/expr_mat.tsv \
            /data/regulons.csv \
            -o /data/auc_mtx.csv \
            --num_workers 6

Singularity
~~~~~~~~~~~

As of release :code:`0.9.19`, pySCENIC Singularity images are no longer being built on `Singularity Hub`_, however images can easily be built using Docker Hub as a source:

.. code-block:: bash

    singularity build aertslab-pyscenic-0.10.0.sif docker://aertslab/pyscenic:0.10.0


To run pySCENIC with Singularity, the usage is very similar to that of Docker.
Note that in Singularity 3.0+, the mount points are automatically overlaid, but bind points can be specified similarly to Docker with :code:`--bind`/:code:`-B`.
The first step (GRN inference) is shown as an example:

.. code-block:: bash

    singularity run aertslab-pyscenic-0.10.0.sif \
        pyscenic grn \
            --num_workers 6 \
            -o expr_mat.adjacencies.tsv \
            expr_mat.tsv \
            allTFs_hg38.txt


Using the Docker or Singularity images with Jupyter notebook
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As of version 0.9.7, the pySCENIC containers have the ``ipykernel`` package installed, and can also be used interactively in a notebook.
This can be achieved using a kernel command similar to the following (for singularity).
Note that in this case, a bind needs to be specified.

.. code-block:: bash

    singularity exec -B /data:/data aertslab-pyscenic-latest.sif ipython kernel -f {connection_file}

More generally, a local or remote kernel can be set up by using the following examples.
These would go in a kernel file in ``~/.local/share/jupyter/kernels/pyscenic-latest/kernel.json`` (for example).

**Remote singularity kernel:**

.. code-block:: bash

    {
      "argv": [
        "/software/jupyter/bin/python",
        "-m",
        "remote_ikernel",
        "--interface",
        "ssh",
        "--host",
        "r23i27n14",
        "--workdir",
        "~/",
        "--kernel_cmd",
        "singularity",
        "exec",
        "-B",
        "/path/to/mounts",
        "/path/to/aertslab-pyscenic-latest.sif",
        "ipython",
        "kernel",
        "-f",
        "{connection_file}"
      ],
      "display_name": "pySCENIC singularity remote",
      "language": "Python"
    }

**Local singularity kernel:**

.. code-block:: bash

    {
        "argv": [
         "singularity",
         "exec",
         "-B",
         "/path/to/mounts",
         "/path/to/aertslab-pyscenic-latest.sif",
         "ipython",
         "kernel",
         "-f",
         "{connection_file}"
        ],
        "display_name": "pySCENIC singularity local",
        "language": "python"
    }


Nextflow
--------

There are two Nextflow implementations available:

* `SCENICprotocol`_: A Nextflow DSL1 implementation.
* `VSNPipelines`_: A Nextflow DSL2 implementation.


.. _`Singularity Hub`: https://www.singularity-hub.org/collections/2033
.. _`SCENICprotocol`: https://github.com/aertslab/SCENICprotocol
.. _`VSNPipelines`: https://github.com/vib-singlecell-nf/vsn-pipelines
.. _dask: https://dask.pydata.org/en/latest/
.. _distributed: https://distributed.readthedocs.io/en/latest/
.. _`Docker Hub`: https://hub.docker.com/r/aertslab/pyscenic
.. _feather: https://github.com/wesm/feather
.. _cisTargetDBs: https://resources.aertslab.org/cistarget/

