---
layout: post
author: dave
---

# Important metrics

This post is intended to be a first pass at the types of metrics that we are
currently (as of 2023-10-25) collecting and prioritizing in the development of
ZipperChain.

## Finalization Times

We will start with a few definitions. Let *T* be a set of transactions and *u*
be a UUID. A batch can be defined by the tuple *(T, u)*. Recall that *T* will
become the leaves of a Merkle tree with id *u*.  That tree will be in at least
one block (that may or may not be on the main chain). Also recall that if a
batch is "retried" multiple times, each time, *T* and *u* do not change.

We define two types of timing metrics that represent two "extremes" of
performance.

For a batch *(T, u)*, the *batch finality time* is the time between the
first pop of batch to storing, in a publicly accessible location, the first
sequence attestation that transitively attests a tree with id *u*. Such a time
is a measurement of the lowest transaction finalization time for a given
transaction.

For a transaction *t* submitted by client *c*, the *transaction latency time*
is the wall clock time from when *c* submits *t* to the point at which *c*
performs a strong verification with *t*.  Such a time is a measurement of what
a client observes as the finalization time for their transaction.

These are both just definitions for times that can be observed in the context
of our system.  Later, we will define specific experimental designs for
collecting the metrics. For both metrics, we define the time as *inf* if the
"end" event does not occur (e.g. sequence attestation write or strong verify
success).

## Experiments

When reporting "descriptive statistics", unless otherwise specified, in
milliseconds, report:

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

Experiments should be performed on the cluster running on EKS, and traffic
should be produced by one or many systems running in the same region as the
cluster.

### EXP-k-batch-finality

For k = {10, 100, 1000, 10000}, generating traffic at rate of (about) *k tx/s*.
Report descriptive stats for the batch finality for a duration of 300
seconds.

Separately, run `omc length` locally.  The test should terminate as a failure
if the length of the chain has not increased in 10 seconds.

Note that there is nothing special about 300 seconds or 10 seconds.  Some
fiddling may be needed.

### EXP-k-latency

For k = {10, 100, 1000, 10000}, generate traffic at rate of (about) *k* tx/s*.
Separately, submit a transaction and perform strong verify locally.
Report descriptive stats for the latency for 10 transactions.

If a transaction has not been strong verified within 30 seconds, the trial
should be marked as *inf* and killed off.

Note that there is nothing special about 10 transactions or 30 seconds.  It
seems like a starting point for getting a signal, some fiddling may be needed.

## Note from Dave

Here are a few notes to keep in mind:

1. While we have already demonstrated that ZipperChain has capacity well above
   10k txn/s with sub-second batch finality. As of this writing, I have no idea
   what our latency is (after all, I just made it up). It is my belief,
   however, that until latency's p(90) < 1s, processing more
   transactions per second is not particularly interesting.

2. These metrics are just a starting point for things that matter.  Collected
   metrics should be re-assessed at least once an OKR period (ideally before
   the start of the period) so that we are collecting (at least) the metrics
   that are supporting a KR. (We may want to continue to collect metrics
   related to old KRs if they are of interest.)
