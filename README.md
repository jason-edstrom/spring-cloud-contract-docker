## Spring Cloud Contract Docker ##

This repo contains a docker image that can generate and execute contract tests using [Spring Cloud Contract](https://cloud.spring.io/spring-cloud-contract/).

## Short intro to JARs

TODO: Describe `groupId`, `artifactId`, `jar` and JAR storage (e.g. Artifactory)

## How it works

Contracts need to be mounted under `/contracts` folder

Output available under `/spring-cloud-contract/build` folder.

### Setup Env Vars

- `PROJECT_GROUP` - group id
- `PROJECT_VERSION` - version
- `PROJECT_NAME` - artifact id
- `REPO_WITH_BINARIES_URL` - e.g. Artifactory URL
- `REPO_WITH_BINARIES_USERNAME` - optional username
- `REPO_WITH_BINARIES_PASSWORD` - optional password
- `PUBLISH_ARTIFACTS` - if set to `true` then will publish artifact to binary storage

### Test Env Vars

- `APPLICATION_BASE_URL` - url against which tests should be executed.
Remember that it has to be accessible from the Docker container
- `APPLICATION_USERNAME` - optional username for basic authentication
- `APPLICATION_PASSWORD` - optional password for basic authentication

## Example of usage

Build locally a Docker image (step required until the image gets published)

```bash
$ git clone https://github.com/marcingrzejszczak/spring-cloud-contract-docker/
$ cd spring-cloud-contract-docker
$ docker build -t spring-cloud-contract-docker .
```

Clone a nodejs application (in this case it has contracts defined)

```bash
$ git clone https://github.com/marcingrzejszczak/bookstore/
$ cd bookstore
$ git checkout sc-contract
$ git submodule init
$ git submodule update
```

IMPORTANT: For now, before you continue, you need to build the Docker image yourself. Just execute `docker build -t spring-cloud-contract-docker .`

### Server side (nodejs)

Run the infra (in real life scenario you would just
run the nodejs application with mocked services). It will run mongodb and artifactory.

```bash
$ ./setup_infra.sh
```

Run the nodejs application (it will start on port `3000`)

```bash
$ node app
```

Run the contract tests. Your contracts will be taken from `/contracts` folder. The output of the test execution is available under `node_modules/spring-cloud-contract/output`. The stubs will be uploaded to artifactory

```bash
$ ./run_contract_tests.sh
```

### Client side (curl)

Let's now imagine that you're the client side. It could be another node js application. In our case it will be simple `curl`. Let's start with running the `Stub Runner Boot` application. (Ultimately we will provide a JAR to run, for the sake of testing we will need to build that jar).

```bash
$ ./run_stub_runner_boot.sh
```

What's happening is that a standalone Stub Runner application got started and it downloaded the stubs from Artifactory. The stubs are running at port `9876` (check the bash script for more info).

Time to check our stateful stubs.

```bash
$ cd json
# let's execute the first request (no response is returned)
$ ./1_request.sh
# Now time for the second request. Let's pipe it to jq if you have it to pretty print it
$ ./2_request.sh | jq
```

## How to build it?

```bash
$ docker build -t spring-cloud-contract-docker .
```
