---
icon: lucide/network
---

# Technical Overview

## Pipeline Architecture

The data pipeline is divided in three main stages, made of steps, made of tasks.
The whole pipeline runs in an [Airflow DAG](https://github.com/opentargets/orchestration),
but all steps can be run independently on [Otter](https://opentargets.github.io/otter/).

### PIS — Pipeline Input Stage

Acquires data from all external sources and prepares it for the main pipeline stage.

### PTS — Pipeline Transformation Stage

Main stage, transform input data into an Open Targets Release.

#### ETL — Old Scala Transform Stage

This is a legacy part of the pipeline that is being slowly ported into regular
PTS steps.

#### Gentropy

This is a new component of the pipeline that contains what was the old Open Targets
Genetics spin-off web. It will slowly be wrapped by PTS steps too.

### POS — Pipeline Output Stage

This stage takes care of building the different distributions of an Open Targets
Release.
