# Demonstration & Example of Aerospike and Docker

This code shows building a simple voting application with Node.js and Aerospike.

It goes with the following talk:

https://www.youtube.com/watch?v=r3hfpiLlkVs

First, the demo shows how easy it is easy to build a container with a local cluster and develop your application.

Second, you want to scale out the application. Building an app with a separate Node layer and Aerospike layer, and using HA proxy, and the service broker, makes a proper architecturally correct set of components.

Then, scaling both the app layers and the aerospike layers are shown.

This has been tested on Docker 1.11 and 1.12 , running on MacOS 10.10 and 10.11 . 

Recent improvements in Swarm keep making the demo shorter and sweeter, and pull requests are gratefully accepted. Especially the ones that use the Mac's native virtualization system instead of VMware.

# Orchestration & Networking demo

## Cavets

* has been tested on os-x 10.10 and 10.11
* scripts will create machines based on the vmwarefusion driver. If you don't have that, then you will need to make some changes. Feel free to submit your changes with pull requests!
* because boot2docker.iso is used, the locations of files will change if you use Ubuntu or something else. 

## Prepare the environment

Create dev machine, where all containers will run:

    $ cd common
    $ scripts/create-dev.sh
    $ echo "$(docker-machine ip dev) dev.myapp.com" | sudo tee -a /etc/hosts

Create a Swarm within the dev machine:

    $ eval $(docker-machine env dev)
    $ scripts/create-swarm.sh
    $ echo "$(docker-machine ip swarm-0) prod.myapp.com" | sudo tee -a /etc/hosts

Start the Viz, a quick visualization tool

     $ cd common/viz
     $ source scripts/setup.sh
     $ scripts/up.sh swarm-0
     $ echo "$(docker-machine ip swarm-consul) viz.myapp.com" | sudo tee -a /etc/hosts

The voting app will be available at http://viz.myapp.com:3000 

## Run the app in dev mode

To start app in development:

    $ cd aerospike/count/dev/
    $ source scripts/setup.sh
    $ docker-compose build
    $ docker-compose up

The app will be available at http://dev.myapp.com:5000

## Run the app in production, and scale the app tier

    $ cd aerospike/count/prod/
    $ source scripts/setup.sh
    $ docker $(docker-machine config --swarm swarm-0) network create --driver overlay --internal prod
    $ docker-compose up -d
    $ docker $(docker-machine config swarm-0) network connect prod prod_haproxy_1
    $ docker $(docker-machine config swarm-0) network connect prod prod_discovery_1
    $ docker-compose scale web=5

The app will be available at http://prod.myapp.com

You can log onto the Aerospike and look at data with aql

    $ docker run -it --rm --net prod aerospike/aerospike-tools aql -h prod_aerospike_1

    aql> select * from test.votes
    aql> select * from test.summary

## In production mode, scale the DB tier

    $ docker-compose scale aerospike=3

You can look at the cluster topology with

    $ docker run -it --rm --net prod aerospike/aerospike-tools asadm -e i -h prod_aerospike_1

# Building the images

If you want to rebuild the images for any reason, you will need to build, push and update the compose files as necessary (since you will push to a new repo)

## Build the web app

    $ HUB_USER="my hub user"
    $ cd dev
    $ eval "$(docker-machine env dev)"
    $ docker build -t $HUB_USER/demo-webapp-as .
    $ docker push $HUB_USER/demo-webapp-as

## Build the aerospike images

    $ HUB_USER="my hub user"

    $ cd aerospike
    $ eval "$(docker-machine env dev)"
    $ docker build -t $HUB_USER/aerospike-server .
    $ docker push $HUB_USER/aerospike-server
