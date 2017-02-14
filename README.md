![Mercator](https://raw.githubusercontent.com/LendingClub/mercator/master/.assets/noun_7987_sm.png) 

# mercator

[![CircleCI](https://circleci.com/gh/LendingClub/mercator.svg?style=svg)](https://circleci.com/gh/LendingClub/mercator)
[![Download](https://api.bintray.com/packages/lendingclub/OSS/mercator/images/download.svg) ](https://bintray.com/lendingclub/OSS/mercator/_latestVersion)

Mercator creates graph-model projections of physical, virtual and cloud infrastructure.

Mercator uses [Neo4j](https://neo4j.com/), a particularly awesome open-source graph database.  

It is intended to be used as a library inside of other services and tools.

# Quick Start

There are two options for getting your feet wet with Mercator.

## Docker

There is a docker image that is able to scan your AWS infrastructure.  To run it:

```bash
$ docker run -it -p 7474:7474 -p 7687:7687 -v ~/.aws:/root/.aws lendingclub/projector-demo
```

The container will fire up a neo4j database inside the container and then invoke Mercator against AWS.

In order for this to work, you need to have read-only credentials in your ~/.aws directory.  When you launch the 
container with the command above, it will make your credentials available to Mercator running inside the container. 
It will use those credentials to perform the scan of your AWS infrastructure.

## Java

If you don't like the Docker approach and you can install Neo4j (not hard), you can run the same AWS demo with gradle.

```bash
$ cd projector-demo
$ ../gradlew run
...
```

It will use the AWS credentials stored in $HOME/.aws and scan your AWS infrasturcutre.  It will connect to Neo4j at
http://localhost:7474

# Usage

## Core Configuration

The [Mercator](https://github.com/LendingClub/mercator/blob/master/mercator-core/src/main/java/org/lendingclub/mercator/core/Mercator.java) class is the cornerstone of the
Mercator project.  It exposes configuration and a REST client for interacting with Neo4j.

Mercator doesn't use Spring, Dagger, Guice or any other DI framework.  That is your choice.  Mercator is intended to be simple and straightorward to use.

To create a Mercator instance that connects to Neo4j at http://localhost:7474 with no username or password:

```java
Projector projector = new BasicProjector();
```

The following configuration options can be passed through the ```Projector(Map config)``` constructor:

| Property Name | Description | Default Value |
|---------------|-------------|---------------|
| neo4j.url     |  URL for connecting to Neo4j REST API | http://localhost:7474 |
| neo4j.username|  Username for authentication | N/A |
| neo4j.password|  Password for authentication | N/A |

Usage is fairly self-explanatory:

```java
Map<String,String> config = new HashMap<>();
config.put("neo4j.url","https://localhost:7473");
config.put("neo4j.username","myusername");
config.put("neo4j.username","mypassword");

Projector projector = new BasicProjector(config);
```


## AWS

Scanning an AWS region involves running something like the following:

```java
BasicProjector projector = new BasicProjector();

AllEntityScanner scanner = projector
    .createBuilder(AWSScannerBuilder.class)
    .withRegion(Regions.US_WEST_2)
    .build(AllEntityScanner.class);
scanner.scan();

```

This will:

1. Use the credentials found in ```${HOME}/.aws/credentials``` 
2. Construct an AWS client
3. Connect to the ```US_WEST_2``` region
4. Enumerate each entity and build a Neo4j graph model

After the scanner has been constructed, it can be used indefinitely.

## VMWare

Mercator can build a graph of VMWare entities with the following bit of code:

```java
BasicProjector projector = new BasicProjector();

VMWareScanner scanner = projector.createBuilder(VMWareScannerBuilder.class)
			.withUrl("https://myvcenter.example.com/sdk")
			.withUsername("myusername")
			.withPassword("mypassword").build();

scanner.scan();
```

## GitHub

Both public GitHub and GitHub Enterprise are supported:

```java

GitHubScanner scanner = projector
	.createBuilder(GitHubScannerBuilder.class)
    .withToken("oauthtoken").build();

scanner.scanOrganization("Apache");
````

OAuth2, Username/Password, and Anonymous access are all supported.


## Jenkins

Mercator will not only scan Jenkins, but it will create relationships to GitHub repos as well!

```java
JenkinsScanner scanner = projector
	.createBuilder(JenkinsScannerBuilder.class).withUrl("https://jenkins.example.com")
	.withUsername("myusername").withPassword("mypassword").build();

scanner.scan();
```
