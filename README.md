## Increase your VM memory to 8 GB
- The default amount of memory available for Docker on macOS is often not enough to get Airflow up and running. If enough memory is not allocated, it might lead to the webserver continuously restarting. You should allocate at least 4GB memory for the Docker Engine (ideally 8GB).
- How to check available mamory for VM/MacOS?
```commandline
docker run --rm "debian:bullseye-slim" bash -c 'numfmt --to iec $(echo $(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE))))'
```
- Example output
```
Unable to find image 'debian:bullseye-slim' locally
bullseye-slim: Pulling from library/debian
14726c8f7834: Pull complete
Digest: sha256:61386e11b5256efa33823cbfafd668dd651dbce810b24a8fb7b2e32fa7f65a85
Status: Downloaded newer image for debian:bullseye-slim
7.7G
```
- In this example 7.7G available for docker, enough for Airflow.

## Upgrade docker-compose version
```commandline
sudo curl -L "https://github.com/docker/compose/releases/download/v2.17.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

docker-compose --version
```

## Fetching docker-compose.yaml
```commandline
mkdir airflow_docker
cd airflow_docker
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.6.3/docker-compose.yaml'
```
- Modify 
- change example: `AIRFLOW__CORE__LOAD_EXAMPLES: 'false'`
- Add following environment variables
```commandline
AIRFLOW__SCHEDULER__DAG_DIR_LIST_INTERVAL: 5
AIRFLOW__CORE__DEFAULT_TIMEZONE: 'Europe/Istanbul'
```
### This file contains several service definitions: 
- airflow-scheduler - The scheduler monitors all tasks and DAGs, then triggers the task instances once their dependencies are complete.
- airflow-webserver - The webserver is available at http://localhost:8080.
- airflow-worker - The worker that executes the tasks given by the scheduler.
- airflow-triggerer - The triggerer runs an event loop for deferrable tasks.
- airflow-init - The initialization service.
- postgres - The database.
- redis - The redis - broker that forwards messages from scheduler to worker.

## Important directories
Some directories in the container are mounted, which means that their contents are synchronized between your computer and the container.
- ./dags - you can put your DAG files here.
- ./logs - contains logs from task execution and scheduler.
- ./config - you can add custom log parser or add airflow_local_settings.py to configure cluster policy.
- ./plugins - you can put your custom plugins here.

## Initializing Environment
```commandline
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

## Initialize the database
You need to run database migrations and create the first user account. To do this, run.
```commandline
docker-compose up airflow-init
```
## Up docker-compose
- After init, you can up your platform
```commandline
docker-compose up -d
```

- List containers
```commandline
(base) [train@localhost airflow_docker]$ docker-compose ps
NAME                                 IMAGE                  COMMAND                  SERVICE             CREATED             STATUS                            PORTS
airflow_docker-airflow-scheduler-1   apache/airflow:2.6.3   "/usr/bin/dumb-init …"   airflow-scheduler   59 seconds ago      Up 5 seconds (health: starting)   8080/tcp
airflow_docker-airflow-triggerer-1   apache/airflow:2.6.3   "/usr/bin/dumb-init …"   airflow-triggerer   59 seconds ago      Up 5 seconds (health: starting)   8080/tcp
airflow_docker-airflow-webserver-1   apache/airflow:2.6.3   "/usr/bin/dumb-init …"   airflow-webserver   59 seconds ago      Up 5 seconds (health: starting)   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp
airflow_docker-airflow-worker-1      apache/airflow:2.6.3   "/usr/bin/dumb-init …"   airflow-worker      59 seconds ago      Up 5 seconds (health: starting)   8080/tcp
airflow_docker-postgres-1            postgres:13            "docker-entrypoint.s…"   postgres            5 minutes ago       Up 5 minutes (healthy)            5432/tcp
airflow_docker-redis-1               redis:latest           "docker-entrypoint.s…"   redis               5 minutes ago       Up 5 minutes (healthy)            6379/tcp
```
- It will take more than 1 minute to become healthy and web server available.
## Open web UI
- username: airflow 
- Password: airflow
