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

In docker-compose version of running the postgresql we have 2 options. All these options are available in the docker-compose folder, in the root directory of the project.

* Option no. 1: ***WithoutAutoVolume***
In this scenario, we are not creating the volume manualy in advance. Instead we have statement inside the docker-compose file 
    ```docker-compose
        volumes:
            pgdata:
                driver: local
            pgadmindata:
                driver: local
    ``` 
    Which, indicates that once we will launch the containers, the corresponding volumes will be created automatically, and hence there will not be a data loss.
    These volumes **pgdata** and **pgadmindata** are referenced in the docker-compose file on lines 14 and 28, i.e. it is expected that these volumes will be created before starting the containers. 
    One more difference with the previous configuration is the **pgadmin** service, which is the web interface helping to connect and browse the postgresql database from the web. The service is launched on the port 8000 and we have hardcoded username and passwords for pgadmin as well as for the postgresql database. 
    Pay attention to the line 19 and 20 in the docker-compose file. The service **pgadmin** will not start until the postgresql database is launched. The ***depend_on*** statement helps to do that.
    In line 15 we also have an example of providing the init.sql script to the database, as it was in previous case. 

* Option no. 2: ***WithSpecificVolume***
This case is not that different from the Option no 1, the only change you will see is in the *docker-compose.yaml* file in line 14 and 28. We specify the exact path to the file system folder, where the docker container volume should be mounted, on the host machine (where docker engine is running).
In this case no AutoCreated volumes are generated and we can easily copy or move the database related content.

## Minikube

In order to create the database deployment in the minikube, first we need to create the persistent volume, using the ./minikube/pv.yml file. Afterwards to make the persistent volume available for the deployment, we need to create the persistent volume claim. 

for creating the persistent volume and persistent volume claim, we can simply run the following commands:

```shell
kubectl apply -f minikube/pv.yml
kubectl apply -f minikube/pvc.yml
```

In order to see the created persistent volume, persistent volume claim and other storage related details, we can run the following command:

```shell
kubectl get pv,pvc,storageclass
```
You should be able to see the ***postgres-pv*** and ***postgres-pvc***.

The next step is to create the database secrets, i.e. username and password. For that we need to encode them using base64 encoding. IMPORTANT NOTE - this is not an encryption, NEVER RELY ON base64 encoding!

To encode the username and password you need to execute the follwing commands

***echo devopscourseuser | base64***
***echo devopscoursepassword | base64***

outputs should be saved in the ./minikube/postgres-secrets.yml file as a username and as a password.

## EKS

[MIT](https://choosealicense.com/licenses/mit/)