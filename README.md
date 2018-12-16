# ALL - Agora Learning Library

This repository consists of two plugins for the autotuner [mARGOt](https://gitlab.com/margot_project).
These plugins are written in [R](https://cran.r-project.org/) and use CSV or [Apache Cassandra](http://cassandra.apache.org/) for data storage.
Plugins are made to work with mARGOt, it is possible to run the R scripts on their own, if the data are prepared according to their documentation.

First plugin [DOE](doe/) is focused on sampling the application design space and returns a set of configurations which should be run by the [mARGOt](https://gitlab.com/margot_project) to maximize the information given by exploring these configurations.

Second plugin [HTH](hth/) is a learning plugin which uses the results of the explored configurations and creates a model for the whole design space.
For this it uses several models:
- linear
- MARS
- polymars
- kriging
and their ensembles.
After the learning procedure it validates models on the holdout data.
User specifies minimum R2 and maximum MAE for the models.
If there are models which has acceptable R2 and MAE, the one with the minimum MAE is chosen as the best model.
Finally, the HTH plugin writes predicted values for a whole design space into CSV file or CASSANDRA.

Data to make the test runs are provided in the folder *data*.

# Acknowledgement

This work was supported by the ESF in “Science without borders” project, reg. nr. CZ.02.2.69/0.0/0.0/16_027/0008463 within the Operational Programme Research, Development and Education.
