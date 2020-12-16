# Initial page



## Introduction

### Zendro

Zendro is a software tool to quickly create a data warehouse tailored to your specifications. You tell Zendro what the structure of your data is, in the form of JSON-formatted models, and where the data is or shall be stored through a simple database configuration file. Zendro will then automatically create two standardized interfaces for your data models.

#### Unified standard API

Both interfaces provide access to the standard CRUD \(create, read, update, and delete\) functions, that will be available for each of the defined data models. One of the two interfaces is an intuitive graphical browser-based single page application implemented in Google’s standard material design. The other is an exhaustive application programming interface built with Facebook’s efficient GraphQL framework, enabling a connection to your data warehouse from any programming language or data analysis pipeline with utmost ease, simply by sending HTTP requests to your GraphQL server.

#### Distributed data in a cloud

Data can be distributed over several databases and servers without losing the relationships between your data records. This means, that an arbitrary network of any number of Zendro-BrAPI Servers can be set up and all data will be accessible from any of these nodes. Importantly, data records can have associations with other records stored on different servers, e.g. the "Maize drought resistance meta-study" Study can be stored on node "A" and be associated with "Yield" Observations stored on servers "B", "C", ..., "X".

Find out more on the official [documentation](https://zendro-dev.github.io/) and [github](https://github.com/Zendro-dev).

### Zendro-BreedingAPI

The Zendro-BreedingAPI aims to implement specifically the _schema_, i.e. the data models as they are defined in the [BreedingAPI](https://brapi.org/) and make them accessible through the graphql API provided by Zendro. This enables exhaustive and standardized **CRUD** access to the data in form of the above described two interfaces without the need to define the "API" part of the BreedingAPI directly yourself. All functionality and code is generated by Zendro, there is _no_ need for _manual programming work_.

{% hint style="info" %}
The schemas defined in this Sandbox are a prototype. For now, only essential parts of BrAPI-core and BrAPI-phenotyping as well as the germplasm schema are implemented.
{% endhint %}

For an overview of the implemented data models and their associations, have look at the following UML \(also included as `pdf` in the Sandbox\).

## Prerequisites

The following software tools are necessary to run the Sandbox

* [docker](https://docs.docker.com/)
* [docker-compose](https://docs.docker.com/compose/install/)
* _Note_, that this project is meant to be used on a `*nix` system, preferably Linux.

## Setup

### Download the Sandbox

The extracted folder consists of a few important files and folders:

* `BreedingAPI_model_definitions:`JSON definitions of the used BrAPI schemas that are fed into zendro to generate the API as well as the corresponding interfaces.
* `BreedingAPI_models_UML.pdf`: high-resolution pdf of the above UML diagram
* `graphql-server`: The heart of Zendro. The backend source code for the API, generated by Zendro
* `single-page-app`: Source code for the REACT-spa, generated by Zendro
* `graphiql-auth`: Source code for the browser-based, IDE-like query interface based on Facebook's [GraphiQL](https://github.com/graphql/graphiql), with added authentication.
* `docker-compoe.yml`: docker-compose configuration to run the services.

### Run the Sandbox

Start the data-warehouse by running:

```bash
docker-compose up --force-recreate --remove-orphans -d   
```

This will build and run four different docker images

1. **PostgreSQL:** Database to store the data.
2. **GraphQL-server:** Server to manage and access the data.
3. **GraphiQL-interface:** IDE-like interface in the browser to access the API directly by building and executing graphql queries.
4. **REACT-SPA:** graphical Browser single-page-application that lets the user intuitively browse and manage the data.

Additionally, a docker-network and a docker-volume to persist the stored data are created.

The process of installing and running the docker containers will take a few minutes the first time it is run. Once the command finishes all the containers are up and running.

Note, that other relational databases are supported, and in the near future Mongo-DB and Cassandra can be used, too. Also, data models can be distributed over different storages.

#### Cleanup

To stop the docker-compose run

```bash
docker-compose down
```

If needed, delete the created containers, network, volume, and images with the appropriate docker commands.

To remove docker containers execute:

```text
docker ps -a | grep zendro-breedingapi-sandbox | awk '{print "docker rm " $1}' | sh
```

To remove the docker images execute:

```text
docker images | grep zendro-breedingapi-sandbox | awk '{print "docker rmi " $1}' | sh
```

To delete the volumes _permanently_ in which your data has been stored execute:

```text
docker volume ls | grep zendro-breedingapi-sandbox | awk '{print "docker volume rm " $2}' | sh
```

## Access the services

When all docker-containers are up and both the REACT-SPA and the GraphiQL browser interfaces finished compiling, the services can be accessed on the browser. \(On first startup this can take up to five minutes.\) The GraphiQL IDE-like query interface will be located at `http://localhost:7000`, the REACT-SPA at `http://localhost:8080`.

The default user to login is:

```text
user: admin@zendro.org
password: admin
```

### REACT single page application

We tried to make the graphical user interface as intuitive as possible by using web standards and google's material UI design. In principle, the UI is divided in three main interfaces:

* **Table view:** When clicking on a model in the side-bar, a table-view of records is presented. You can sort, order, paginate, and fuzzy search directly from here.
* **Detail view:** When clicking on a record, you can explore the attributes and the associations to other models for that specific record.
* **Update/Create:** Similar to the detail view, you can create new records and set associations directly in the browser.

### GraphiQL - Programmatic Interface

You can build your queries and mutations using Facebook's [GraphQL](https://graphql.org/). The interface is self-documenting, with all available API functions detailed in the top-right Documentation.

#### Simple Examples

To add a new study a graphql-mutation like this can be used:

```graphql
mutation{
  addStudy(
  studyDbId:"myStudy_1",
  active:true,
  commonCropName:"maize",
  studyName:"maizeStudy_1",
  studyDescription:"This study is cool") {
    studyDbId #Note that graphql requires you to specify what attributes it should return
    studyName
  }
}
```

A new trial associated with the above study could be added like this:

```graphql
mutation{
  addTrial(
  trialDbId:"myTrial_1",
  trialName:"maizeTrial_1" ,
  active:true,
  commonCropName:"maize",
  addStudies:["myStudy_1"]){ #directly associated the afore created study to the new trial
    trialDbId
  }
}
```

An example of a query that queries the newly created trial and finds all associated studies that have a "commonCropName" `like` "aize" could look like this:

```graphql
{
  readOneTrial(trialDbId:"myTrial_1"){
    trialDbId # Again the attributes have to be specified
    trialName
    studiesFilter(
    pagination:{limit:10}, # giving a pageSize is mandatory to avoid querying huge amounts of data 
    search: {field: commonCropName, value:"%aize%", operator:like}) { # define the search
      studyDbId
      active
      studyName
      commonCropName
    }
  }
}
```



{% hint style="info" %}
Note that the above detailed API is independent of the data model. Meaning that every data models CRUD functions are standardized and _exactly_ the same!
{% endhint %}

