# ABQ vehicle emissions reduction

### Content:
1. Project Introduction
1. Project Brief
1. Project Ideation
1. Exploratory Data Analysis
   1. Reverse Modeling of Test Result
   1. biased or erroneous data entries
1. Predictive Model
   1. Baseline Model
   1. improved Baseline Model
   1. Final Model
1. Product Vision


## Project introduction

A project initiated and mentored by [Noa Tamir](https://github.com/noatamir) and [Katharina Rasch](https://github.com/krasch), hosted by [Women in Machine Learning and Data Science (WIMLDS)](https://github.com/wimlds/).

A detailed description of the "berlin-ds-lab", as well as the original data and a notebook for data cleaning  can be found [here](https://github.com/wimlds/berlin-ds-lab). Data cleaning will result in the subsample "sample.csv", which was used in this project. 

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

2. based on the current results, so that the car owner can be informed about the likelihood of failing in a future test *after having their car checked*, by using the cars testing variables. 

## Sources:

* [Original emission data of Albuquerque (open data)](https://opendata.cabq.gov/dataset/vehicle-emissions)
* [Data cleaning (Jupyter Notebook)](https://github.com/wimlds/berlin-ds-lab/blob/master/projects/cars/cars_cleanup_and_sample.ipynb)


## Installation
The basic setup such as requirements.txt, [data preparation]((emissionscheck_alb/data_prep.py)) and [basic model setup](emissionscheck_alb/baseline_model.py) setup with preprocessing pipeline for Logistic Regression and SVC are contributions by [Marielle Dado](https://github.com/marielledado), extensions in model setup for Random Forest Classifier and hyperparameter tuning options by Hannah Bohle.

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

**Result of the sanity check:** Of the data set, it seems like only about 5% of the variables (or less) are used for the decision process. Nevertheless, after finding the important variables, the test results can completely be reconstructed, which speaks to a high data quality (ROC AUC = 1). 
It seems like most of the variables are neither needed for the decision process or used for the test results nor are they collected, as a large percentage of the variables doesn't contain any information (< 95% NaN). 

**Overview over the variables:** The data set contains 127 variables (= total of information provided = 100%). Content can be subdivided into different categories:

|  | Total of information provided 127 (= 100%):   | Actually used for result decision  |
|---:|---|---|
| Information about car and testing procedure (model, odometer.., station)  | 50 (39%)  |  -- |
| Emissions (HC, CO, CO2, O2) | 45 (35%) | 5 (4%)  |
|On-Board Diagnosis Tools (OBD) | 18 (14%) | 1 (1%) |
|Repair features  | 10 (8%) | --  |
| Visual Inspection (e.g. smoke)  | 5 (4%)  | 5 (4%)  |
|---|---|---|
| **TOTAL**                            | **= 100%**   | **= 9%** |

Of the provided data, only a small proportion of 9% is actually used for the decision of the emission result (Pass/Fail), and not all of them might even be necessary.  

The variables that were used in the reverse model to explain the test results variable are the following: 

* emissions: 'E_HIGH_CO_RESULT', 'E_HIGH_HC_RESULT', 'E_IDLE_CO_RESULT', 'E_IDLE_HC_RESULT', 'E_RESULT_STRING'
* OBD: 'OBD_RESULT'
* visual inspection: 'V_SMOKE1', 'V_SMOKE2', 'V_GASCAP', 'V_CAT', 'V_RESULT', 


Repair features and OBD variables hardly contain data (mostly around 99% missing values). Only the OBD_results variable contains data (15% missing data) and seems important for the test result.

In addition to PASS and FAIL, the results variable also contains information about ABORTED tests and also rare occurrences of the values "O" and "I" with unknown meaning. In the current and in future models only Pass/Fail results were used, information regarding all other results was dropped. 

![Emission results](/figures/overall_results.png)

All variables were thoroughly explored before using. 

#### erroneous, flawed, biased data

Problems we determine from the EDA and point out to the client are:
1. STATIONS: Some stations are biased, i.e. have a higher percentage of passing and failing than the average, e.g. station 422 and 430 have a higher ratio of fails compared to other stations (see Fig. "station"). 
2. SOFTWARE: Also software version 602 seems to have a bias towards failing (see Fig. "software"). 
3. ODOMETER: Some mechanics in certain checking stations seem to enter strange values such as 888.888, 88.888. and 8.888 into the odometer variable, also odometer values of 0, exactly 100.000 and above 400.000 miles seem odd. Cars with these odometer values are treated as outliers and are removed. 
4. REPEATED CHECKS: In the data set (with more than 370.000 cars entered), several thousand cars are checked more than once a day. As repairs within a day are rather unlikely, the data of these cars are considered as suspicious. (Info by [CD](https://github.com/CaitDunc))

![Biased stations](/figures/station_num.png)


![Biased software](/figures/software.png)

[Biased stations and software](exploration/01_EDA/EDA_important_car_features_biased_stations_software_HB.ipynb), as well as engine size, car model, cylinder, transmission type and dual exhaust were explored to gain information about their potential value for the predictive model. 

With [scatterplots](exploration/01_EDA/EDA_emissions_scatterplot_HB.ipynb) I've been digging deeper into 'E_IDLE_CO2' and 'E_HIGH_CO2'. It was necessary to remove some extreme outliers, to see their actual distribution and correlation. 

[Correlations](exploration/01_EDA/EDA_correlations_HB.ipynb) were used to explore the connection between various car features and the result variable. For the different types of variables different correlations were used (Pearson, pointbiserial, Cramers V), as described [here](https://medium.com/@outside2SDs/an-overview-of-correlation-measures-between-categorical-and-continuous-variables-4c7f85610365).

## Predictive Model 

### Baseline Model

Goal of the baseline model was to predict emission results (passing and failing of car)
based on *a small selection of static car features* that are provided, to establish a baseline.

The [baseline model](exploration/02_baseline_model/vanilla_baseline_4_static_car_features.ipynb) (colab) was based on a selection of five car features that we expected to explain test results without actually touching the information of the test (e.g. emissions, visual inspection..) itself.

After exploring the data we considered the following five car features a good starting point:

1. ODOMETER
1. MODEL YEAR - converted to age
1. VEHICLE TYPE
1. FUEL TYPE
1. GVW_TYPE

However, even after cleaning the variables (removing outliers and odd values), and by modeling only passes and fails (discarding aborts) and after training four different vanilla models (naive bayes, logistic regression, support vector machines (SVM) and random forest), "out of the box" all models perform extremely bad, with a ROC score of ~.5, which is equivalent to chance level. 

### improved Baseline Model

For the [second baseline model](exploration/02_baseline_model/second_baseline_model_cleaned_scaled.ipynb), the following improvements were implemented: 

* Feature Engineering: 
  * cars with odd ODOMETER values (0,888.888,88.888,8.888,100.000) were removed
  * FUEL_TYPE reduced from 6 to 3 categories (= Gasoline, Diesel, Other) (= "FUEL_REDUCED)
* coping with imbalanced target value (~12% FAIL): 
  * for LogReg: "class-weight = balanced"
  * for SVM: downsampling the frequency of PASS result values
* continuous features: ODOMETER, CAR AGE 
* categorical features: FUEL_REDUCED,VEHICLE_TYPE, GVW_TYPE

A) 
   1. *unscaled* continuous features (ODOMETER, AGE), but
   1. *one-hot-encoded* categorical features
   
=> ROC score = .5 

B)
   1. *scaled* continuous features (ODOMETER, AGE with MinMaxScaler)
   1. *one-hot-encoded* categorical features
   
=> ROC score of the LogReg = .66, of the SVM = .65.

   1. *scaled* continuous features (ODOMETER, AGE with MinMaxScaler)
   1. no categorical features used
   
=> ROC score of the LogReg = .66

**Summary:** Feature scaling of continuous features (Odometer, car age) improves ROC score. 
Odometer and car age seem crucial. Model type doesn't make a difference. We decide to stick to LogReg for now. 

### Final Model

Model 1: "Modeling emission results based on static car features"

To "beat the baseline", we decide to use feature selection and not hyperparameter tuning, for now. 

For the final model, we implemented a [data preparation toolbox](emissionscheck_alb/data_prep.py) and a [preprocessing pipeline](emissionscheck_alb/baseline_model.py) for Logistic Regression (by Marielle Dado) which was subsequently extended by Random Forest and hyperparameter tuning options (by Hannah Bohle).
 
In particular, data was preprocessed with the following features: 
 
* Data Preparation: 
  * select only Pass/Fail results
  * calculate car age from model year
  * encode P = 0, F = 1
  * continuous features: **MinMaxScaler**
  * categorical features: **One-Hot-Encoding**
  * cars with odd ODOMETER values (0,888.888,88.888,8.888,100.000 *and above 400.000*) were removed later
* coping with imbalanced target value (~12% FAIL): 
  * for LogReg: "class-weight = balanced"
* preprocessing performed with pipeline

With the following features: 
* [categorical:] "VEHICLE_TYPE", "FUEL_TYPE", "GVW_TYPE", 
* [continuous:] "ODOMETER", "CAR_AGE"

a LogReg again receives a better result than the baseline model, but about the same ROC SCORE as a LogReg with "ODOMETER", "CAR_AGE" only.

Preliminary result: ROC = .71

A Random Forest Classifier resulted in a similar ROC score. 

### Feature selection 

To improve the [Model 1](exploration/03_extended_model/improved_baseline_model_extended_HB.ipynb)  feature selection was used (by Hannah Bohle).

Baseline model features were: 
ODOMETER / MODEL AGE / VEHICLE TYPE / FUEL TYPE / GVW_TYPE (ROC score = .70)

* substituting "vehicle type" by "model" --> ROC: .73
* adding "engine_size" --> ROC: .74

*Summary:* A Logistic Regression with "ODOMETER", "CAR_AGE", "MODEL" "ENGINE-SIZE" receives the best ROC Score with .74 

## Product vision

We suggest the client to implement a web application were car owners can enter "ODOMETER", "CAR_AGE", "MODEL" and probably "ENGINE-SIZE" to get an impression if their car is likely to pass or fail their emissions test. 

The application will be easy to implement and maintain and might make emission testing more efficient. 

On the downside, the confidence of the predictions is still not high, even if 67% of the cars will in fact be classified correctly. That is, in the current model 30% of the cars will be announced as "likely to pass", while they will actually be failing ("false positive"), and 3% of the cars will be announced as "likely to fail", while they  will be passing the test ("false negative"). 

**Open tasks:** try beating the current score by either hyperparameter tuning and/or by using a more powerful model. 

The planned [MODEL 2] could unfortunately not be implemented within the time frame of the current project. Preliminary investigations such as the "quality check model" as well as some attempts during [feature selection](exploration/03_extended_model/improved_baseline_model_extended_HB.ipynb) give rise to the expectation that future predictions with the actual testing variables will be easier to implement and more powerful, because they are closely related to the results variable. However, the assumption still has to be checked. 



## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)