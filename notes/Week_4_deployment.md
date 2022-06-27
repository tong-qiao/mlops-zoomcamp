# Model deployment

There are mulltiple options for model deployment. If we can wait, we can go for batch deployment (off line which runs regularly), and online model deployment. In the online option, there are web service and streaming.

## Batch mode

* Run the model regularly
* Requires a database for storing data and another for the prediction results to go to
* Could be used for marketing related tasks, such as churn (measure of how many customers may stop using a product)

## Web service

* Such as the ride duration prediction example
* Needs to be up and running all the time
* 1-1 client-server

## Streaming

* There are producers and consumers. Producers push events and consumers will read events.
* 1-N client-servers

## Deploy models with Flask and Docker

* Dockerfile
```bash
FROM python:3.9.12-slim # slim version of python

RUN pip install -U pip
RUN pip install pipenv 

WORKDIR /app # create a folder for working directory

COPY [ "Pipfile", "Pipfile.lock", "./" ] # copy over pipfile and pip lock file

RUN pipenv install --system --deploy # no need to install another virtual environment inside docker so --system and --deploy are needed here

COPY [ "predict.py", "lin_reg.bin", "./" ] # copy over the predict file and model

EXPOSE 9696 # expose the port for using production server

ENTRYPOINT [ "gunicorn", "--bind=0.0.0.0:9696", "predict:app" ]
```

* Build the docker image
```bash
docker build -t ride-duration-prediction-service:v1 .
```

* Run the docker image
```bash
docker run -it --rm -p 9696:9696  ride-duration-prediction-service:v1
```

* Test if the server is running
```bash
python test.py
```

## Getting the models from the model registry (mlflow)

Use `relative/path/to/local/model` as model uri to load model. In this case, we need to set tracking uri and experiment name:
```python
import mlflow

MLFLOW_TRACKING_URI = 'http://127.0.0.1:5000'
mlflow.set_tracking_uri(MLFLOW_TRACKING_URI)
mlflow.set_experiment("green-taxi-duration")

RUN_ID = '5f62fe78c10d45d7a2dd88a8d5c61df6'

logged_model = f'runs:/{RUN_ID}/model'
model = mlflow.pyfunc.load_model(logged_model)
```

Use `runs:/<mlflow_run_id>/run-relative/path/to/model` as model uri to load model. No tracking uri and experiment name is needed:
```python
import mlflow

RUN_ID = '5f62fe78c10d45d7a2dd88a8d5c61df6'

logged_model = f'./mlruns/1/{RUN_ID}/artifacts/model'
model = mlflow.pyfunc.load_model(logged_model)
```

## Batch:Preparing a scoring script





## Tips and tricks

* Show the current version of `scikit-learn` and install a virtual environment via `pyenv`
```
pip freeze | grep scikit-learn
pipenv install scikit-learn==1.0.2 flask --python=3.9
```

* Activate the virtual environment. The pip and lock file will be found in the directory.
```bash
pipenv shell
```

* Install the production WSGI server
```bash
pipenv install gunicorn
```
or
```bash
pip install gunicorn
```

* Run flask app on the production server
```bash
gunicorn --bind=0.0.0.0:9696 predict:app
```

* Install library (requests for this example) only for dev purposes
```bash
pipenv install --dev requests
```

* Convert Jupyter notebooks to .py files
```bash
jupyter nbconvert --to script score.ipynb
```





