---
layout: post
title: How to setup a django cluster with a postgresql database
---

# GKE cluster and CloudSQL, a saga

Yupp, I said saga. So you wanna make a GKE cluster with Django and a CloudSQL (postgres) backend, hold on for the ride.

Alright so you're here and reading this so I'm assuming you're struggling with [this](https://cloud.google.com/python/django/kubernetes-engine).

Here are the prereqs for this post:
* you have a django app
* you can build a container of that app
* you're ok with using postgresql as the SQL backend

If you don't you have one you can use [this](https://github.com/fergyfresh/bookface)

## Setup project on Google Cloud.

### Create project
First thing we need to do before we can setup the project we are going to want to Create a project in order to group all of our cloud components.

You can create a project at https://cloud.console.google.com after you create an account in the top left of the screen. See screenshot below:
![Create Project](/assets/gke-blog/create-project.png)

### Create CloudSQL instance

Create a postgres CloudSQL with a private IP since that is what we are going to need when connecting GKE to cloudSQL, you can technically do it with it being a public IP, but then you need to run a `cloud_sql_proxy` inside of the django container, so let's not get into that right now, this is actually easier.
![Create CloudSQL](/assets/gke-blog/create-sql.png)

After you've created the CloudSQL instance we're going to want to create a database user and a database for our django project and grant `CREATE`, `READ`, `UPDATE`, `DELETE` to a user that I'll call `myproject-user` to the  database we will call `myproject-db`.

The things we need to remember from this are the private IP address (can be found under the `CONNECTION` tab. Database name was `myproject-db` and the database user was `myproject-user` and the password that you set when you created the user. We will need this stuff later.

### Pushing docker image to Google docker registry (GKE cluster deployment prereq)

Before we create a cluster, we are going to want to figure out what your project ID is in order to use the Google cloud docker registry `gcr.io`. Here is how you would do it if you were in the directory of the Dockerfile for the django app:

```
docker build . -t gcr.io/<project-id>/django
docker push gcr.io/<project-id>/django
```

This will give us an image to deploy to our cluster when we get there.

### Creating a cluster

A cluster is a set of nodes that will run your services (instances of workloads, we will get to this).

I suggest if this is your first time to create the smallest physical node type and only create one node. I understand this kinda defeats the purpose, but it will save you money while you're still figuring everything out. Also clicking the `Your first cluster` template is also a good start, but we need to click the `Availability, networking, security, and additional features` link near the bottom of the cluster settings, and we are looking to ENABLE the setting VPC-native.
![VPC Native](/assets/gke-blog/vpc-native.png)

### Deploy a workload

After you click deploy under the workload tab you'll get a window that has nginx in the image section, we are gonna wanna use our own. In order to do this you'll want to click the blue SELECT link and navigate to the image we created earlier and here is where I will need you to fill in your environment variables that you set in section above that I said are things to remember.
![Workload finish](/assets/gke-blog/workload-finish.png)

### Exposing a service to the world.

If we go into the Services section and select the service for the workload we just deployed we can PORT Forward the django port to port 80 or whatever port you want!

### Important Key takeaways.

The easiest way I could get this to work was to have a VPC-Native enabled GKE cluster and a CloudSQL instance that has the Private IP section enabled.

Support articles 
* [Django on GKE](https://cloud.google.com/python/django/kubernetes-engine)
* [Connecting to PostgresQL from Kubernetes](https://cloud.google.com/sql/docs/postgres/connect-kubernetes-engine)

### If you messed up Environment variables

If you think you might have setup an environment variable incorrectly go to the `Configuration` section and you can find a Config Map called `<service-name>-config`.

# Questions, comments, concerns

Feel free to click one of these buttons in order to signal me with something that was messed up with this. I'd be glad to fix anything that didnt work for you.
