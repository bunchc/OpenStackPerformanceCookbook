# Chapter 1 - Before We Get Started

Now that you have a good idea of what this book aims to cover and how it will be laid out, we have one last mandatory chapter that will help put the rest of the book into context for you. In this chapter we will discuss at a high level the OpenStack projects and our testing environment(s). Additionally we will discuss our testing methodology and provide a framework for you to both benchmark your OpenStack installation up front as well as monitor it on an ongoing basis.

Specifically, in this chapter we will cover the following:
- OpenStack Overview
- Environment Overview
- Testing and Measuring Methodology
    + Rally
    + Ceilometer
- Security vs Performance

## OpenStack Overview

A full discussion of OpenStack is well beyond the scope of this book. In fact, there are huge amounts of great materials available to you to provide the foundational knowledge you will need before getting much deeper into this book. If that sounds like you, hop on over to openstack.org and get your learn on.

A discussion of OpenStack Performance would not be complete without first providing just enough of an overview of what OpenStack is. In that sense, we will start with a summary of OpenStack and touch on how we have broken it down into roles for easier consumption.

### What Is OpenStack?

** TODO: This needs something from openstack.org Also that Ken Pepple diagram

### The Roles

** TODO: This also needs something from openstack.org

## Environment Overview

In the context of a performance tuning book, it is important to know what equipment and environment the authors are using to setup the environment and provide a consistent platform for performing tests. Additionally, we wanted to be able to provide a representative environment that would target a middle ground. That is, not a super-sized Cern environment, nor a single laptop's devstack environment. Thus, we've settled on the following build as a starting point. If we make changes to this environment, we will call them out in the text.

- 1x Utility Server
- 2x Controller Nodes
- 2x Compute Nodes
- 1x Object Storage Node

** TODO: Network Diagram

### The Roles

In this environment, instead of calling out where each OpenStack project was installed, instead gathered them by complementary sets of projects that comprise a role.

These roles will serve as a starting point and a reference for us as we dig into performance tuning. However, you will see in later chapters, as we dissect the tunable aspects of each composite component, we will provide recommendations as to which services to scale independently of their respective role.

What follows is a brief discussion of what constitutes a given role:

#### Controller

The controller role provides all of the services that allow a user to manage the remainder of the OpenStack environment. That is, they provide the following:

- Horizon Dashboard
- RabbitMQ
- API Endpoints for
    + Nova
    + Heat
    + Ceilometer
    + Cinder
    + Neutron
- Nova-Conductor
- HA Proxy
- TODO: The rest

#### Compute Nodes

In an OpenStack environment, the compute nodes are the work horses. They are responsible for the actual running of instances. In our example they will run:

- Ubuntu 14.04
- KVM
- Open vSwitch
- Nova
- TODO: The rest

### The Equipment

** TODO: It would be nice to find a way to allow others to update this section with stuff they've run rally on.

## Testing and Measuring Methodology

### Rally

### Ceilometer

## Security vs Performance

While reading this book it is important to understand that security and performance are not mutually exclusive. In fact, when sizing compute hardware, changing from 10Gbit to 40Gbit networking, or making decisions on overcommit ratios, there is little security impact.

That said, we also realize that some performance decisions, such as encrypted ephemeral volumes, can have a big impact on both security and performance. Where possible, given time commitments and available gear for testing, we will provide benchmark numbers for what we consider to be the best setting for performance as well as the 'secure' setting. However, there is an extremely high possibility that we will miss something. As a reader, it is important for you to know that the omission was not intentional, and that this book was written with a performance first slant.