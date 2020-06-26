# ABQ vehicle emissions reduction

## Project introduction

A project initiated and mentored by [Noa Tamir](https://github.com/noatamir) and [Katharina Rasch](https://github.com/krasch), hosted by [Women in Machine Learning and Data Scienc (WIMLDS)](https://github.com/wimlds/).

A detailed description of the "berlin-ds-lab", data and data cleaning initialisation can be found [here](https://github.com/wimlds/berlin-ds-lab).

Goal of the project was to create a data science project based on open data, simulating the work of a freelance data science team.

## Project brief

"Hi, this is Nisha, I work at the Albuquerque Environmental Health Department. We are on a mission to improve the air quality in our city. Our current mission: get polluting cars off the road and repaired before they are coming in for their regular emissions check. Can you help us?"

## Project ideation

We suggest to build a product that is able to predict emission results (passing and failing of car) based
* on the current results
* or based on the static car features that are provided (such as model, odometer, car age)

## Sources:

* [Original emission data of Albuquerque (open data)](https://opendata.cabq.gov/dataset/vehicle-emissions)
* [Data cleaning (Jupyter Notebook)](https://github.com/wimlds/berlin-ds-lab/blob/master/projects/cars/cars_cleanup_and_sample.ipynb)




## Installation


```bash
pip install foobar
```

## Usage

Baseline_model and data_prep are contributions by [Marielle Dado](https://github.com/marielledado).

1. Clone or download this repo

`git clone https://github.com/hhnnhh/berlin_datascience_lab.git`

2. Change working directory to root folder of the project

`cd ds-lab`

3. Make sure you have Anaconda/`conda` installed. Verify using the command `conda --version`.

4. Create a new virtual environment named `dslab`

`conda create --name dslab`

5. Activate virtual environment `dslab`

`conda activate dslab`

6. Install `requirements.txt`

```python
pip install -r requirements.txt
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)