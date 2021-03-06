# This Repository contains the datasets as well as the scripts which were used in the Paper:
COVER: Conformational Oversampling as Data Augmentation for Molecules.
-----------------------------------------------------------------------------------------------------------

The folder "data" contains the sdf files with the 3D conformations which were used for the different training runs.

The folder "descriptors" contains a csv files with the descriptor names and an xml file for DRAGON 7. Due to the proprietary nature
of the descriptors the files containing the used datasets cannot be made publicly available. In addition it contains a KNIME (3.6)
workflow which generates 3D conformations using RDKits ETKDG algorithm (https://doi.org/10.1021/acs.jcim.5b00654).

The folder "clustering" contains a script to Cluster the data and optionally assign the clusters randomly to five folds.

The folder "training" contains two scripts: cross_validate.py which runs a nested 5x4 cross-validation
and train_models.py which is required by cross_validate.py but can also be used on its own to train one single model.
The rationale for using two distinct scripts was the problem that Keras does not allow any real reset of the trained model,
thus to prevent data leakage between models it was chosen to use a separate script to avoid any possible issues.
(see also https://github.com/keras-team/keras/issues/609, https://github.com/keras-team/keras/pull/1908)

## Support:
--------
Jennifer Hemmerich, jennifer.hemmerich[at]univie.ac.at

## Important Note:
--------

The procedure of model training requires your own pre-generated file with 3D based descriptors. To generate such a file
you need to generate 3D conformations and subsequently calculate 3D descriptors. For conformer generation we used the KNIME workflow available in the descriptors folder.
Unfortunately the descriptors used are not open source, therefore, after using the workflow you need to generate your own csv file with descriptors.
With this file you can follow the steps to generate your own models.

# Dependencies
--------------
All Scripts:
python 3.6.6
pandas 0.23.4  
numpy 1.14.3 

Clustering:  
rdkit 2018.09.1  


Training: 
keras 2.2.4  
scikit-learn 0.20.1  
tensorflow-gpu 1.12.0

To create a conda environment to use with the scripts you can either use the included environment.yml file which contains all packages present in the environment
which was used for this study. 
The environment_minimal.yml only has the minimal requirements for the scripts to run.  

```
conda env create -f environment.yml
```


Please note: The *.yaml files were created using Ubuntu 16.04 LTS. Therefore the *.yml files most likely will not work for WIN or MAC systems.
To create an environment install Anaconda and create an environment with python 3.6.6, then install the necessary packages mentioned above.


# Clustering
--------------
You need to first run the Cluster.py script to cluster the data and assign the clusters randomly to folds. These folds are later used as folds for the cross-validation.
The assign_folds.py script adds the columns "Clusters" and "Folds" to your csv file.
Alternatively you can generate your own clusters and folds.
NOTE: to use the cross-validation script your input csv file MUST contain a column named "Clusters" a column named "Reference" and a column named "Folds"

```
usage: Cluster.py [-h] [--out-base-dir OUT_BASE_DIR] --input-file INPUT_FILE
                  [--output-filename OUTPUT_FILENAME] [--assign-folds]

optional arguments:
  -h, --help            show this help message and exit
  --out-base-dir OUT_BASE_DIR
                        Specify the directory for the output, defaults to the
                        directory where the script is executed (os.getcwd())
  --input-file INPUT_FILE
                        Specify the sdf file for which the clusters should be
                        computed
  --output-filename OUTPUT_FILENAME
                        Specify how the output file containing the clusters
                        should be named, default is Clusters.csv
  --assign-folds        Use this flag if after the clustering clusters should
                        be randomly assigned to one of 5 folds
````

# Running the Double Cross-validation:
------------------------------------
The double cross-validation perfoms an inner cross-validation to get the best hyperparameter set and then retrains the model.
In the outer loop the retrained model is then evaluated on unseen data. 
In the outer loop the loop number indicates the fold which is used for testing (i.e. in Loop 1 Fold 1 is used for testing).
The inner loop loop numbers indicate the file which is used to validate the inner model. For early stopping and to find the best model
the balanced accuracy is used. 

Please note that the input file needs to contain three columns (apart from the descriptor columns). 
Namely :

(i) "Reference" which is used as an index for the molecules, for multiple conformations this should be the same name to calculate the metrics with respect to a single input
(ii) "Fold" which is generated by the Cluster.py script and used for splitting the dataset into the training and test fold. 

If you want to supply it manually it needs to contain numbers between 1 and 5. 

The whole script will generate a file structure in the folder where the script is run:
```
Run_YYYY_MM_DD-HH_MM_SS
├── code
├── data
│   ├── Grid.csv
│   ├── config.json
├── final_models
│   ├── Loop_1
│   ├── Loop_2
│   ├── Loop_3
│   ├── Loop_4
│   └── Loop_5
└── trained_models
    ├── Loop_1
    ├── Loop_2
    ├── Loop_3
    └── Loop_4

```
Code:   
Contains the code file which was run to build the model 
 
Data:   
Contains a csv file with all the different grid search conditions and the config *.json file supplied by the user  

Models folders:   
Contains data from the 4 trained models (inner loop)  and the final 5 models for the outer loop of the 5x4 crossvalidation
The final models are saved, whereas for the inner loop only the settings and the log is saved. Models are saved as *.json files 
and the weights in the respective *.h5 file. The csv files performance and performance_traceback store the model performances. 
In the inner loop each file contains the performance per loop, i.e. to get the cross-validation estimate the 4 values have to be used
to calculate the mean. The file performance_traceback indicates the performance when the metrics are not calculated using each conformation separately,
but using the mean of all conformations of a molecule as the final prediction. 


## Using the Script
------------------------------

Files which need to be supplied:
*.json: The configuration file for the model training, please use the included file as a template

cross_validate.py :

```
usage: cross_validate.py [-h] [--out-base-dir OUT_BASE_DIR]
                         [--input-dir INPUT_DIR] [--config-file CONFIG_FILE]
                         [--gpus GPUS]

optional arguments:
  -h, --help            show this help message and exit
  --out-base-dir OUT_BASE_DIR
                        Specify base directory for output directory structure,
                        defaults to the directory where the script is executed
                        (os.getcwd())
  --input-dir INPUT_DIR
                        Specify directory where source files are located,
                        defaults to the directory where the script is executed
                        (os.getcwd())
  --config-file CONFIG_FILE, -cf CONFIG_FILE
                        Specify config JSON to train models with
  --gpus GPUS           Which gpus to use for training, only the specfied gpus
                        will be visible for the training, at least 2 gpus need
                        to be specified!

```
runs the crossvalidation script, the following parameters are available

| Parameter | Description | default value |
|-----------|-------------|---------------|
|--out-base-dir | Specifies the base directory for output directory structure | defaults to the directory where the script is executed (os.getcwd())|
|--input-dir | Specifies the directory where source files are located | defaults to the directory where the script is executed (os.getcwd()) |
|--config-file| Specifies the config JSON to train models with | defaults to the config.json file found in the current working directory |
|--gpus| Defines which gpus to use for training, only the specfied gpus will be visible for the training, important: needs to be set to at least two gpus, values should be entered comma separated, e.g 0,1 | defaults to GPU 0 and 1 when ordering by BUS ID |

## Training single models
----------------------------

This script is called by the cross-validation script with all necessary parameters,
however it can also be used to train a single neural network, the following parameters have to be set:

Script train_model.py
```
usage: train_model.py [-h] [--test-fold TEST_FOLD] [--drop-fold DROP_FOLD]
                      [--save-directory SAVE_DIRECTORY]
                      [--config-file CONFIG_FILE]
                      [--hidden-units HIDDEN_UNITS]
                      [--learning-rate LEARNING_RATE]
                      [--dropout-input DROPOUT_INPUT]
                      [--dropout-hidden DROPOUT_HIDDEN] [--layers LAYERS]
                      [--gpus GPUS] [--save-model]

optional arguments:
  -h, --help            show this help message and exit
  --test-fold TEST_FOLD
                        Specifies the fold which should be used to validate
                        the models, defaults to 1
  --drop-fold DROP_FOLD
                        Specifies a fold which should be dropped, e.g. if it
                        should be used later for external testing, defaults to
                        0 (i.e. no fold is dropped)
  --save-directory SAVE_DIRECTORY
                        Specifies the directory to save files in that
                        directory it will create a foler Run_* where * is
                        dependent on the hyperparameters, defaults to the
                        directory where the script is executed (os.getcwd())
  --config-file CONFIG_FILE
                        Specifies config JSON to train models with
  --hidden-units HIDDEN_UNITS
                        Specifies the number of hidden units, default is 2
  --learning-rate LEARNING_RATE
                        Specifies the learning rate, default is 0.1
  --dropout-input DROPOUT_INPUT
                        Specifies the dropout rate for the input layer,
                        default is 0.2
  --dropout-hidden DROPOUT_HIDDEN
                        Specifies the row of dropout rate for hidden layers,
                        default is 0.5
  --layers LAYERS       Specifies the number of hidden layers, default is 2
  --gpus GPUS           Which gpus to use for training, only the specfied gpus
                        will be visible for the training, at least 2 gpus need
                        to be specified!
  --save-model          Use this flag if the best model should be saved after
                        training
```

| Parameter | Description | default value |
|-----------|-------------|---------------|
|--test-fold | Specifies the fold which should be used to validate the models | defaults to 1 |
|--test-fold | If needed: specifies a fold which should be dropped, e.g. because it should not be used at that stage, if 0: no fold will be dropped | defaults to 0 |
|--save-directory | Specifies the directory to save files in that directory it will create a foler Run_* where * is dependent on the hyperparameters | defaults to the directory where the script is executed (os.getcwd())|
|--config-file | Specifies the config JSON to train models with | defaults to the config.json file found in the current working directory |
|--hidden-units | Specifies the number of hidden units | defaults to 2 |
|--learning-rate | Specifies the learning rate | defaults to 0.1 |
|--dropout-input | Specifies the dropout rate for the input layer | defaults to 0.2 |
|--dropout-hidden | Specify the row of dropout rate for hidden layers | defaults to 0.5 |
|--layers | Specify the number of hidden layers | defaults to 2 |
|--gpus | Defines which gpus to use for training, only the specfied gpus will be visible for the training, important: needs to be set to at least two gpus, values should be entered comma separated, e.g 0,1 | defaults to GPU 0 and 1 when ordering by BUS ID |
|--save-model | Use this flag if the best model should be saved after training |
