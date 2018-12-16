*metric*# Plugin DOE (Design of Experiments)

## Description

This plugin reads the description of the application\`s design space.
User can define following parameters
- sampling algorithm
- distance parameter for the sampling algorithms
- number of points to be chosen
- how many times should [mARGOt](https://gitlab.com/margot_project) run each configuration
- restrictions on the design space.

At the moment it is possible to chose from the following sampling algorithms provided by the [DiceDesign](https://cran.r-project.org/web/packages/DiceDesign) package.
- full factorial designs
- Strauss Design
- Maximum Entropy Design (Dmax)
- Latin Hypercube Design
- WSP design.

Output of the algorithm is a table with *knobs* configurations and counter column.
For example if the *knobs* are named *x,y,z*, the output will be.
```
x,y,z,counter
0,0,0,1
0.5,0,0,1
```

## Requirements

The main script that perform the computation is written in y.
Communication with Cassandra is done using JDBC driver for Cassandra.
- Rscript (tested with version 3.43)

## Dependency

In the current implementation requires [R](https://cran.r-project.org/) and the following R packages
- [DiceDesign](https://cran.r-project.org/web/packages/DiceDesign/index.html)
- [tidyverse](https://cran.r-project.org/web/packages/tidyverse/)
- [rlang](https://cran.r-project.org/web/packages/rlang/)
- the [RJDBC](https://cran.r-project.org/web/packages/RJDBC/) (only when Cassandra is used)
```R
install.packages('DiceDesign')
install.packages('tidyverse')
install.packages('rlang')
install.packages('RJDBC')
```

## Running the plugin without [mARGOt](https://gitlab.com/margot_project)

To run the plugin without the [mARGOt](https://gitlab.com/margot_project) you can run the main.R script with one parameter with the path to the plugin folder.
```
Rscript main.R "~/git/all/doe"
```

### Config file

In this folder is also the configuration file called *agora_config.env*.
This file should contain following settings
- STORAGE_TYPE, the storage type *CSV* or *CASSANDRA*.
- STORAGE_ADDRESS, relative or absolute path to the folder/keyspace with data
- APPLICATION_NAME, name of the application
- DOE_NAME, name of the sampling algorithm (*dmax*, *lhs*, *strauss*, *wsp*, *full_factorial*)
- NUMBER_CONFIGURATIONS_PER_ITERATION, number of point which should be sampled from the design space
- NUMBER_OBSERVATIONS_PER_CONFIGURATION, number how many run should [mARGOt](https://gitlab.com/margot_project) do on each configuration
- MAX_NUMBER_ITERATION, number of iteration in [mARGOt](https://gitlab.com/margot_project) (does not make sense unless You use iterative system on sampling design space and model learning)
- MINIMUM_DISTANCE, a number between 0 and 1, to determine distance on points sampled in the [0,1] space.
- LIMITS, optional string of restrictions on the *knobs*, delimited by ";". example `"x + y < 1; x > -2"`

### Input data files

In the data folder given by $STORAGE_ADDRESS should be two CSV files given by $APPLICATION_NAME_knobs.csv and $APPLICATION_NAME_metrics.csv.
If the application name includes "/", these will be changed to "\_" due to the [mARGOt](https://gitlab.com/margot_project) configurations.
So if the application name is "kursawe/v1/test", resulting file names will be
- kursawe_v1_test_knobs.csv
- kursawe_v1_test_metrics.csv.

$APPLICATION_NAME_knobs.csv should have two comma separated columns *name,values*, where *values* should contain all the possible values that are feasible for given *knobs* separated by ";".
In the example files, this file also has column *type* which is used by [mARGOt](https://gitlab.com/margot_project).

Example
```
name,type,values
x,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0;1.1;1.2;1.3;1.4;1.5;1.6;1.7;1.8;1.9
y,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0;1.1;1.2;1.3;1.4;1.5;1.6;1.7;1.8;1.9
z,double,0;0.1;0.2;0.3;0.4;0.5;0.6;0.7;0.8;0.9;1.0;1.1;1.2;1.3;1.4;1.5;1.6;1.7;1.8;1.9
```

$APPLICATION_NAME_metrics.csv should have one column *name*, where are listed all the possible *metrics*.
In practice [mARGOt](https://gitlab.com/margot_project) uses this file to determine *metrics* and the plugin which should be used to learn given model *metric*.
Therefore, in the example file there are two more columns *type,prediction*.

Example
```
name,type,prediction
m,double,hth
n,double,hth
```

### Output files

The script creates two output files
- $APPLICATION_NAME_doe.csv
- $APPLICATION_NAME_model.csv.

$APPLICATION_NAME_doe.csv contains all the configuration which should be explored by [mARGOt](https://gitlab.com/margot_project) and number of runs for each exploration.

Example for *knobs* named *x,y,z*
```
x,y,z,counter
0.5,1.3,0.8,1
1.3,1,1.3,1
```

$APPLICATION_NAME_model.csv is a file which should be output of the learning module and in this case its output is only to create a dummy file with all the possible configurations and no results.

Example for *knobs* named *x,y,z* and metric *m* and *n*
```
"x","y","z","m_avg","m_std","n_avg","n_std"
2,0,1,NA,NA,NA,NA
3,0,1,NA,NA,NA,NA
4,0,1,NA,NA,NA,NA
```

# Acknowledgement

This work was supported by the ESF in “Science without borders” project, reg. nr. CZ.02.2.69/0.0/0.0/16_027/0008463 within the Operational Programme Research, Development and Education.
