# Plugin HTH (Highway to Heel)

## Description

This plugin reads the *metric* values for explored *knobs* configurations and *knobs* and *features* definition of the application.
Based on these it tries to find the best model describing the given *metric*.

Model used in this plugin are
For this it uses several models:
- linear
- MARS
- polymars
- kriging

and their ensembles (bagging and stacking).
This plugin uses holdout set for the validation of models and computes the *R2* and *MAE* on the holdout set.
User may define what should be the minimum *R2*, maximum *MAE*, size of the holdout set and number of subset for cross-validation.
Cross-validation models are used for computation of the ensembles.

Output of the plugin is a table with the *metric* values predictions on the whole design space and table with evaluation of all the models predictive capabilities.

## Requirements

The main script that perform the computation is written in [R](https://cran.r-project.org/).
Communication with Cassandra is done using JDBC driver for Cassandra.
- Rscript (tested with version 3.43)

## Dependency

In the current implementation requires [R](https://cran.r-project.org/) and the following R packages
- [DiceKriging](https://cran.r-project.org/web/packages/DiceKriging)
- [tidyverse](https://cran.r-project.org/web/packages/tidyverse/)
- [rlang](https://cran.r-project.org/web/packages/rlang/)
- [mda](https://cran.r-project.org/web/packages/mda)
- [polspline](https://cran.r-project.org/web/packages/polspline/)
- [quadprog](https://cran.r-project.org/web/packages/quadprog/)
- the [RJDBC](https://cran.r-project.org/web/packages/RJDBC/) (only when CASSANDRA DB is used)
```R
install.packages('DiceKriging')
install.packages('tidyverse')
install.packages('rlang')
install.packages('mda')
install.packages('polspline')
install.packages('quadprog')
install.packages('RJDBC')
```

## Running the plugin without [mARGOt](https://gitlab.com/margot_project)

To run the plugin without the [mARGOt](https://gitlab.com/margot_project) you can run the main.R script with one parameter with the path to the plugin folder.
```
Rscript model.R "~/git/all/hth"
```

### Config file

In this folder is also the configuration file called *agora_config.env*.
This file should contain following settings
- STORAGE_TYPE, the storage type *CSV* or *CASSANDRA*.
- STORAGE_ADDRESS, relative or absolute path to the folder/keyspace with data
- APPLICATION_NAME, name of the application
- METRIC_NAME, metric name as is in the trace table
- ITERATION_COUNTER, an integer defining the iteration. This is used by [mARGOt](https://gitlab.com/margot_project) to keep track if there were several iterations of the DOE sampling.
- NUMBER_CONFIGURATIONS_PER_ITERATION, number of point which should be sampled from the design space, this number is used only in the models.log output file to identify how many input points were used in the [DOE](../doe/) plugin
- MAX_NUMBER_ITERATION, number of iteration in [mARGOt](https://gitlab.com/margot_project) (does not make sense unless You use iterative system on sampling design space and model learning)
- MAX_MAE, a number between 0 and 1. It sets the maximum *MAE* which is allowed for the model selection. *MAE* is normalized by the range of *metric*
- MIN_R2, a number between 0 and 1. It sets the minimum *R2* which is allowed for the model selection.
- VALIDATION_SPLIT, a number between 0 and 1. It sets how large should be the holdout set: number of observation \* VALIDATION_SPLIT
- K_VALUE, integer, how many folds should be used for cross-validation
- LIMITS, optional string of restrictions on the *knobs*, delimited by ";". example `"x + y < 1; x > -2"`

### Input data files

In the data folder given by $STORAGE_ADDRESS should be three CSV files given by $APPLICATION_NAME_knobs.csv, $APPLICATION_NAME_features.csv and $APPLICATION_NAME_trace.csv.
If the application name includes "/", these will be changed to "\_" due to the [mARGOt](https://gitlab.com/margot_project) configurations.
So if the application name is "kursawe/v1/test", resulting file names will be
- kursawe_v1_test_knobs.csv
- kursawe_v1_test_features.csv
- kursawe_v1_test_trace.csv.

$APPLICATION_NAME_knobs.csv should have two comma separated columns *name,values*, where *values* should contain all the possible values that are feasible for given *knobs* separated by ";".
In the example files, this file also has column *type* which is used by [mARGOt](https://gitlab.com/margot_project).

Example
```
name,type,values
x,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0;1.1;1.2;1.3;1.4;1.5;1.6;1.7;1.8;1.9
y,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0;1.1;1.2;1.3;1.4;1.5;1.6;1.7;1.8;1.9
z,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0;1.1;1.2;1.3;1.4;1.5;1.6;1.7;1.8;1.9
```

$APPLICATION_NAME_features.csv should have two comma separated columns *name,values*, where *values* should contain all the possible values that are feasible for given *features* separated by ";".
In the example files, this file also has column *type* which is used by [mARGOt](https://gitlab.com/margot_project).
The difference between *knobs* and *features* is that *knobs* may be set by the application, while *features* are constant.
Outside [mARGOt](https://gitlab.com/margot_project) it probably does not make much sense to use *features*.

Example for feature *k*
```
name,type,values
k,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0
```

$APPLICATION_NAME_trace.csv should contain column with *knobs*, *features* and *metrics*.
In the example files, this file also has columns *sec*, *nanosec* and *client_id* which are used by [mARGOt](https://gitlab.com/margot_project).

Example for knobs *x,y,z* and metrics *m,n*
```
sec,nanosec,client_id,x,y,z,m,n
1543768546,462641948,'14879',1.000000,1.000000,1.000000,-15.072766,15.622065
1543768546,512865089,'14879',1.600000,0.800000,1.900000,-13.613519,5.056408
1543768546,563166653,'14879',1.600000,0.800000,1.900000,-13.613519,5.05640
```

### Output files

The script creates three output files
- $APPLICATION_NAME_model.csv.
- models.log
- kriging.log

$APPLICATION_NAME_model.csv is a file which should be output of the learning module and in this case its output is only to create a dummy file with all the possible configurations and no results.

Example for *knobs* named *x,y,z* and metric *m*
```
"x","y","z","m_avg","m_std"
1.1,0,0,-17.8917153615383,-1
1.2,0,0,-17.7113031333671,-1
```

Files models.log and kriging.log are created in the plugin directory.
models.log contains report on the models predictive capabilities.
It consists of *model*, *R2*, *MAE*, *iteration* which is given by the $ITERATION_COUNTER, *run* which is a counter of how many times the plugin was run (it increases by 1, if the models.log already existed and the $ITERATION_COUNTER == 1), *nconf* given by $NUMBER_CONFIGURATIONS_PER_ITERATION and *metric* for which the plugin was run.
This file may be used to analyse how different model fare in multiple runs or with different settings.

```
model, R2, MAE, iteration, run, nconf, metric
bagging_kriging,0.9995540339352345,0.0034857728515770513,1,1,50,m
bagging_linear,0.9214566581343357,0.048445546081857895,1,1,50,m
bagging_linear2,0.9795891589393507,0.023869407415315964,1,1,50,m
bagging_mars,0.9948878084663,0.011473032022374275,1,1,50,m
bagging_polymars,0.9979668513115559,0.008160478735693723,1,1,50,m
kriging,0.9994666225041993,0.0033545162273879558,1,1,50,m
linear,0.9215788660551102,0.04839387284175288,1,1,50,m
linear2,0.9795382137036733,0.023821195084716404,1,1,50,m
mars,0.991958731227792,0.013926465525898492,1,1,50,m
polymars,0.9953495882958415,0.011025999959461423,1,1,50,m
stacking,0.999460155255085,0.00365435259580509,1,1,50,m
```

File kriging.log is just a log from the kriging fitting, so the stdout is less bloated.

# Acknowledgement

This work was supported by the ESF in “Science without borders” project, reg. nr. CZ.02.2.69/0.0/0.0/16_027/0008463 within the Operational Programme Research, Development and Education.
