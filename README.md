# Kapture

Kapture is an open source smoke testing and load generation tool based on modern application architectures, which typically combine N microservices against a durable message queue.

It is meant to be an 'infrastructure centric' implementation of the original bigpetstore paper, which largely
focuses on generating large amounts of streaming data w/ the apache bigtop project.

However, its current load generator is very primitive, and is still being fleshed out.

# Kapture data flow

This is only partially implemented as of April 8, 2019. The Postgres Mongo spout as well as the 
fine grained topics are remaining.  We'll update this shortly !

```
  Kafka <--- Generated BPS (or other) transactions <--------------- Webapp ----+
  / \ |                                                                        |
   |   |                                                   + ------> Redis <---+
   |   |                                                   |
   |   +-----> topic (massachusets)---> |                  |
   |   +-----> topic (california)  ---> | --> ETL [Spark, or just a script]
   |   +-----> topic (texas)       ---> |                  |
   |                                                       |
   |                                                       |
   -------------------Postgres/Mongo <---------------------+
```

# Usage

## Your first time ?
 
To run kapture, just download this repo, cd to it, and run:
```
./kapture.sh kapture-spam-my-namespace
# wait a while for your cluster to come up...
sleep 120
# Now, generate load !
kubectl create -f load-gen.yml -n kapture-spam-my-namespace
```

This will create a single load store generation that will write to various kafka topics, which then get fed 
into a redis in-memory data store.  It will trigger a wide variety of JVM, disk, Memory, and I/O patterns
proportional to the number of load generators, i.e., the number of petstore transactions which are generated.

For further configuration, try running `./kapture.sh --help` to see other configuration options specifics to the BigPetStore implementation.

## What if I want to test a more advanced scenario ?

Kapture is not overengineered to support a myriad of YAML situations.  We expect engineers and developers to 
use this framework and modify the YAMLs as required.  Out of the box, it does one thing, very well: Generating
realistic load for an enterprise grade application with a message queue and scalable key value store.

## So, what if you want to do something more advanced ?

Hack it ! The YAML recipes are all in this repository so that you can build your own new tests on top of 
kapture.  Over time, please do file issues if you feel strongly that we should modularize/helmify our deployments
to support a broader range of test types.

The possibilities are endless !

## How do I clean my cluster up?

Just run `./kapture.sh kapture-spam-my-namespace --delete`!  Kapture will take acre of the rest.

### Example Kapture projects to create new load tests

- Modify the kafka volumes to use persistent storage. (note we probably will support this soon.. not sure yet though).
- Increase the CPU and Memory parameters for Kafka
- Change the memory foot print or scaling factor of Redis
- Add a different load generator container than BigPetStore

## Scaling

Right now, both Kafka and the load generator can be scaled up.  To scale up Kafka:
* `kubectl scale --replicas=<REPLICA_COUNT> -f kafka.yml`

To increase the amount of load on the system, run: `kubectl scale --replicas=<REPLICA_COUNT> -f load-gen.yml`

# WHY IS THIS JAVA ?

As popular as Golang is in the infrastructure universe, the fact is that enterprises ship Java code at massive scales,
and the goal of Kapture is to simulate enterprise enviornments for upstream kubernetes load and scale testing.

So, yeah, we've got some java in here.

The goal of Kapture is tightly focused on simulating enterprise kubernetes applications at large scales, anything
largely outside of that relm, including generic load testing for performance SLAs and so on, might be more well
serviced by other scale and density tests in the upstream Kuberentes community.

# Future

- Dockerization of the kapture container with
  - kubectl (poor mans operator)
  - gradle : to build kafka connectors 
  - jvm : to support running bps, gradle, etc  
  
# License

All source in this Repo is Apache Commmons 2.0 licensed, please contribute ideas, code, whatever.  
We'll have prototypes up and running soon.

# Dev policy

- Push to master
- Accept almost any PR, this is a work in progress
- Ideas that are half baked : Merge them to contrib/ :)
