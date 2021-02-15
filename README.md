# DeepVariant

[![release](https://img.shields.io/badge/release-v1.1.0-green?logo=github)](https://github.com/google/deepvariant/releases)
[![announcements](https://img.shields.io/badge/announcements-blue)](https://groups.google.com/d/forum/deepvariant-announcements)
[![blog](https://img.shields.io/badge/blog-orange)](https://goo.gl/deepvariant)

DeepVariant is a deep learning-based variant caller that takes aligned reads (in
BAM or CRAM format), produces pileup image tensors from them, classifies each
tensor using a convolutional neural network, and finally reports the results in
a standard VCF or gVCF file.

DeepVariant supports germline variant-calling in diploid organisms.

*   NGS (Illumina) data for either a
    [whole genome](docs/deepvariant-case-study.md) or
    [whole exome](docs/deepvariant-exome-case-study.md).
*   PacBio HiFi data, see the
    [PacBio case study](docs/deepvariant-pacbio-model-case-study.md).
*   Hybrid PacBio HiFi + Illumina WGS, see the
    [hybrid case study](docs/deepvariant-hybrid-case-study.md).
*   Oxford Nanopore long-read data by using
    [PEPPER-DeepVariant](https://github.com/kishwarshafin/pepper/blob/master/docs/PEPPER_variant_calling.md).
*   GenapSys data, by using a
    [model retrained by GenapSys](https://github.com/GenapsysInc/genapsys_deepvariant/blob/master/docs/GenapSys_DeepVariant_WES_Model.md).

Please also note:

*   For somatic data or any other samples where the genotypes go beyond two
    copies of DNA, DeepVariant will not work out of the box because the only
    genotypes supported are hom-alt, het, and hom-ref.
*   The models included with DeepVariant are only trained on human data. For
    other organisms, see the
    [blog post on non-human variant-calling](https://google.github.io/deepvariant/posts/2018-12-05-improved-non-human-variant-calling-using-species-specific-deepvariant-models/)
    for some possible pitfalls and how to handle them.

## DeepTrio

DeepTrio is a deep learning-based trio variant caller built on top of
DeepVariant. DeepTrio extends DeepVariant's functionality, allowing it to
utilize the power of neural networks to predict genomic variants in trios or
duos. See [this page](docs/deeptrio-details.md) for more details and
instructions on how to run DeepTrio.

DeepTrio supports germline variant-calling in diploid organisms for the
following types of input data:

*   NGS (Illumina) data for either
    [whole genome](docs/deeptrio-wgs-case-study.md) or whole exome.
*   PacBio HiFi data, see the
    [PacBio case study](docs/deeptrio-pacbio-case-study.md).

Please also note:

*   All DeepTrio models were trained on human data.
*   It is possible to use DeepTrio with only 2 samples (child, and one parent).
*   External tool [GLNexus](https://github.com/dnanexus-rnd/GLnexus) is used to
    merge output VCFs.

## How to run DeepVariant

We recommend using our Docker solution. The command will look like this:

```
BIN_VERSION="1.1.0"
docker run \
  -v "YOUR_INPUT_DIR":"/input" \
  -v "YOUR_OUTPUT_DIR":"/output" \
  google/deepvariant:"${BIN_VERSION}" \
  /opt/deepvariant/bin/run_deepvariant \
  --model_type=WGS \ **Replace this string with exactly one of the following [WGS,WES,PACBIO,HYBRID_PACBIO_ILLUMINA]**
  --ref=/input/YOUR_REF \
  --reads=/input/YOUR_BAM \
  --output_vcf=/output/YOUR_OUTPUT_VCF \
  --output_gvcf=/output/YOUR_OUTPUT_GVCF \
  --call_variants_extra_args="use_openvino=true" \ **Optional. Setting this will use OpenVINO on Intel CPUs, which empirically reduces call_variants runtime by 15%-25%.
  --num_shards=$(nproc) \ **This will use all your cores to run make_examples. Feel free to change.**
  --logging_dir=/output/logs **Optional. This saves the log output for each stage separately and enables runtime profiling.
```

NOTE: `--call_variants_extra_args="use_openvino=true"` is added in 1.1.0. We are
considering setting this as default for CPU in the future. If you have any
questions or feedback, feel free to
[open an issue](https://github.com/google/deepvariant/issues/new).

To see all flags you can use, run: `docker run
google/deepvariant:"${BIN_VERSION}"`

If you're using GPUs, or want to use Singularity instead, see
[Quick Start](docs/deepvariant-quick-start.md) for more details or see all the
[setup options](#deepvariant_setup) available including solutions on external
platforms.

For more information, also see:

*   [Full documentation list](docs/README.md)
*   [Detailed usage guide](docs/deepvariant-details.md) with more information on
    the input and output file formats and how to work with them.
*   [Best practices for multi-sample variant calling with DeepVariant](docs/trio-merge-case-study.md)
*   [(Advanced) Training tutorial](docs/deepvariant-training-case-study.md)

## How to cite

If you're using DeepVariant in your work, please cite:

[A universal SNP and small-indel variant caller using deep neural networks. Nature Biotechnology 36, 983–987 (2018).](https://rdcu.be/7Dhl) <br/>
Ryan Poplin, Pi-Chuan Chang, David Alexander, Scott Schwartz, Thomas Colthurst, Alexander Ku, Dan Newburger, Jojo Dijamco, Nam Nguyen, Pegah T. Afshar, Sam S. Gross, Lizzie Dorfman, Cory Y. McLean, and Mark A. DePristo.<br/>
doi: https://doi.org/10.1038/nbt.4235

Additionally, if you are generating multi-sample calls using our
[DeepVariant and GLnexus Best Practices](docs/trio-merge-case-study.md), please
cite:

[Accurate, scalable cohort variant calls using DeepVariant and GLnexus. bioRxiv
10.1101/2020.02.10.942086v1 (2020).](https://www.biorxiv.org/content/10.1101/2020.02.10.942086v1)<br/>
Taedong Yun, Helen Li, Pi-Chuan Chang, Michael F. Lin, Andrew Carroll, and Cory Y.
McLean.<br/>
doi: https://doi.org/10.1101/2020.02.10.942086

## Why Use DeepVariant?

*   **High accuracy** - DeepVariant won 2020
    [PrecisionFDA Truth Challenge V2](https://precision.fda.gov/challenges/10/view/results)
    for All Benchmark Regions for ONT, PacBio, and Multiple Technologies
    categories, and 2016
    [PrecisionFDA Truth Challenge](https://precision.fda.gov/challenges/truth/results)
    for best SNP Performance. DeepVariant maintains high accuracy across data
    from different sequencing technologies, prep methods, and species. For
    [lower coverage](https://google.github.io/deepvariant/posts/2019-09-10-twenty-is-the-new-thirty-comparing-current-and-historical-wgs-accuracy-across-coverage/),
    using DeepVariant makes an especially great difference. See
    [metrics](docs/metrics.md) for the latest accuracy numbers on each of the
    sequencing types.
*   **Flexibility** - Out-of-the-box use for
    [PCR-positive](https://ai.googleblog.com/2018/04/deepvariant-accuracy-improvements-for.html)
    samples and
    [low quality sequencing runs](https://blog.dnanexus.com/2018-01-16-evaluating-the-performance-of-ngs-pipelines-on-noisy-wgs-data/),
    and easy adjustments for
    [different sequencing technologies](https://google.github.io/deepvariant/posts/2019-01-14-highly-accurate-snp-and-indel-calling-on-pacbio-ccs-with-deepvariant/)
    and
    [non-human species](https://google.github.io/deepvariant/posts/2018-12-05-improved-non-human-variant-calling-using-species-specific-deepvariant-models/).
*   **Ease of use** - No filtering is needed beyond setting your preferred
    minimum quality threshold.
*   **Cost effectiveness** - With a single non-preemptible n1-standard-16
    machine on Google Cloud, it costs ~$11.8 to call a 30x whole genome and
    ~$0.89 to call an exome. With preemptible pricing, the cost is $2.84 for a
    30x whole genome and $0.21 for whole exome (not considering preemption).
*   **Speed** - See [metrics](docs/metrics.md) for the runtime of all supported
    datatypes on a 64-core CPU-only machine</sup>. Multiple options for
    acceleration exist, taking the WGS pipeline to as fast as 40 minutes
    (see [external solutions](#external-solutions)).
*   **Usage options** - DeepVariant can be run via Docker or binaries, using
    both on-premise hardware or in the cloud, with support for hardware
    accelerators like GPUs and TPUs.

<a name="myfootnote1">(1)</a>: Time estimates do not include mapping.

## How DeepVariant works

![diagram of stages in DeepVariant](docs/images/inference_flow_diagram.svg)

For more information on the pileup images and how to read them, please see the
["Looking through DeepVariant's Eyes" blog post](https://google.github.io/deepvariant/posts/2020-02-20-looking-through-deepvariants-eyes/).

DeepVariant relies on [Nucleus](https://github.com/google/nucleus), a library of
Python and C++ code for reading and writing data in common genomics file formats
(like SAM and VCF) designed for painless integration with the
[TensorFlow](https://www.tensorflow.org/) machine learning framework. Nucleus
was built with DeepVariant in mind and open-sourced separately so it can be used
by anyone in the genomics research community for other projects. See this blog
post on
[Using Nucleus and TensorFlow for DNA Sequencing Error Correction](https://google.github.io/deepvariant/posts/2019-01-31-using-nucleus-and-tensorflow-for-dna-sequencing-error-correction/).

## DeepVariant Setup

### Prerequisites

*   Unix-like operating system (cannot run on Windows)
*   Python 3.6

### Official Solutions

Below are the official solutions provided by the
[Genomics team in Google Health](https://health.google/health-research/).

Name                                                                                                | Description
:-------------------------------------------------------------------------------------------------: | -----------
[Docker](docs/deepvariant-quick-start.md)           | This is the recommended method.
[Build from source](docs/deepvariant-build-test.md) | DeepVariant comes with scripts to build it on Ubuntu 14 and 16, with Ubuntu 16 recommended. To build and run on other Unix-based systems, you will need to modify these scripts.
Prebuilt Binaries                                                                                   | Available at [`gs://deepvariant/`](https://console.cloud.google.com/storage/browser/deepvariant). These are compiled to use SSE4 and AVX instructions, so you will need a CPU (such as Intel Sandy Bridge) that supports them. You can check the `/proc/cpuinfo` file on your computer, which lists these features under "flags".

### External Solutions

The following pipelines are not created or maintained by the
[Genomics team in Google Health](https://health.google/health-research/).
Please contact the relevant teams if you have any questions or concerns.

Name                                                                                                                                                                                                                                                                                                                                                                                                                     | Description
:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | -----------
[Running DeepVariant on Google Cloud Platform](https://cloud.google.com/genomics/docs/tutorials/deepvariant)                                                                                                                                                                                                                                                                                                             | Docker-based pipelines optimized for cost and speed. Code can be found [here](https://github.com/googlegenomics/gcp-deepvariant-runner).
[DeepVariant-on-spark from ATGENOMIX](https://github.com/atgenomix/deepvariant-on-spark)                                                                                                                                                                                                                                                                                                                                 | A germline short variant calling pipeline that runs DeepVariant on Apache Spark at scale with support for multi-GPU clusters (e.g. NVIDIA DGX-1).
[NVIDIA Clara Parabricks](https://www.nvidia.com/en-us/docs/parabricks/variant-callers/deepvariant/)                                                                                                                                                                                                                                                                                                                     | An accelerated DeepVariant pipeline with multi-GPU support that runs our WGS pipeline in just 40 minutes, at a cost of $2-$3 per sample. This provides a 7.5x speedup over a 64-core CPU-only machine at lower cost.
[DNAnexus DeepVariant App](https://platform.dnanexus.com/app/deepvariant_germline)                                                                                                                                                                                                                                                                                                                                       | Offers parallelized execution with a GUI interface (requires platform account).
[Nextflow Pipeline](https://github.com/nf-core/deepvariant)                                                                                                                                                                                                                                                                                                                                                              | Offers parallel processing of multiple BAMs and Docker support.
[DNAstack Pipeline](https://app.dnastack.com/auth/realms/DNAstack/protocol/openid-connect/auth?client_id=dnastack-client&redirect_uri=https%3A%2F%2Fapp.dnastack.com%2F%3Fredirect_fragment%3D%252Forg%252F473079%252Fproj%252F473096%252Fapp%252Fworkflow%252F425685%252Frun&state=42231553-9fbc-4d71-a10e-d6ce42415c01&nonce=daf2568d-4fe7-48e2-ab60-858937244a87&response_mode=query&response_type=code&scope=openid) | Cost-optimized DeepVariant pipeline (requires platform account).

## Contribution Guidelines

Please [open a pull request](https://github.com/google/deepvariant/compare) if
you wish to contribute to DeepVariant. Note, we have not set up the
infrastructure to merge pull requests externally. If you agree, we will test and
submit the changes internally and mention your contributions in our
[release notes](https://github.com/google/deepvariant/releases). We apologize
for any inconvenience.

If you have any difficulty using DeepVariant, feel free to
[open an issue](https://github.com/google/deepvariant/issues/new). If you have
general questions not specific to DeepVariant, we recommend that you post on a
community discussion forum such as [BioStars](https://www.biostars.org/).

## License

[BSD-3-Clause license](LICENSE)

## Acknowledgements

DeepVariant happily makes use of many open source packages. We would like to
specifically call out a few key ones:

*   [Boost Graph Library](http://www.boost.org/doc/libs/1_65_1/libs/graph/doc/index.html)
*   [abseil-cpp](https://github.com/abseil/abseil-cpp) and
    [abseil-py](https://github.com/abseil/abseil-py)
*   [CLIF](https://github.com/google/clif)
*   [GNU Parallel](https://www.gnu.org/software/parallel/)
*   [htslib & samtools](http://www.htslib.org/)
*   [Nucleus](https://github.com/google/nucleus)
*   [numpy](http://www.numpy.org/)
*   [SSW Library](https://github.com/mengyao/Complete-Striped-Smith-Waterman-Library)
*   [TensorFlow and Slim](https://www.tensorflow.org/)

We thank all of the developers and contributors to these packages for their
work.

## Disclaimer

This is not an official Google product.
