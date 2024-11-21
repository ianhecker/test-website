---
layout: post
author: bear
---

# User Observed Txn Latency

For a transaction t submitted by client *c*, the transaction latency time is 
the wall clock time from when *c* submits *t* to the point at which *c* performs 
a strong verification with *t*. Such a time is a measurement of what a client 
observes as the finalization time for their transaction.

For this first iteration of txn latency experiments we are interested in the 
"hugging" time. Meaning, the txn latency for a client running as close to our 
system services as possible. Thus, the program(s) conducting the txn latency 
measurements should be running on an EC2 instance (of adequate size and resources) 
that is located in the same region as the EKS cluster running our ZPR services.

## Test Configuration
Here we detail the configurations for the different testing components.

### ZPR Services

```terraform
locals {
  deployment_region     = "us-east-1"                 # The AWS region to deploy the cluster in
  cluster_name          = "txn-latency"  #
  docker_image_name     = "blockymc/zprsvc:latest"   # The namespace/repo:tag DockerHub image name for the ZPR service version to deploy
  default_zpr_log_level = "debug"                     # The default log level for all ZPR services. For more granular control, see zpr_service_log_levels below
  deploy_cluster        = true                       # Must be applied as true before deploying zpr
  deploy_zpr            = true                       # Must be applied as false before destroying cluster
}
```

```toml
[zpr.storage]
required_piece_count = 3
retry_count = 10
buckets = [
    { provider = "AWS",   cred_id = "zpr-aws",      name = "zpr-dev-east1-001", region = "us-east-1" },
    { provider = "AWS",   cred_id = "zpr-aws2",     name = "zpr-dev-east2-001", region = "us-east-2" },
    { provider = "Azure", cred_id = "zpr-azure",    name = "zpr-dev-east1-001" },
    { provider = "GCP",   cred_id = "zpr-gcp",      name = "zpr-dev-northamerica-northeast1-001" },
    { provider = "GCP",   cred_id = "zpr-gcp",      name = "zpr-dev-east1-001" },
    { provider = "GCP",   cred_id = "zpr-gcp",      name = "zpr-dev-east4-001" },
]
```

Above is the terraform and bucket configuration used by the ZPR service deployments.
Note that at the time of testing the ZPR commit hash for `zprsvc:latest` 
was: bd7cd73627248e663678be8c14db3a72d8eadd28

The list of Atlantis Deploy commit hashes for the ZPR service deployments is:
- 52680013e59fa9253d4f2d00178e74d038a23b23
- 0736b7d615c60013d8cfbb8bbe78a80ee26f1d3c
- 7c0826e02bd7d499d35ee1178b4bac2ed21d4f61
- 4648109b8cc66c9f8c03411d0e02b96f047870ea
- d51e4464eac9a5918cfd8ed464d5ef460bcd955e
- 7f1f6422fc7ff05dba6b1af2f9673181f5d96814
- d9e140991c8fdde9ef2acf47e02a06b68715d0d6
- 195e5f2251bb5176c5138728ede5adb9de3f4a14

### Traffic Generation
Traffic was generated using K6 and the `tps.js` script in `zpr-benchmark` at
commit hash: 1d1cc6c1ea3615507287c8339be3941b2e006710

The script was run on an EC2 instance with the following configuration:
- i-01e849d49e2244ce5 (bear-dev-k6)
- us-east-1
- c3.8xlarge
- 32 vCPU
- 16GB
- 10 Gigabit

### Txn Latency Client
Transaction latency was measured using the `zpr/cmd/kit/cmd/latency.go` command
at ZPR commit hash: b29b1e578478daf470d9710bfe2ff540b758a17a

The command was run on an EC2 instance with the following configuration:
- i-0eb6daf3fa5ae609d (bucket-test)
- us-east-1
- t3.xlarge
- 4 vCPU
- 16GB
- Up to 5 Gigabit

Furthermore, the command was configured to run with Watcher and OMC wait values:
- 100ms bucket-to-queue
- 100ms queue-to-cache
- 50ms continuous OMC

## Experimental Results

We report the following "descriptive statistics" for user-observed transaction
latency. Unless otherwise specified, the unit of measurement is milliseconds.

1. min
2. max
3. mean
4. variance
5. p(25)
6. p(50)
7. p(75)
8. p(90)
9. p(95)
10. #(trials): number of trials
11. #(x=inf): number of trials with value inf

Note that *inf* times should not be included in metrics 1-9.

### EXP-k-latency

For k = {0, 10, 100, 250, 500, 750, 1000, 2000}, we generate traffic at rate of
(about) *k* tx/s*. Separately, we submit a transaction and perform strong verify
locally. If a transaction was not strong verified within 30 seconds, the trial
was marked as *inf* and not included in the metric calculations.

| Test     | #(trials) | #(x=inf) | Min | Max  | Mean | Variance | p(25) | p(50) | p(75) | p(90) | p(95) |
|----------|-----------|----------|-----|------|------|----------|-------|-------|-------|-------|-------|
| 0 tps    | 250       | 1        | 663 | 1727 | 1022 | 23447    | 920   | 1000  | 1090  | 1154  | 1231  |
| 10 tps   | 250       | 0        | 561 | 1919 | 1017 | 23484    | 925   | 1000  | 1087  | 1182  | 1230  |
| 100 tps  | 250       | 0        | 679 | 1725 | 1014 | 18836    | 928   | 1007  | 1083  | 1153  | 1231  |
| 250 tps  | 250       | 0        | 620 | 1772 | 1015 | 25190    | 925   | 1006  | 1094  | 1196  | 1232  |
| 500 tps  | 250       | 0        | 597 | 5559 | 1092 | 157449   | 929   | 1030  | 1133  | 1327  | 1583  |
| 750 tps  | 250       | 0        | 619 | 2101 | 1093 | 61637    | 932   | 1033  | 1214  | 1450  | 1640  |
| 1000 tps | 250       | 0        | 611 | 2151 | 1047 | 46236    | 931   | 1021  | 1123  | 1232  | 1476  |
| 2000 tps | 170       | 1        | 662 | 2374 | 1320 | 239238   | 931   | 1082  | 1821  | 2108  | 2146  |
