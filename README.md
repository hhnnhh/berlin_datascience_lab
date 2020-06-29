# ABQ vehicle emissions reduction

### Content:
1. Project Introduction
1. Project Brief
1. Project Ideation
1. Exploratory Data Analysis
   1. Reverse Modeling of Test Result
1. Baseline Model
   1. improved Baseline Model
1. Extended Model




## Project introduction

A project initiated and mentored by [Noa Tamir](https://github.com/noatamir) and [Katharina Rasch](https://github.com/krasch), hosted by [Women in Machine Learning and Data Science (WIMLDS)](https://github.com/wimlds/).

A detailed description of the "berlin-ds-lab", as well as the original data and a notebook for data cleaning  can be found [here](https://github.com/wimlds/berlin-ds-lab). Data cleaning will result in the subsample "sample.csv" which was used in this project. 

Team members were 
Hannah Bohle (HB), [Marielle Dado](https://github.com/marielledado) (MD), [Caitlin Duncan](https://github.com/CaitDunc) (CD) and [Isabelle Nguyen](https://github.com/izzbizz/) (IN). Authorship of notebooks is marked with initials. 

**Goal of the project** was to create a data science project based on open data, simulating the work of a freelance data science team.

## Project brief

"Hi, this is Nisha, I work at the Albuquerque Environmental Health Department. We are on a mission to improve the air quality in our city. Our current mission: get polluting cars off the road and repaired before they are coming in for their regular emissions check. Can you help us?"

## Project ideation

_Predictive forecasting model:_

We suggest the client a product that is able to predict emission results (passing and failing of car)

[MODEL 1] (current)
1. based on the static car features that are provided (such as model, odometer, car age), so that car owners know if their car is going to fail before going to the testing station

[MODEL 2] (future, not implemented within project timeframe)
1. based on the current results, so that the car owner can be informed about the likelihood of failing in a future test after having their car checked. 

## Sources:

* [Original emission data of Albuquerque (open data)](https://opendata.cabq.gov/dataset/vehicle-emissions)
* [Data cleaning (Jupyter Notebook)](https://github.com/wimlds/berlin-ds-lab/blob/master/projects/cars/cars_cleanup_and_sample.ipynb)


## Installation
The basic setup such as requirements.txt, baseline_model.py for Logistic Regression and data_prep are contributions by [Marielle Dado](https://github.com/marielledado), extensions for Support Vector Machine, Random Forest Classifier, hyperparameter tuning by Hannah Bohle.

1. Clone or download this repo

`git clone https://github.com/hhnnhh/berlin_datascience_lab.git`

2. Change working directory to root folder of the project

`cd berlin_datascience_lab`

3. Make sure you have Anaconda/`conda` installed. Verify using the command `conda --version`.

4. Create a new virtual environment named `dslab`

`conda create --name dslab`

5. Activate virtual environment `dslab`

`conda activate dslab`

6. Install `requirements.txt`

```python
conda install -r requirements.txt
```

## Exploratory Data Analysis
The EDA was performed in Jupyter Notebooks and with notebooks in Google Colab. 
All notebooks can be found under "exploration/01_EDA/"

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googlecolab/colabtools/blob/master/notebooks/colab-github-demo.ipynb)

All notebooks are accessible via Github, however, Colab Notebooks will require a Google Colab registration. 

### Reverse modeling of test result

[Sanity Check Model](exploration/01_EDA/sanityCheck_dataQuality_reconstruct_overall-results.ipynb) (Jupyter Notebook) by Hannah Bohle

Attention: *This section does not describe the actual forecasting model but a data quality check.*

I performed a sanity check of the data by reverse modeling the results variable ("overall_results") with the testing variables, using a Logistic Regression. 

**Goal:** If the data is clean and of high quality, it should be possible to reconstruct the overall results (pass/fail of emission test) with the testing variables that are used for this decision.

**Result of the sanity check:** Of the data set, it seems like only about 5% of the variables are used for the decision process. Nevertheless, after finding the important variables, the test results can completely be reconstructed, which speaks to a high data quality (ROC SCORE = 1). 
It seems like most of the variables are neither needed for the decision process or used for the test results nor are they collected, as a large percentage of the variables contain mostly NaN. 

**Overview over the variables:** The data set contain 127 variables (= total of information provided = 100%). Content can be subdivided into different categories:

|  | Total of information provided 127 (= 100%):   | Actually used for result decision  |
|---|---|---|
| Information about car and testing procedure (model, odometer.., station)  | 50 (39%)  |  -- |
| Emissions (HC, CO, CO2, O2) | 45 (35%) | 5 (4%) |
|On-Board Diagnosis Tools (OBD) | 18 (14%) | 1 (1%) |
|Repair features  | 10 (8%) | --  |
| Visual Inspection (e.g. smoke)  | 5 (4%)  | 5 (4%)  |

Of the provided data, only a small proportion is actually used for the decision of the emission result (Pass/Fail). 

The variables that explain the results are the following: 
* emissions: 'E_HIGH_CO_RESULT', 'E_HIGH_HC_RESULT', 'E_IDLE_CO_RESULT', 'E_IDLE_HC_RESULT', 'E_RESULT_STRING'
* OBD: 'OBD_RESULT'
* visual inspection: 'V_SMOKE1', 'V_SMOKE2', 'V_GASCAP', 'V_CAT', 'V_RESULT', 

Repair features and OBD variables hardly contain data (mostly around 99% missing values). Only the OBD_results variable contains data (15% missing data) and seems important for the test result.

In addition to PASS and FAIL, the data set also contains information about ABORTED tests and also rare occurrences of the values "O" and "I" with unknown meaning. Information about aborted tests and other results were discarded in the current and future models. 

![Emission results](/figures/results_piechart.png)

All variables were thoroughly explored before using. 

Problems we determine from the EDA and point out to the client are:
1. STATIONS: Some stations are biased, i.e. have a higher percentage of passing and failing than the average. 
2. SOFTWARE: A software version is biased. 
3. ODOMETER: Some mechanics in certain checking stations enter strange values such as 888.888, 88.888. and 8.888 into the odometer variable, also odometer values of 0, exactly 100.000 and above 400.000 miles seem odd. These values are treated as outliers and are removed. 
4. REPEATED CHECKS: In the data set (with more than 370.000 cars entered), several thousand cars are checked more than once a day. As repairs within a day are rather unlikely, the data of these cars are considered as suspicious. 

Digging deeper into the scatterplot of 'E_IDLE_CO2' and 'E_HIGH_CO2', removing outlier, to be able to see their true distribution. 

## Baseline Model

The [baseline model](exploration/02_baseline_model/vanilla_baseline_4_static_car_features.ipynb) (colab) was based on a selection of five car features that we expected to explain test results without actually touching the information of the test itself.

After exploring the data we considered these five car features as crucial:

1. ODOMETER
1. MODEL YEAR - converted to age
1. VEHICLE TYPE
1. FUEL TYPE
1. GVW_TYPE

Even after cleaning the variables, and modeling only passes and fails (leaving aborts out), and after training four different vanilla models (naive bayes, logistic regression, support vector machines (SVM) and random forest), "out of the box" all models perform extremely bad, with a ROC Score of ~.5, which is equivalent to chance level. 

### extended baseline model

For the [second baseline model](exploration/02_baseline_model/second_baseline_model_cleaned_scaled.ipynb), the following improvements were implemented: 
* categorical features were scaled with One-Hot-Encoding
* continuous features scaled with MinMaxScaler
* coping with imbalanced target value (~12% FAIL): 
  * for LogReg: class-weight = balanced 
  * for SVM: downsampling of PASS values

Now the ROC SCORE for the LogReg increases to .66, for the SVM to .65.  













## Modeling

Model 1: 
"Modeling emission results based on static car features"

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)