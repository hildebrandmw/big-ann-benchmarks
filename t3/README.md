# T3 Track

## Table Of Contents

- [Introduction](#introduction)  
- [For Participants](#for_participants) 
  - [Getting Started](#getting_started) 
  - [Starting Your Development](#starting_your_development)
  - [Developing Your Dockerfile](#developing_your_dockerfile)
  - [Developing Your Algorithm](#developing_your_algorithm) 
  - [Submitting Your Algorithm](#submitting_your_algorithm) 
  - [How To Get Help](#how_to_get_help)
  - [Leaderboard Ranking](#leaderboard_ranking)
    - [Baseline Thresholds](#baseline_thresholds)
    - [Recall Leaderboard](#recall_leaderboard)
    - [Throughput Leaderboard](#throughput_leaderboard)
    - [Power Leaderboard](#power_leaderboard)
    - [Cost Leaderboard](#cost_leaderboard)
- [For Evaluators](#for_organizers)  
  - [Evaluating Participant Algorithms](#evaluating_participant_algorithms)
    - [Participant Sends Hardware To Evaluators](#participant_sends_hardware_to_organizers)
    - [Participant Gives Remote Access To Evaluators](#participant_gives_remote_access_to_organizer)
    - [Participant Runs And Submits Benchmarks](#participant_runs_and_submits_benchmark)
  - [Evaluating Power Consumption](#evaluating_power_consumption)   
- [Appendix](#appendix)

## Introduction

The T1 and T2 tracks of the competition restrict the evaluation of algorithms to standard Azure CPU servers with 64GB of RAM and 2TB of SSD.  The only restriction in the T3 track is that the evaluation machine can be any hardware that is commercially available ( including any commercially available add-on PCIe boards ).  T3 will maintain four leaderboards:
* One based on recall
* One based on throughput
* One based on power consumption
* One based on hardware cost

Participants must submit their algorithm via a pull request and index file(s) upload (one per participating dataset).  Participants are not required to submit proprietary source code such as software drivers or firmware.

Competition evaluators will evaluate the participant's algorithm and hardware via one of these options:
* Participants send their hardware to the organizers at the participant's expense.
* Participants give organizers remote access to the hardware.
* Participants run the evaluation benchmarks on their own, and send the results to the organizers.

## For_Participants

### Requirements

You will need the following installed on your machine:
* Python ( we tested with Anaconda using an environment created for Python version 3.8.5 )
* Note that we tested everything on Ubuntu Linux 18.04 but other environments should be possible.
* It's assumed that all the software drivers and services need to support your hardware are installed on development machines.  For example, to run the T3 baseline, your system must have a Cuda 11 compatibile GPU, Cuda 11.0, and the cuda 11.0 docker run-time installed.  See the T3 baseline [installation instructions](faiss_t3/README.md). Cuda versions greater than 11.0 should be possible, but weren't tested.

### Getting_Started

This section will present a small tutorial about how to use this framework and several of the key scripts you will use throughout the development of your algorithm and eventual submission.

First, clone this repository and cd into the project directory:
```
git clone <REPO_URL>
```
Install the python package requirements:
```
pip install -r requirements.txt
```
Create a small, sample dataset:
```
python create_dataset.py --dataset random-xs
```
Build the docker container for the T3 baseline:
```
python install.py --dockerfile t3/faiss_t3/Dockerfile
```
Run a benchmark evaluation using the algorithm's definition file:
```
python run.py --t3 --definitions t3/faiss_t3/algos.yaml --dataset random-xs
```
Please note that the *--t3* flag is important.  

Now analyze the results:
```
python plot.py --definitions t3/faiss_t3/algos.yaml --dataset random-xs
```
This will place a plot of the algorithms performance, recall-vs-throughput, into the *results/* directory.

### Starting_Your_Development

First, please create a short name for your team without spaces or special characters.  Henceforth in these instructions, this will be referenced as [your_team_name].

Create a custom branch off main in this repository:
```
git checkout -b t3/[your_team_name]
```
In the *t3/* directory, create a sub-directory using that name.
```
mkdir t3/[your_team_name]
```

### Developing_Your_Dockerfile

This framework evaluates algorithms in Docker containers by default.  Your algorithm's Dockerfile should live in your team's subdirectory at *t3/[your_team_name]*.  Ideally, your Docker file should contain everything needed to install and run your algorithm on a system with the same hardware.  Given the nature of T3, this will not likely be entirely possible since custom hardware host drivers and certain low level host libraries require an installation step outside of what can be accomplished with Docker alone.  Please make your best effort to include as much installation and setup within your Docker container, as we want to promote as much transparency as possible among all participants.

Please consult the Dockerfile [here](faiss_t3/Dockerfile) for an example.

To build your Docker container, run:
```
python install.py --dockerfile t3/[your_team_name]/Dockerfile
```

### Developing_Your_Algorithm

Develop and add your algorithm's python class to the [benchmark/algorithms](../benchmark/algorithms) directory.
* You will need to subclass from the [BaseANN class](../benchmark/algorithms/base.py) and implement the functions of that parent class.
* You should consult the examples already in the directory.

As you develop and test your algorithm, you will likely need to test on smaller datasets.  This framework provides a way to create datasets of various sizes.  For example, to create a dataset with 10000 20-dimensional random floating point vectors, run:
```
python create_dataset.py --dataset random-xs
```
To see a complete list of datasets, run the following:
```
python create_dataset.py --help
```
When you are ready to test on the competition datasets, use the create_dataset.py script as follows:
```
python create_dataset.py --dataset [sift-1B|bigann-1B|text2image-1B|msturing-1B|msspacev-1B|ssnpp-1B]
```
To benchmark your algorithm, first create an algorithm configuration yaml in your teams directory called *algos.yaml.*  This file contains the index build parameters and query parameters that will get passed to your algorithm at run-time.  Please look at [this example](faiss_t3/algos.yaml).

Now you can benchmark your algorithm using the run.py script:
```
python run.py --t3  --definitions t3/[your_team_name]/algos.yaml --dataset random-xs
```
This will write the results to the toplevel [results](../results) directory.

Now you can analyze the results by running:
```
python plot.py --definitions t3/[your_team_name]/algos.yaml --dataset random-xs
```
This will place a plot of the algorithms performance, recall-vs-throughput, into the toplevel [results](../results) directory.

The plot.py script supports other benchmarks.  To see a complete list, run:
```
python plot.py --help
```

### Submitting_Your_Algorithm

A submission is composed of the following:
* 1 index binary file(s) for each dataset for which you are participating (see *Index File* section below.)
* 1 *algos.yaml* with only one set of build parameters and at most 10 sets of query parameters for each dataset in which you are participating. Please put that file into the *t3/[your_team_name]/* directory.
* Your algorithm's python class ( placed in the [benchmark/algorithms/](../benchmark/algorithms) directory.)
* 1 README file with specific information about your hardware and software (see *README File* section below.)
* Evidence of the cost of your hardware components (see *README File* section below.)
* Optional information (see *Optional Information* section below.)

### Index File

The binary index file(s) must be http or azcopy accessible and is referenced within your *t3/[your_team_name]/algos.yaml* config file.  Please see the *t3/faiss_t3/algos.yaml* example.

### The README File

Your submission's top-level directory should contain a README.md with the following sections:
* **Hardware Configuration And Cost**  This section must contain a table that breaks down the hardware components of your system and the cost.  Each entry should link to evidence of the component cost.  
* **Hardware Access** This section describes how evaluators acquire access to the hardware (specific instructions or contact information.)
* **No Source Declarations**  This section must contain a list of software components that were not provided with the submission.
* **Hardware Setup And Software Installation**  This section should contain any hardware and software installation instructions.

Please consult the *t3/faiss_t3/README.md* example.

### Optional Information

Please feel free to append sections to the base README requirements.  For example, you can include other benchmarks of interest.

### How_To_Get_Help

There are several ways to get help as you develop your algorithm using this framework:
* You can submit an issue at this github repository.
* Send an email to the competition's T3 organizer, gwilliams@gsitechnology.com
* Send en email to the competition's googlegroup, big-ann-organizers@googlegroups.com

### Leaderboard_Ranking

T3 will maintain four different leaderboards 1) one based on recall 2) one based on throughput 3) one based on power consumption and 4) one based on cost.  The details of the ranking metrics are described here.

#### Baseline_Thresholds

Thresholds of performance have been put in place for this competition, based on both queries per second (qps) and recall measured as recall@10.  For the recall leaderboard, we will rank participants by recall@10 at 2K qps.  The table below shows the baseline recall@10 for all the (knn search type) datasets near 2K qps.

|   dataset    |    qps   | recall@10 |
| ------------ | -------- | --------- |
| msturing-1B  | 2011.542 |   0.910   |
| bigann-1B    | 2058.950 |   0.927   |
| text2image-1B| 2120.635 |   0.860   |
| deep-1B      | 2002.490 |   0.942   |
| msspacev-1B  | 2190.829 |   0.850   |

For the throughput leaderboard, we will rank participants by qps at 90% recall@10. The table below shows the baseline throughput for all the (knn search type) datasets near 90% recall@10.

|   dataset    |    qps   | recall@10 |
| ------------ | -------- | --------- |
| msturing-1B  | 2421.856 |   0.902   |
| bigann-1B    | 2186.755 |   0.905   |
| text2image-1B| 1510.624 |   0.882   |
| deep-1B      | 3422.473 |   0.916   |
| msspacev-1B  | 1484.217 |   0.869   |

Baseline thresholds were measured on an 56 core Intel Xeon system with 700GB RAM and a V100 Nvidia GPU using the FAISS library using the index strategy called IVF1048576,SQ8.  More information can be found in the Appendix at the end of this README.

Here are FAISS baseline recall@10 vs throughput plots for the (knn search type) datasets:
* [msturing-1B](faiss_t3/baseline_plots/msturing-1B-r-vs-t.png)
* [bigann-1B](faiss_t3/baseline_plots/bigann-1B-r-vs-t.png)
* [text2image-1B](faiss_t3/baseline_plots/text2image-1B-r-vs-t.png)
* [deep-1B](faiss_t3/baseline_plots/deep-1B-r-vs-t.png)
* [msspacev-1B](faiss_t3/baseline_plots/msspacev-1B-r-vs-t.png)

Note these plots were acquired using this repo's eval framework.  The baseline thresholds we performed using an old (not obsolete) software framework (see Appendix for more information.)

#### Recall_Leaderboard

This leaderboard leverages the standard recall@10 vs throughput benchmark that has become a standard benchmark when evaluating and comparing approximate nearest neighbor algorithms.  We will rank participants based on recall@10 at 2K qps por each dataset.  The evaluation framework allows for 10 different search parameter sets and we will use the best value of recall@10 from the set.

The final ranking will be based on a computed score, which is the sum of the improvements in recall over the baseline for the participating databases.  A submission must participate in at least 3 databases.

Participants that cannot meet or exceed the baseline qps threshold for a dataset will be dropped from ranking consideration for that dataset.

#### Throughput_Leaderboard

This leaderboard also leverages the standard recall@10 vs throughput benchmark.  We will rank participants based on throughput (qps) at the recall@10 threshold of 90%.  The evaluation framework allows for 10 different search parameter sets and we will use the best value of throughput from the set.

The final ranking will be based on a computed score, which is the sum of the improvements in throughput over the baseline for the participating databases.  A submission must participate in at least 3 databases.

Participants that cannot meet or exceed the baseline recall@10 threshold for a dataset will be dropped from ranking consideration for that dataset.

#### Power_Leaderboard

This leaderboard is related to power consumption, which is an important consideration when scaling applications and servers in a datacenter.  The primary ranking metric is ( kilowatt-hour / query.)  Participants must meet or exceed the recall@10 of the baseline threshold. The reason for those minimum thresholds is to discourage algorithm’s designers from purposefully sacrificing too much performance in order to lower the power consumption.

The evaluation framework leverages the power sensors available in the standard IPMI power management interface of most commercial server chassis’.  We also leverage the open source project ipmicap ( https://github.com/fractalsproject/ipmicap ) to capture the power sensors and calculate the power consumption.

During evaluation, for each search parameter set, power consumption is acquired over at least 10 seconds running search on the entire query set.  During that 10 seconds, multiple consecutive runs on the query set may occur in order to maintain a minimum duration of 10 seconds.  Also, the duration may be greater than 10 seconds if a run of 1 query set takes longer than 10 seconds.  So a run could be composed of 1 batch query or several and the duration will be at least 10 seconds  The power consumption acquired for the run is divided by the total number of queries performed during the run, resulting in ( kilowatt-hour / query ).  Up to 10 search parameter sets are allowed, and we use the minimum value for ranking participants, for each dataset.

The final ranking will be based on a computed score, which is the sum of the improvements in power consumption over the baseline for the participating databases.  A submission must participate in at least 3 databases.

Participants that cannot meet or exceed the recall@10 baseline threshold for a dataset will be dropped from ranking consideration for that dataset.

Here are all the baseline recall@10 vs watt-seconds/query plots for the (knn search type) datasets:
* [msturing-1B](faiss_t3/baseline_plots/msturing-1B-r-vs-p.png)
* [bigann-1B](faiss_t3/baseline_plots/bigann-1B-r-vs-p.png)
* [text2image-1B](faiss_t3/baseline_plots/text2image-1B-r-vs-p.png)
* [deep-1B](faiss_t3/baseline_plots/deep-1B-r-vs-p.png)
* [msspacev-1B](faiss_t3/baseline_plots/msspacev-1B-r-vs-p.png)

#### Cost_Leaderboard

This leaderboard is related to cost, which is an important consideration when scaling applications and servers in a datacenter.  The primary ranking metric will be an estimate of capital expense (capex) + operational expense (opex) that is required to scale the participant’s system to 100,000 qps that meets or exceeds the baseline recall@10.

The formula for the capex estimate is as follows:

capex = (MSRP of all the hardware components of the system ) X ( minimum number of systems needed to scale to support 100,000 qps )

The hardware components include the chassis and all of the electronics within the chassis including the power supplies, motherboard, HDD/SSD, and all extension boards.  Participants must provide evidence of MSRP of components ( either published on a web-site or a copy of a invoice/receipt with customer identifiable information removed. )  Volume based pricing is not considered.

The formula for the opex estimate is as follows:

opex = ( max qps at or greater than the baseline recall @10 threshold ) X ( kilowatt-hour / query ) X ( seconds / hour ) X ( hours / year) X ( 5 years ) X ( dollars / kilowatt-hour ) X ( minimum number of systems needed to scale to support 100,000 qps )

Notes on this formula:
* We will use the maximum qps actually measured that meets or exceeds the baseline recall@10 threshold across all query set parameters.
* We do not account for the cost related to the physical footprint of the system(s) such as the cost of the space occupied by the system(s) in the datacenter.
* We assume linear horizontal scalability of systems with zero cost.  In other words, we do not account for the costs associated when actually clustering multiple systems needed to obtain 100,000 qps ( networking equipment, costs due to routing traffic among systems, costs due to merging results, etc. ) 
* We will use $0.10 / kilowatt-hour for the power consumption cost.
* 5 years is the standard hardware depreciation schedule used for tax purposes with the Internal Revenue Service
* We’d like to thank David Rensin, former Senior Director at Google Cloud, now SVP at Pendo.io for his valuable contribution and consultation with respect to the capex and opex formulas.

The final ranking will be based on a computed score, which is the sum of the improvements in cost over the baseline for the participating databases.  A submission must participate in at least 3 databases.

Participants that cannot meet or exceed the baseline thresholds for a dataset will be dropped from ranking consideration for that dataset.

## For_Evaluators

### Evaluating_Participant_Algorithms

How a participant's algorithm is benchmarked will depend on how they registered for the T3 competition, one of these options:
* Participant sent hardware to competition evaluator at participant's expense.
* Participant is giving the competition valuator remote SSH access to their machine.
* Participant will run the evaluation framework on their own and send the benchmark results to the competition evaluator.

Evaluation steps for each option is detailed in the next sections.

### Participant_Sends_Hardware_To_Evaluators

Evaluators will work with participant's that send hardware during competition on-boarding. Hardware will be sent and returned at the participant's expense.

Evaluators and participants will work closely to make sure the hardware is properly installed and configured.

Evaluators may allow remote access to the machines in order to complete the setup, as needed.

### Participant_Gives_Remote_Access_To_Evaluators

Participants give competition evaluators access to remote machines via SSH.

### Participant_Runs_And_Submits_Benchmarks

This is a very special case, and not all participant's will have this option.  In this case, the participant will run the evaluation on their own.  They will export the data to a CSV via the export.py script and send it to the the competition evaluators.  Participants are still required to submit a pull request and upload their best index.

## Evaluating_Power_Consumption

The hardware chassis which houses all the hardware must support the IPMI management interface.

Determine the IP address, port, and authentication credentials of that interface.

Follow the instructions at IPMICAP open-source project ( http://www.github.com/fractalsproject/ipmicap ) to access the IPMI and configure it to listen to an available port number.

Capture the machine IP address of the machine which is running IPMICAP ( it does not have to be the same machine as the target hardware. )

Now run the following for each competition dataset:
```
python run.py --dataset [DATASET] --t3 --definitions [DEFINITION FILE] --powercapture [IPMICAP_MACHINE_IP]:[IPMICAP_LISTEN_PORT]:[TIME_IN_SECONDS]
```
This will monitor power consumption over that period of time ( 10 seconds is a good number ).

You can retrieve a plot of the power consumptions ( measured as watt-seconds/query ) using the plot.py script.

## Appendix

### Baseline Threshold Experiments

The following table lists the full results used to obtain baseline thresholds:

|              dbase|                 QPS|            recall@10|
|-------------------|--------------------|---------------------|
|          bigann-1B|         2186.754570|             0.904860|
|          bigann-1B|         1926.901416|             0.911140|
|          bigann-1B|         1657.226695|             0.919860|
|          bigann-1B|         2058.950046|             0.926560|
|          bigann-1B|         1931.042641|             0.932450|
|          bigann-1B|         1770.748406|             0.937190|
|          bigann-1B|         1609.052224|             0.941330|
|          bigann-1B|         1504.748288|             0.943890|
|      text2image-1B|         2607.779941|             0.834820|
|      text2image-1B|         2456.621393|             0.841845|
|      text2image-1B|         2285.966847|             0.851920|
|      text2image-1B|         2120.635218|             0.860156|
|      text2image-1B|         1917.445903|             0.867244|
|      text2image-1B|         1748.662912|             0.873469|
|      text2image-1B|         1612.313130|             0.878757|
|      text2image-1B|         1510.624227|             0.882487|
|        msspacev-1B|         2465.473370|             0.844805|
|        msspacev-1B|         2190.828587|             0.850205|
|        msspacev-1B|         1935.385102|             0.854864|
|        msspacev-1B|         1931.506970|             0.858998|
|        msspacev-1B|         1748.525911|             0.862437|
|        msspacev-1B|         1585.766679|             0.865152|
|        msspacev-1B|         1477.389358|             0.867912|
|        msspacev-1B|         1484.216732|             0.868812|
|        msturing-1B|         3625.040250|             0.881202|
|        msturing-1B|         3197.403722|             0.888140|
|        msturing-1B|         2907.993722|             0.893669|
|        msturing-1B|         2655.951474|             0.898400|
|        msturing-1B|         2421.855941|             0.902413|
|        msturing-1B|         2233.241641|             0.905846|
|        msturing-1B|         2070.942269|             0.908949|
|        msturing-1B|         2011.542149|             0.910115|
|            deep-1B|         3422.472565|             0.915540|
|            deep-1B|         2732.133452|             0.920430|
|            deep-1B|         2507.486404|             0.927790|
|            deep-1B|         1992.323615|             0.932950|
|            deep-1B|         2037.783443|             0.937940|
|            deep-1B|         2002.489712|             0.941740|
|            deep-1B|         1967.826369|             0.945130|
|            deep-1B|         1874.898854|             0.947430|

These baseline numbers were performed on the machine configuration used for the T3 faiss baseline(see).

An older (now obsolete) code framework was used to determine these thresholds, not the existing evaluation framework so there is no algos.yaml configuration file.
