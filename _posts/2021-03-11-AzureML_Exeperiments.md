---
layout: post
title: Azure Machine Learning experiments
date: 2021-03-11 10:05:00
description: In Azure Machine Learning, an experiment is a named process, usually the running of a script or a pipeline, that can generate metrics and outputs and be tracked in the Azure Machine Learning workspace
tags: Azure ML MLOPs
categories: research Azure AI ML
thumbnail: assets/img/azure_ml_03-01-experiment.jpg
---
# Azure Machine Learning experiments

Like any scientific discipline, data science involves running experiments; typically to explore data or to build and evaluate predictive models. In Azure Machine Learning, an experiment is a named process, usually the running of a script or a pipeline, that can generate metrics and outputs and be tracked in the Azure Machine Learning workspace.

![AzureML Experiment](/assets/img/azure_ml_03-01-experiment.jpg)

An experiment can be run multiple times, with different data, code, or settings; and Azure Machine Learning tracks each run, enabling you to view run history and compare results for each run.

## The Experiment Run Context

When you submit an experiment, you use its run context to initialize and end the experiment run that is tracked in Azure Machine Learning, as shown in the following code sample:

```Python
from azureml.core import Experiment

# create an experiment variable
experiment = Experiment(workspace = ws, name = "my-experiment")

# start the experiment
run = experiment.start_logging()

# experiment code goes here

# end the experiment
run.complete()
```

## Logging Metrics and Creating Outputs

Experiments are most useful when they produce metrics and outputs that can be tracked across runs

### Logging Metrics

Every experiment generates log files that include the messages that would be written to the terminal during interactive execution. This enables you to use simple print statements to write messages to the log. However, if you want to record named metrics for comparison across runs, you can do so by using the Run object; which provides a range of logging functions specifically for this purpose. These include:

- log: Record a single named value.
- log_list: Record a named list of values.
- log_row: Record a row with multiple columns.
- log_table: Record a dictionary as a table.
- log_image: Record an image file or a plot.

For example, following code records the number of observations (records) in a CSV file:

```Python
from azureml.core import Experiment
import pandas as pd

# Create an Azure ML experiment in your workspace
experiment = Experiment(workspace = ws, name = 'my-experiment')

# Start logging data from the experiment
run = experiment.start_logging()

# load the dataset and count the rows
data = pd.read_csv('data.csv')
row_count = (len(data))

# Log the row count
run.log('observations', row_count)

# Complete the experiment
run.complete()

```
