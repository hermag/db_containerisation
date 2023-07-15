# Database deployment from docker to EKS through minikube

In this repository you will find out instructions how to deploy the postgresql with secrets and some-preconfiguration using
1. Docker file
2. DockerCompose file
3. Minikube as deployment with configmaps and secrets
4. EKS as deployment and statefulset with configmaps and secrets

## Docker File

In the folder docker, you will find dockerfile, which pulls the latest postgres image from the [dockerhub](https://hub.docker.com/_/postgres) repository.

Dockerfile looks like this, it sets certain environmental variables that will be used to create the database with the name **devopscourse**, user **devopscourseuser** and the database password **devopscoursepassword**. It also copies the entrypoint sql script, the **init.sql**, which creates the table *users* and defines two columns *name* and *email*.

```docker
FROM postgres:latest
ENV POSTGRES_DB devopscourse
ENV POSTGRES_USER devopscourseuser
ENV POSTGRES_PASSWORD devopscoursepassword
COPY init.sql /docker-entrypoint-initdb.d/
```

The next step is to build the image, for that you need to be located in the same folder where the Dockerfile resides

```shell
docker build -t mypostgresqldb .
```

or if you are in other directory than docker, you should specify the location of the Dockerfile with an option -f, however in this case you also need to change the line 5 in Dockerfile, since build process will look for the init.sql file in the current location and not in a docker folder. 

```shell
docker build -f docker/Dockerfile -t mypostgresqldb .
```

Once we have the docker image, we can use the docker command to run the container, also to make the storage persistent, we need to specify the volume, where the database files will be stored. This is necessary since docker containers are stateless and no data is stored outside of memory, when the container is running. 

```shell
docker run -d --name mypostgresqldbservice -p 54321:5432 mypostgresqldb -v <Absolute Path to the folder on the local file system>/:/var/lib/postgresql/data
```
## Docker Compose

```python
import foobar

# returns 'words'
foobar.pluralize('word')

# returns 'geese'
foobar.pluralize('goose')

# returns 'phenomenon'
foobar.singularize('phenomena')
```

## Minikube

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## EKS

[MIT](https://choosealicense.com/licenses/mit/)