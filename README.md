# General Reference
### A personal reference for commands, tools, and hacks
#### © Gerrit Egnew, 2020


## relayr cloud http requests
```
http GET https://cloud.rlr-eu-dev.relayr.io/analytics/config/models/configurations \
    "Authorization: Bearer $RLR_TOKEN"
```

```
echo '[{"message": "The device is too hot because of some reason", "name": \
"missing_measurements", "state": "set", "timestamp": \ 
"2017-06-13T15:19:59.673+02:00" }]' \ | http POST \ 
https://cloud.rlr-eu-dev.relayr.io/devices/47e99bc4-6a52-4312-af71-247ab3d8c61e/alerts?deviceid=47e99bc4-6a52-4312-af71-247ab3d8c61e \
'Authorization: bearer 2701a7b36c26445fa2db65f09039980f'
```

## relayr AWS accounts
account: 189709606833
role: developers
display name: rlr-eu-dev

account: 339265221855
display name: rlr-eu-stg

# datetime reference

Local to ISO 8601:
`datetime.datetime.now().isoformat()`

UTC to ISO 8601:
`datetime.datetime.utcnow().isoformat()`

Local to ISO 8601 without microsecond:
`datetime.datetime.now().replace(microsecond=0).isoformat()`

UTC to ISO 8601 with TimeZone information (Python 3):
`datetime.datetime.utcnow().replace(tzinfo=datetime.timezone.utc).isoformat()`

UTC to ISO 8601 with Local TimeZone information without microsecond (Python 3):
`datetime.datetime.utcnow().replace(tzinfo=datetime.timezone.utc).astimezone().replace(microsecond=0).isoformat()`

Local to ISO 8601 with TimeZone information (Python 3):

```
import datetime, time

# Calculate the offset taking into account daylight saving time
utc_offset_sec = time.altzone if time.localtime().tm_isdst else time.timezone
utc_offset = datetime.timedelta(seconds=-utc_offset_sec)
datetime.datetime.now().replace(tzinfo=datetime.timezone(offset=utc_offset)).isoformat()
```

## relayr run airflow locally

Start Consul
`docker-compose up -d` in the same directory as the `docker-compose.yaml`

Connect consul to a docker network
`docker network connect mynet consul`

Build the project with `make build` in the workflow manager repo

Start a local postgres image:
`docker run --rm -d --name postgres -it postgres:9.6`

Connect to the postgres container and create roles
`docker exec -it postgres psql -U postgres`

Connect it to `mynet`
`docker network connect mynet postgres`

```
postgres=# create role root;
postgres=# create role analytics_workflow_manager;
postgres=# alter role analytics_workflow_manager with login;
postgres=# create database analytics_workflow_manager;
```

Run docker for the workflow manager
`docker run -it --network mynet --rm -p 2020:8080 -e CONSUL_HOST=http://consul:8500 -e CONSUL_HTTP_TOKEN=placeholder -e ENVIRONMENT=dev analytics-workflow-manager`

Access Airflow dashboard on http://localhost:2020/admin/

## more Docker stuff
To install a local python package with `pip` to a docker container, it must be
inside the context of the container, i.e.:
```
# Dockerfile
ADD ./analytics-utils
./analytics-utils
RUN pip install ./analytics-utils
```

## Kafkacat
kafkacat -C -b kafka-2.rlr-eu-dev:9092 -t device-events-analytics -f '%k\n\n%s\n\n=====\n' -X sasl.username=relayr -X sasl.password=e2bXqAJnXkehtEQRPCq7Yza5JYNfqHWatbSV4fFVw98Xo -X sasl.mechanisms=PLAIN -X security.protocol=sasl_plaintext  -X "api.version.request=true" -o -10000 -e

## Hive at relayr (deprecated)
To run tests locally, you need to tunnel from the HDFS cluster to local on both 10000 and 14000.
`ssh gerrit.egnew@emr-hdfs.rlr-eu-dev -N -L 10000:0.0.0.0:10000`

## Tools I've used
- CrateDB
- Spark
- Kafka
- Docker


