# Simple Delivery Pipeline

Intended to provide a quick start point for performing a standup of the basic building blocks for testing.

The docker-compose file will bring up a Gogs server for git server based version control, Jenkins OSS for CI, and Nexus OSS for asset management.


### Pre-Requisites

Requires that you have installed [Docker](http://docker.com) to your machine, this will handle the creation of the local docker engine and installing the supporting commands for running `docker` and `docker-compose`

### Usage

Once you have cloned this repo and navigated to the directory, this will start all the servers but the jenkins nodes will fail, see below if you want to use them as well.

    $ pwd
    /Users/you/workspaces/simple-delivery-pipeline
    $ ls
    README.md               docker-compose.yml
    $ docker-compose up -d
    Creating network "simpledeliverypipeline_default" with the default driver
    Creating simpledeliverypipeline_jenkins-master_1 ...
    Creating simpledeliverypipeline_gogs_1 ...
    Creating simpledeliverypipeline_nexus_1 ...
    Creating simpledeliverypipeline_gogs_1
    Creating simpledeliverypipeline_nexus_1
    Creating simpledeliverypipeline_jenkins-master_1 ... done
    Creating simpledeliverypipeline_jenkins-node-jnlp_1 ...
    Creating simpledeliverypipeline_nexus_1 ... done


This will download, start the docker containers, and provide you with 3 running images for testing with upon completion

 * [Gogs - 0.11.29](http://localhost:10080)
 * [Nexus - 3.6.0](http://localhost:8081)
 * [Jenkins - 2.85](http://localhost:8080)

The first time you run this command the servers will come up un-configured, but once the startup complete you can see what images are running with.

    $ docker-compose ps
                   Name                                 Command                State                          Ports                      
    -----------------------------------------------------------------------------------------------------------------------------------------
    simpledeliverypipeline_gogs_1                /app/gogs/docker/start.sh  ...   Up         0.0.0.0:10022->22/tcp, 0.0.0.0:10080->3000/tcp  
    simpledeliverypipeline_jenkins-master_1      /bin/tini -- /usr/local/bi ...   Up         0.0.0.0:50000->50000/tcp, 0.0.0.0:8080->8080/tcp
    simpledeliverypipeline_jenkins-node-jnlp_1   jenkins-slave                    Exit 255                                                   
    simpledeliverypipeline_nexus_1               bin/nexus run                    Up         0.0.0.0:8081->8081/tcp   


### Setup Jenkins

The initial secret and admin password is output in the logs during the docker startup or you can open a shell on the running container.

Checking the logs:

    $ docker-compose logs jenkins-master|grep -n4 secret
    73-jenkins-master_1     | Please use the following password to proceed to installation:
    74-jenkins-master_1     |
    75-jenkins-master_1     | bbb549e897c2488b8e4a96962e50e05d
    76-jenkins-master_1     |
    77:jenkins-master_1     | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
    78-jenkins-master_1     |
    79-jenkins-master_1     | *************************************************************
    80-jenkins-master_1     | *************************************************************
    81-jenkins-master_1     | *************************************************************

Or opening a session to the running container:

    $ docker ps -a
    CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                           PORTS                                              NAMES
    eeda237b6d97        jenkinsci/jnlp-slave:latest   "jenkins-slave"          About an hour ago   Exited (255) About an hour ago                                                      simpledeliverypipeline_jenkins-node-jnlp_1
    8dd89ea93a2b        jenkinsci/jenkins:2.85        "/bin/tini -- /usr..."   About an hour ago   Up About an hour                 0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   simpledeliverypipeline_jenkins-master_1
    4766af2690cf        sonatype/nexus3:3.6.0         "bin/nexus run"          About an hour ago   Up About an hour                 0.0.0.0:8081->8081/tcp                             simpledeliverypipeline_nexus_1
    b18a7da3f58d        gogs/gogs:0.11.29             "/app/gogs/docker/..."   About an hour ago   Up About an hour                 0.0.0.0:10022->22/tcp, 0.0.0.0:10080->3000/tcp     simpledeliverypipeline_gogs_1
    $ docker exec -ti 8dd /bin/bash
    jenkins@8dd89ea93a2b:/$ cat /var/jenkins_home/secrets/initialAdminPassword
    bbb549e897c2488b8e4a96962e50e05d
    jenkins@8dd89ea93a2b:/$ exit
    exit
    $

Now login as admin and you can add the three JNLP Nodes to play around with. You need to manually create them and get the secret to edit the docker-compose file `JENKINS_SECRET: ...` line with on jenkins-node-jnlp, jenkins-node-jnlp2, and jenkins-node-jnlp3.


Once your server is up you can then start the jnlp node or nodes so you have some additional agents besides master if you need to test any multiple agent scenarios.


### Setup Nexus

Nothing specific to do here using default admin / admin123 credentials you should be able to login immediately.

### Setup Gogs

On first startup you will be prompted for the database type, for simple testing using SQLite is fine there are also a few updates to make this work more seamlessly from the default forwarded ports when constructing clone urls (**DO NOT UPDATE HTTP PORT** that is used for running the server)

 * ROOT_URL = http://localhost:10080
 * SSH Port = 10022

Accept the rest of the defaults and continue. This will bring you to a login screen where you can register a new user, by default the first user created is an administrator so create whatever user you want here and you are ready to start using Gogs.

## Working with your images

Now once you create the containers you will need to setup your data inside them as will any environment. However be mindful that the `docker-compose up` command is intended for building the container and will overwrite your data. So once you have an envitonment setup you can
