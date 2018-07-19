---
title: "Chaincode Libraries"
date: 2018-07-13T16:09:42+05:30
---

The aim of this guide is to enable adding new software to chaincode containers.

For whatever reason, esp for research experimentation, there is a requirement to bundle/package some software into the chaincode container. This software could be extra libraries that can be used by your chaincode in some way. Maybe, it is in the same language that the chaincode is written in but cannot be vendored in. Maybe it is a special implmentation in some other language.

The core idea is this: boost the default `ccenv` image to include the required software.

### Approach 1: The easy way

This approach allows you to achieve the result by overriding the image used by Fabric peers at runtime.

##### Pros
Dynamically choose chaincode base. Good for experiments and one-off runs.

##### Cons
Used this way, there is a chance of mistakenly reaching a state where different peers have different chaincode bases. Not recommended for production or large setups.

#### Step 1: Create a Dockerfile packing your software

The only difficult step here is the base image. We need to use the image currently being used by Fabric ccenv as our base image.

We can find this version using something like:
```
docker images | grep "fabric-baseos"
```
The base image we need to use is the full name of the image that comes up including the tag. Something like,
`hyperledger/fabric-baseos:<arch>-<version>`.

Now we create a Dockerfile that looks like this:
```
FROM <name of baseimage above>

RUN <commands to install stuff you need here>
RUN <more stuff>
```

#### Step 2: Build your alternative ccenv container

We build a Docker images from this Dockerfile, using a command like:
```
docker build -t <name> .
```
Name can be something suitable to your setup, for eg, `myname/ccenv:latest`.

#### Step 3: Instruct your peers to use this ccenv as chaincode runtime

This is the final step. In you `docker-compose.yml` files or equivalent that is used to setup Fabric, there are sections pertaining to Fabric orderers, CAs, cli and peers. Often the peer services extend from a peer-base service that is used to store configuration common to all peers. We need to add an extra configuration here that tells all peer containers to use our new chaincode base environment. Thanks to the way Fabric is written, this can be done via an environment variable.

In the peer base yml file, all you need to add is an env variable like so in the environment section.
```
  - CORE_CHAINCODE_GOLANG_RUNTIME="myname/ccenv:latest"
```

That's it. Now when you start a network using the equiavalent of `docker-compose up`, new chaincodes should be spawned using the new bases. You can call programs installed into the new base from your chaincode.

You can add additional variables replacing GOLANG with JAVA or NODE for chaincodes written in the respective languages. You can also use different bases for different languages.

Note that you can also override the variable on a per peer basis in the docker compose files as needed to have different bases for different peers.

### Approach 2: Modify Fabric source

Useful in cases where you need to control a full set of Fabric peers and you would rather maintain a patched version of Fabric.

#### Pros
Good for production setup where all peers have to be uniform. Less chances of mistakes in runtime.

#### Cons
You will have to recompile fabric from source in this approach.

#### Step 1: Add your software to the base ccenv

TODO
#### Step 2: Prepare fabric peers

TODO

Authors: "Chander Govindarajan & Dandasai S Kotireddy"
