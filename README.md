Apache PredictionIO Docker
==========================

## Development

### Build PredictionIO Image

```
docker build -t predictionio/pio pio
```

## Usage

### Run PredictionIO with Selectable docker-compose Files

You can choose storages for event/meta/model to select docker-compose.yml.

```
docker-compose -f docker-compose.yml -f ... up
```

Supported storages are as below:

| Type  | Storage                   |
|:-----:|:--------------------------|
| Event | Postgresql, Elasticsearch |
| Meta  | Postgresql, Elasticsearch |
| Model | Postgresql, LocalFS       |

If you run PredictionIO with Postgresql, run as below:

```
docker-compose -f docker-compose.yml \
  -f pgsql/docker-compose.base.yml \
  -f pgsql/docker-compose.meta.yml \
  -f pgsql/docker-compose.event.yml \
  -f pgsql/docker-compose.model.yml \
  up
```

To use localfs as model storage, change as below: 

```
docker-compose -f docker-compose.yml \
  -f pgsql/docker-compose.base.yml \
  -f pgsql/docker-compose.meta.yml \
  -f pgsql/docker-compose.event.yml \
  -f localfs/docker-compose.model.yml \
  up
```

## Tutorial

In this demo, we will show you how to build a recommendation template.

### Run PredictionIO environment

The following command starts PredictionIO with an event server.
PredictionIO docker image mounts ./templates directory to /templates.

```
$ docker-compose -f docker-compose.yml \
    -f pgsql/docker-compose.base.yml \
    -f pgsql/docker-compose.meta.yml \
    -f pgsql/docker-compose.event.yml \
    -f pgsql/docker-compose.model.yml \
    up
```

We provide `pio-docker` command as an utility for `pio` command.
`pio-docker` invokes `pio` command in PredictionIO container.

```
$ export PATH=`pwd`/bin:$PATH
$ pio-docker status
...
[INFO] [Management$] Your system is all ready to go.
```

### Download Recommendation Template

This demo uses [predictionio-template-recommender](https://github.com/apache/predictionio-template-recommender).

```
$ cd templates
$ git clone https://github.com/apache/predictionio-template-recommender.git MyRecommendation
$ cd MyRecommendation
```

### Register Application

You need to register this application to PredictionIO:

```
$ pio-docker app new MyApp1
[INFO] [App$] Initialized Event Store for this app ID: 1.
[INFO] [Pio$] Created a new app:
[INFO] [Pio$]       Name: MyApp1
[INFO] [Pio$]         ID: 1
[INFO] [Pio$] Access Key: i-zc4EleEM577EJhx3CzQhZZ0NnjBKKdSbp3MiR5JDb2zdTKKzH9nF6KLqjlMnvl
```

Since an access key is required in subsequent steps, set it to ACCESS_KEY.

```
$ ACCESS_KEY=i-zc4EleEM577EJhx3CzQhZZ0NnjBKKdSbp3MiR5JDb2zdTKKzH9nF6KLqjlMnvl
```

`engine.json` contains an application name, so replace `INVALID_APP_NAME` with `MyApp1`.

```
...
"datasource": {
  "params" : {
    "appName": "MyApp1"
  }
},
...
```

### Import Data

To import training data to Event server for PredictionIO, this template provides an import tool.
The tool depends on PredictionIO Python SDK and install as below:

```
$ pip install predictionio
```
and then import data:
```
$ curl https://raw.githubusercontent.com/apache/spark/master/data/mllib/sample_movielens_data.txt --create-dirs -o data/sample_movielens_data.txt
$ python data/import_eventserver.py --access_key $ACCESS_KEY
```

### Build Template

This is Scala based template.
So, you need to build this template by `pio` command.

```
$ pio-docker build --verbose
```

### Train and Create Model

To train a recommendation model, run `train` sub-command:

```
$ pio-docker train
```

### Deploy Model

If a recommendation model is created successfully, deploy it to Prediction server for PredictionIO.

```
$ pio-docker deploy

```
You can check predictions as below:
```
$ curl -H "Content-Type: application/json" \
-d '{ "user": "1", "num": 4 }' http://localhost:8000/queries.json
```
