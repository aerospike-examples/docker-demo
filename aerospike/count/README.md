# Orchestration & Networking demo

Please see the README.md at the root of this project.

# Amazon EC2 - WORK IN PROGRESS!

Some work has been done to make this demo work on EC2. You will need to setup a VPC and configure it correctly (blog post is in the works). Creating through the "VPC Wizard" is the simplest way to get this right. The swarm can be created on EC2 thus

    $ export AWS_ACCESS_KEY_ID=<my key>
    $ export AWS_SECRET_ACCESS_KEY=<my secret key>
    $ export AWS_SECURITY_GROUP=<my group name eg. alvin-dockercon>
    $ export AWS_SUBNET_ID=<my subnet eg. subnet-6c87e947>
    $ export AWS_VPC_ID=<my VPC name eg. alvin-dockercon>

If you are not using the default Region of `us-east-1` then you will also need to set your Region and Zone that the VPC and Subnet were created in.

    $ export AWS_DEFAULT_REGION=us-west-2
    $ export AWS_ZONE=c

    $ scripts/create-swarm.sh amazonec2

In order for this to work, you will need to create a VPC via the VPC Wizard (not by just creating the VPC). You will also need to ensure that the secutiry group opens up the following ports

Docker Engine / Swarm
- TCP / 2376 / 0.0.0.0
- TCP / 3376 / 0.0.0.0

Consul
- TCP / 8400 / 0.0.0.0
- TCP / 8500 / 0.0.0.0
- UDP / 8600 / 0.0.0.0

Serf
- TCP / 7946 / 0.0.0.0

Web App
- TCP / 80 / 0.0.0.0

VxLAN
- UDP / 46354 / 0.0.0.0
Once the following is resolved https://github.com/docker/libnetwork/issues/358#issuecomment-128160349 
- UDP / 4789 / 0.0.0.0

You will also need to modify `prod/haproxy.yml` to change the volumnes mounted to the container
     # boot2docker images use the following
     # - "/var/lib/boot2docker:/etc/docker"
     # ubuntu / ec2 images use
     - "/etc/docker:/etc/docker"
