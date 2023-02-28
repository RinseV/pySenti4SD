# Dockerized pySenti4SD

This repository encapsulates PySenti4SD as a docker image. The Dockerfile within 
this repository can be used to build PySenti4SD as a docker image. Moreover, the 
built image within this repository is also pushed to DockerHub as nathancassee/pysenti4sd. 

To run it, simply use `docker run nathancassee/pysenti4sd`. You can use volumes to 
get input and output files in the container. This image opens a bash shell in the 
folder in which Senti4SD is installed. To use Senti4Sd please see the original 
Senti4SD documentation below. 

To ease the usage of pysenti4sd we provide `utility.py`. This file can be 
run using the following command:

`python utility.py --train --model-name [name] --input [input-file] --output [output-file]`

The utility script has three modes:
- `--models`: List all pretrained models that are present (Currently only `Senti4SD`).
- `--train`: Given a CSV with text and polarity labels train Senti4SD. 
- `--predict`: Label a CSV file with predicted sentiment polarity according to an existing model.

# Senti4SD

Python implementation of Senti4SD. Senti4SD is an emotion polarity classifier specifically trained to support sentiment analysis in developers' communication channels. 
Senti4SD is trained and evaluated on a gold standard of over 4K posts extracted from Stack Overflow. It is part of the Collab Emotion Mining Toolkit, ([EMTk](https://github.com/collab-uniba/EMTk)).

## Fair Use Policy
Please, cite the following paper if you intend to use Senti4SD tool for your own research:
> Calefato, F., Lanubile, F., Maiorano, F., Novielli N. (2018) "Sentiment Polarity Detection for Software Development," _Empirical Software Engineering_, 23(3), pp:1352-1382, doi: https://doi.org/10.1007/s10664-017-9546-9. [(BibTeX)](https://scholar.googleusercontent.com/scholar.bib?q=info:2Vtb0Wmx7hEJ:scholar.google.com/&output=citation&scisig=AAGBfm0AAAAAW9gCvJzwrHV1MKhoxzqLaJZA8lPDFxgx&scisf=4&ct=citation&cd=-1&hl=en)


## Usage ##
In the following, we show first how to train a new model for polarity classification and, then, how to test the model on unseen data.  
For testing purposes, you can use the Sample.csv input file available in the root of the repo.
### Train a new classification model ###
```bash
sh train.sh -i train.csv [-d csv_delimiter] [-g] [-c chunk-size] [-j jobs-number] [-o model-name]
```
or you can run the script with two separated datasets, one for training and the other for testing:
```bash
sh train.sh -i train.csv -i test.csv [-d csv_delimiter] [-g] [-c chunk-size] [-j jobs-number] [-o model-name]
```

where
* ```-i dataset.csv```: is a file containing the data to train a classification model.  
  The dataset must contain at least the following two columns, in any order:
  ```text
  Text;Polarity  
  …  
  """@DrabJay: excellent suggestion! Code changed. :-)""";positive  
  """@IgnacioOcampo, I gave up after a while I am afraid :(""";negative    
  …
  ```
  same settings are valid if the test set is used separately.
* ```-d csv-delimiter```: the delimiter used in the csv file, where c stands for comma and sc for semicolon. [Default value: "c"]
* ```-F features```: all features to be considered. A stands for all, L stands for lexicon fetures, S stands for semantic features and K stands for keyword features. [Default value: A]
* ```-g```: enables the extraction of n-grams (i.e,. bigrams and unigrams). [optional]
* ```-c chunk-size```: the number of rows to read from the dataset per time, to avoid high memory usage. [Default value: 1000]
* ```-j jobs-number```: the number of cores to use during csv reading phase. If you pass -1 all cores will be used. 
If you pass a number higher than your total core number, the script will use all the cores. [Default value: 1] 
* ```-o model-name```: the name of trained model. [Default value: "Senti4SD"]

As a result, the script will generate the following output files:
* ```liblinear_perfomance/```: a subfolder containing the perfomance of all liblinear solvers on given test set
* ```UnigramsList and BigramsList files```: in the case the extraction of n-grams was enabled.
* ```Model-name.model```: trained classification model
* ```Model-name_info```: a file containing some info about the trained classification model

### Classification task ###
```bash
sh classification.sh -i dataset.csv [-d csv_delimiter] [-g] [-t] [-m model-name] [-c chunk-size] [-j jobs-number] [-o predictions.csv]
```

where
* ```-i dataset.csv```: is a file containing the documents to classify.  
  The dataset must contain at least the following column:
  ```text
  Text 
  …  
  """@DrabJay: excellent suggestion! Code changed. :-)"""  
  """@IgnacioOcampo, I gave up after a while I am afraid :(""" 
  …
  ```
  If the dataset contains a column named ID, this will be saved inside the predictions.csv file.
* ```-d csv-delimiter```: the delimiter used in the csv file, where c stands for comma and sc for semicolon. [Default value: "c"]
* ```-F features```: all features to be considered. A stands for all, L stands for lexicon fetures, S stands for semantic features and K stands for keyword features. [Default value: A]
* ```-g```: enables use of UnigramsList and BigramsList.
* ```-t```: enables documents saving along with the prediction labels inside "predictions.csv" file. [optional]
* ```-m model-name```: the name of classification model to use to classifiy documents. [Default value: "Senti4SD"] 
* ```-c chunk-size```: the number of rows to read from the dataset per time, to avoid high memory usage. [Default value: 1000]
* ```-j jobs-number```: the number of cores to use during csv reading phase. If you pass -1 all cores will be used. 
If you pass a number higher than your total core number, the script will use all the available cores. [Default value: 1] 
* ```-o prediction-file-name```: the name of the csv file where to save the model predictions. [Default value: "predictions.csv"]

As a result, the script will create a ```prediction-file-name.csv``` inside ```predictions``` folder containing:
```text
  Polarity 
  …  
  positive
  negative
  …
  ```
  or for example, in the case the input dataset contains a column named "ID" and the ```-t``` parameter is used, the ```predictions-file-name.csv``` will look like this: 
```text
  ID,Text,Polarity 
  …  
  21,"""@DrabJay: excellent suggestion! Code changed. :-)""",positive
  22,"""@IgnacioOcampo, I gave up after a while I am afraid :(""",negative
  …
  ```
For example, if you wanted to detect the polarity of the documents in the input file Sample.csv, you would have to run:

```bash
sh classification.sh -i Sample.csv -d sc
```
