# Update a VCF header line sample ID

This pipeline is useful when you have a set of single-sample VCFs
in Cloud Storage in which the sample ID in the header needs to be changed.

For example, suppose you have a set of 24 VCFs for a single individual
where each VCF contains variants for a single chromosome (1-22, X, Y). 
Suppose further that the header line of each VCF looks like:

```
#CHROM POS ID REF ALT QUAL FILTER INFO FORMAT SAMPLE-001
```

and you want it to look like:

```
#CHROM POS ID REF ALT QUAL FILTER INFO FORMAT SAMPLE-001-TEST-01
```

The scripts in this example can be used to make this change to a set of VCFs.
The input VCFs can be compressed with gzip or bzip2 or they can be uncompressed.
The output VCFs' compression state will reflect the input VCFs'.

| API Note |
|----------|
| This example demonstrates using the Pipelines API to run custom code *without building a Docker image*. In this example, the scripts are copied into Cloud Storage and then downloaded at pipeline run time.  The docker image used is the stock [Python 2.7 image from dockerhub](https://hub.docker.com/_/python/). |

| API Note |
|----------|
| This example demonstrates the use of non-file input parameters. These parameters (`ORIGINAL_SAMPLE_ID` and `NEW_SAMPLE_ID`) get set in the environment and are then available to the code running in the docker container. |

## (1) Copy scripts to Cloud Storage

```
gsutil cp set_vcf_sample_id.sh differ.py \
  gs://YOUR-BUCKET/pipelines/set_vcf_sample_id/
```

* Replace `YOUR-PROJECT-ID` with your project ID.

## (2) Launch the Pipeline

```
PYTHONPATH=.. python cloud/run_set_vcf_sample_id.py \
  --project YOUR-PROJECT \
  --zones "us-*" \
  --disk-size 100 \ 
  --script-path gs://YOUR-BUCKET/pipelines/set_vcf_sample_id \
  --input gs://genomics-public-data/ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/working/20140123_NA12878_Illumina_Platinum/**.vcf.gz \
  --original-sample-id NA12878 \
  --new-sample-id NA12878-NEW \
  --output gs://YOUR-BUCKET/set_vcf_sample_id/output \
  --logging gs://YOUR-BUCKET/set_vcf_sample_id/logging \
  --poll-interval 20
```

* Replace `YOUR-PROJECT-ID` with your project ID.
* Replace `YOUR-BUCKET` with a bucket in your project.

The `PYTHONPATH` must include the top-level directory of the
`pipelines-api-examples` in order to pick up modules in the
[pipelines_pylib](../pipelines_pylib) directory.

The output will be the JSON description of the operation, followed by periodic
messages for polling. When the operation completes, the full operation will
be emitted.
```
{ u'done': False,
  u'metadata': { u'@type': u'type.googleapis.com/google.genomics.v1.OperationMetadata',
                 u'clientId': u'',
                 u'createTime': u'2016-03-25T22:16:26.000Z',
                 u'events': [],
                 u'projectId': u'YOUR-PROJECT'},
  u'name': u'operations/YOUR-NEW-OPERATION-ID'}

Polling for completion of operation
Operation not complete. Sleeping 20 seconds
Operation not complete. Sleeping 20 seconds
...
Operation not complete. Sleeping 20 seconds

Operation complete

{ u'done': True,
  u'metadata': { u'@type': u'type.googleapis.com/google.genomics.v1.OperationMetadata',
                 u'clientId': u'',
                 u'createTime': u'2016-03-25T22:16:26.000Z',
                 u'endTime': u'2016-03-25T22:19:50.000Z',
                 u'events': [ { u'description': u'start',
                                u'startTime': u'2016-03-25T22:17:38.796730133Z'},
                              { u'description': u'pulling-image',
                                u'startTime': u'2016-03-25T22:17:38.796805264Z'},
                              { u'description': u'localizing-files',
                                u'startTime': u'2016-03-25T22:18:08.830199428Z'},
                              { u'description': u'running-docker',
                                u'startTime': u'2016-03-25T22:18:18.508789255Z'},
                              { u'description': u'delocalizing-files',
                                u'startTime': u'2016-03-25T22:19:43.563854558Z'},
                              { u'description': u'ok',
                                u'startTime': u'2016-03-25T22:19:49.778866602Z'}],
                 u'projectId': u'YOUR-PROJECT',
                 u'request': { u'@type': u'type.googleapis.com/google.genomics.v1alpha2.RunPipelineRequest',
                               u'ephemeralPipeline': { u'description': u'Set the sample ID in a VCF header',
                                                       u'docker': { u'cmd': u'mkdir /mnt/data/output && export SCRIPT_DIR=/mnt/data/scripts && chmod u+x ${SCRIPT_DIR}/* && ${SCRIPT_DIR}/set_vcf_sample_id.sh "${ORIGINAL_SAMPLE_ID}" "${NEW_SAMPLE_ID}" "/mnt/data/input/*" "/mnt/data/output"',
                                                                    u'imageName': u'python:2.7'},
                                                       u'name': u'set_vcf_sample_id',
                                                       u'parameters': [ { u'description': u'Cloud Storage path to input file(s)',
                                                                          u'name': u'inputFile0'},
                                                                        { u'description': u'Cloud Storage path to set_vcf_sample_id.sh script',
                                                                          u'name': u'setVcfSampleId_Script'},
                                                                        { u'description': u'Cloud Storage path to differ.py script',
                                                                          u'name': u'differ_Script'},
                                                                        { u'description': u'ID which must already appear in the VCF (default "none")',
                                                                          u'name': u'ORIGINAL_SAMPLE_ID'},
                                                                        { u'description': u'New ID to set in the VCF header',
                                                                          u'name': u'NEW_SAMPLE_ID'},
                                                                        { u'description': u'Cloud Storage path for where to copy the output',
                                                                          u'name': u'outputPath'}],
                                                       u'projectId': u'YOUR-PROJECT',
                                                       u'resources': { u'disks': [ { u'autoDelete': True,
                                                                                     u'name': u'datadisk'}]}},
                               u'pipelineArgs': { u'clientId': u'',
                                                  u'inputs': { u'NEW_SAMPLE_ID': u'NA12878-NEW',
                                                               u'ORIGINAL_SAMPLE_ID': u'NA12878',
                                                               u'inputFile0': u'gs://genomics-public-data/ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/working/20140123_NA12878_Illumina_Platinum/**.vcf.gz'},
                                                  u'logging': { u'gcsPath': u'gs://YOUR-BUCKET/set_vcf_sample_id/logging'},
                                                  u'outputs': { u'outputPath': u'gs://YOUR-BUCKET/set_vcf_sample_id/output'},
                                                  u'projectId': u'YOUR-PROJECT',
                                                  u'resources': { u'bootDiskSizeGb': 0,
                                                                  u'disks': [ { u'autoDelete': True,
                                                                                u'mountPoint': u'',
                                                                                u'name': u'datadisk',
                                                                                u'readOnly': False,
                                                                                u'sizeGb': 100,
                                                                                u'source': u'',
                                                                                u'type': u'TYPE_UNSPECIFIED'}],
                                                                  u'minimumCpuCores': 0,
                                                                  u'minimumRamGb': 1,
                                                                  u'preemptible': False,
                                                                  u'zones': [ u'us-central1-a',
                                                                              u'us-central1-b',
                                                                              u'us-central1-c',
                                                                              u'us-central1-f',
                                                                              u'us-east1-b',
                                                                              u'us-east1-c',
                                                                              u'us-east1-d']},
                                                  u'serviceAccount': { u'email': u'default',
                                                                       u'scopes': [ u'https://www.googleapis.com/auth/compute',
                                                                                    u'https://www.googleapis.com/auth/devstorage.full_control',
                                                                                    u'https://www.googleapis.com/auth/genomics']}}},
                 u'startTime': u'2016-03-25T22:16:55.000Z'},
  u'name': u'operations/YOUR-NEW-OPERATION-ID'}
```

## (3) Check the results

Check the operation output for a top-level `errors` field.
If none, then the operation should have finished successfully.

## (4) Check that the output exists

```
$ gsutil ls -l gs://YOUR-BUCKET/set_vcf_sample_id/output

   5055692  2016-03-25T22:19:46Z  gs://YOUR-BUCKET/set_vcf_sample_id/output/NA12878.wgs.illumina_platinum.20140122.indel.genotypes.vcf.gz
  29162279  2016-03-25T22:19:47Z  gs://YOUR-BUCKET/set_vcf_sample_id/output/NA12878.wgs.illumina_platinum.20140122.snp.genotypes.vcf.gz
   5243803  2016-03-25T22:19:46Z  gs://YOUR-BUCKET/set_vcf_sample_id/output/NA12878.wgs.illumina_platinum.20140404.indels_v2.vcf.gz
  29014774  2016-03-25T22:19:47Z  gs://YOUR-BUCKET/set_vcf_sample_id/output/NA12878.wgs.illumina_platinum.20140404.snps_v2.vcf.gz
     31103  2016-03-25T22:19:45Z  gs://YOUR-BUCKET/set_vcf_sample_id/output/NA12878.wgs.illumina_platinum.20140404.svs_v2.vcf.gz
TOTAL: 5 objects, 68507651 bytes (65.33 MiB)
```

## (5) Check the header in the output

```
$ gsutil cat gs://YOUR-BUCKET/set_vcf_sample_id/output/* \
 | zcat \
 | grep ^#CHROM
#CHROM  POS ID  REF ALT QUAL  FILTER  INFO  FORMAT  NA12878-NEW
#CHROM  POS ID  REF ALT QUAL  FILTER  INFO  FORMAT  NA12878-NEW
#CHROM  POS ID  REF ALT QUAL  FILTER  INFO  FORMAT  NA12878-NEW
#CHROM  POS ID  REF ALT QUAL  FILTER  INFO  FORMAT  NA12878-NEW
#CHROM  POS ID  REF ALT QUAL  FILTER  INFO  FORMAT  NA12878-NEW
```
