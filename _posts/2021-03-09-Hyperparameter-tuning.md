# Hyperparameter Tuning

In machine learning, models are trained to predict unknown labels for new data based on correlations between known labels and features found in the training data. Depending on the algorithm used, you may need to specify hyperparameters to configure how the model is trained.

For example, the logistic regression algorithm uses a regularization rate hyperparameter to counteract overfitting; and deep learning techniques for convolutional neural networks (CNNs) use hyperparameters like learning rate to control how weights are adjusted during training, and batch size to determine how many data items are included in each training batch.

Data scientists refer to the values determined from the training features as _parameters_, **values that are used to configure training behavior but which are not derived from the training data** are named _hyperparameter_.

The choice of hyperparameter values can significantly affect the resulting model, making it important to select the best possible values.

Hyperparameter tuning is accomplished by training the multiple models, using the same algorithm and training data but different hyperparameter values. The resulting model from each training run is then evaluated to determine the performance metric for which you want to optimize (for example, _accuracy_), and the best-performing model is selected.

In Azure Machine Learning, you achieve this through an _experiment_ that consists of a _hyperdrive run_, which initiates a child run for each hyperparameter combination to be tested. Each child run uses a training script with parameterized hyperparameter values to train a model, and logs the target performance metric achieved by the trained model.

Hyperdrive run configuration must include:

- hyperparameter search space.
- hyperparameter sampling.
- early-termination policy.

## Sampling

### Grid Sampling

Grid Sampling is used to try every possible combination of parameters in the search space.

```Python
from azureml.train.hyperdrive import GridParameterSampling, choice

param_space = {
                 '--batch_size': choice(16, 32, 64),
                 '--learning_rate': choice(0.01, 0.1, 1.0)
              }

param_sampling = GridParameterSampling(param_space)

```

### Random Samplig

Random sampling is used to randomly select a value for each hyperparameter

```Python
from azureml.train.hyperdrive import RandomParameterSampling, choice, normal

param_space = {
                 '--batch_size': choice(16, 32, 64),
                 '--learning_rate': normal(10, 3)
              }

param_sampling = RandomParameterSampling(param_space)

```

### Bayesian sampling 

Bayesian sampling chooses hyperparameter values based on the Bayesian optimization algorithm, which tries to select parameter combinations that will result in improved performance from the previous selection.

```Python
from azureml.train.hyperdrive import BayesianParameterSampling, choice, uniform

param_space = {
                 '--batch_size': choice(16, 32, 64),
                 '--learning_rate': uniform(0.05, 0.1)
              }

param_sampling = BayesianParameterSampling(param_space)
```

 Note: Bayesian sampling can be used only with choice, uniform, and quniform parameter expressions, and can not be combined with an early-termination policy

## Early Termination Policy

With a sufficiently large hyperparameter search space, it could take many iterations (i.e. child runs) to try every possible combination. To prevent wasting time and money, in addition to a maximum number of rins, you can set an **early termination policy** that abandons runs that are unlikely to produce a better result than previously completed runs.

The policy is evaluated at an `evaluation_interval` you specify. You can also set a `delay_evaluation` parameter to avoid evaluating the policy until a minimum number of iterations have been completed.

Early termination is particularly useful for deep learning scenarios where a deep neural network (DNN) is trained iteratively over a number of epochs. The training script can report the target metric after each epoch, and if the run is significantly underperforming previous runs after the same number of intervals, it can be abandoned.

### Bandit Policy

Use a bandit policy to stop a run if the target performance metric underperforms the best run so far by a specified margin expressed as absolute value (`slack_amount`) or factor (`slack_factor`)

This example applies the policy for every iteration after the first five, and abandons runs where the reported target metric is 0.2 or more worse than the best performing run after the same number of intervals.

```Python
from azureml.train.hyperdrive import BanditPolicy

early_termination_policy = BanditPolicy(slack_amount = 0.2,
                                        evaluation_interval=1,
                                        delay_evaluation=5)
```

You can also apply a bandit policy using a slack factor, which compares the performance metric as a ratio rather than an absolute value.

### Median Stopping Policy

A median stopping policy abandons runs where the target performance metric is worse than the median of the running averages for all runs.

```Python
from azureml.train.hyperdrive import MedianStoppingPolicy

early_termination_policy = MedianStoppingPolicy(evaluation_interval=1,
                                                delay_evaluation=5)
```

### Truncation selection policy

A truncation selection policy cancels the lowest performing X% of runs at each evaluation interval based on the truncation_percentage value you specify for X.

```Python
from azureml.train.hyperdrive import TruncationSelectionPolicy

early_termination_policy = TruncationSelectionPolicy(truncation_percentage=10,
                                                     evaluation_interval=1,
                                                     delay_evaluation=5)

```

## Running a hyperparameter tuning experiment

In Azure Machine Learning, you can tune hyperparameters by running a hyperdrive experiment.

### Training script for hyperparameter tuning

To run a hyperdrive experiment, you need to create a training script just the way you would do for any other training experiment, except that your script must:

- Include an argument for each hyperparameter you want to vary.
- Log the target performance metric. This enables the hyperdrive run to evaluate the performance of the child runs it initiates, and identify the one that produces the best performing model.

You can found an example of HyperDrive experiment and a [training script](https://github.com/emanbuc/Optimizing_a_Pipeline_in_AzureML/blob/master/train.py) in the "Optimizing a Pipaline in AzureML" [GitHub repository](https://github.com/emanbuc/Optimizing_a_Pipeline_in_AzureML)

A reference training script example you can use as template is the following one:

```Python
# using a --regularization argument to set the regularization rate 
# hyperparameter, and logs the accuracy metric with the name Accuracy

import argparse
import joblib
from azureml.core import Run
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

# Get regularization hyperparameter
parser = argparse.ArgumentParser()
parser.add_argument('--regularization', type=float, dest='reg_rate', default=0.01)
args = parser.parse_args()
reg = args.reg_rate

# Get the experiment run context
run = Run.get_context()

# load the training dataset
data = run.input_datasets['training_data'].to_pandas_dataframe()

# Separate features and labels, and split for training/validatiom
X = data[['feature1','feature2','feature3','feature4']].values
y = data['label'].values
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30)

# Train a logistic regression model with the reg hyperparameter
model = LogisticRegression(C=1/reg, solver="liblinear").fit(X_train, y_train)

# calculate and log accuracy
y_hat = model.predict(X_test)
acc = np.average(y_hat == y_test)
run.log('Accuracy', np.float(acc))

# Save the trained model
os.makedirs('outputs', exist_ok=True)
joblib.dump(value=model, filename='outputs/model.pkl')

run.complete()

```

### Configuring hyperdrive experiment

To configure HyperDrive you must use HyperDriveConfig class as shown in the following example:

from azureml.core import Experiment
from azureml.train.hyperdrive import HyperDriveConfig, PrimaryMetricGoal

```Python

# Assumes ws, script_config and param_sampling are already defined
hyperdrive = HyperDriveConfig(run_config=script_config,
                              hyperparameter_sampling=param_sampling,
                              policy=None,
                              primary_metric_name='Accuracy',
                              primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                              max_total_runs=6,
                              max_concurrent_runs=4)

experiment = Experiment(workspace = ws, name = 'hyperdrive_training')
hyperdrive_run = experiment.submit(config=hyperdrive)
```

### Monitoring and reviewing hyperdrive runs

You can monitor hyperdrive experiments in Azure Machine Learning studio, or by using the Jupyter Notebooks RunDetails widget.

![monitoring experiment](/assets/img/hyperdrive_monitoring-experiment.png)