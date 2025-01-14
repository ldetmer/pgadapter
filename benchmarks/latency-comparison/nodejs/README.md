# Latency Comparison - node-postgres driver vs Cloud Spanner Node.js Client Library

This benchmark compares the latency of executing a simple, single-row query using the PostgreSQL
`node-postgres` driver against PGAdapter compared to executing the same query using the [native Cloud Spanner
Node.js client library](https://www.npmjs.com/package/@google-cloud/spanner).

## Setup

You must first create a database with a test table before you can run this benchmark.
See [the setup instructions here](../README.md#setup-test-database) for how to do that.

Also make sure that you have set the following environment variables:

```shell
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
export GOOGLE_CLOUD_PROJECT=my-project
export SPANNER_INSTANCE=my-instance
export SPANNER_DATABASE=my-database
```

## Run using Docker

The easiest way to run the benchmarks is to build and run them in a Docker container. This will ensure that:
1. All dependencies are available to the system (e.g. Node, Java, PGAdapter).
2. The benchmark application and PGAdapter are running on the same network. Running PGAdapter in a Docker container while the benchmark application is running on the host machine will add significant latency, as the Docker host-to-Docker network bridge can be slow.
3. Everything is automatically started for you.

```
docker build . --tag benchmark
docker run \
  -t --rm \
  -v ${GOOGLE_APPLICATION_CREDENTIALS}:/credentials.json:ro \
  --env GOOGLE_APPLICATION_CREDENTIALS=/credentials.json \
  --env GOOGLE_CLOUD_PROJECT \
  --env SPANNER_INSTANCE \
  --env SPANNER_DATABASE \
  benchmark \
    --clients=32 \
    --operations=5000
```


## Run directly on the host machine

You can run the benchmark as a Node.js application directly on your host machine if you have Node
and Java installed on your system. This requires you to first start PGAdapter:

```shell
wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter.tar.gz \
  && tar -xzvf pgadapter.tar.gz
java -jar pgadapter.jar
```

Then open a separate shell to execute the benchmark:

```shell
npm start -- --clients=32 --operations=5000
```

## Arguments

The benchmark application accepts the following command line arguments:
* -d, --database: The fully qualified database name to use for the benchmark. Defaults to `projects/$GOOGLE_CLOUD_PROJECT/instances/$SPANNER_INSTANCE/databases/$SPANNER_DATABASE`.
* -c, --clients: The number of parallel clients that execute queries. Defaults to 16.
* -o, --operations: The number of operations (queries) that each client executes. Defaults to 1,000.
* -t, --transaction: The type of transaction to execute. Must be either READ_ONLY or READ_WRITE. Defaults to READ_ONLY.
* -w, --wait: The wait time in milliseconds between each operation. Defaults to 0.
* -e, --embedded (true/false): Whether to start PGAdapter as an embedded test container together with the
  benchmark application. Defaults to false.
* -h, --host: The host name where PGAdapter runs. This argument is only used if `embedded=false`. Defaults to `localhost`.
* -p, --port: The port number where PGAdapter runs. This argument is only used if `embedded=false`. Defaults to `5432`.
* -s, --uds (true/false): Also execute benchmarks using Unix Domain Sockets. Defaults to false.
* -f, --dir: The directory where PGAdapter listens for Unix Domain Socket connections. Defaults to `/tmp`. This argument is only used if `embedded=false`.
* -u, --udsport: The port number where PGAdapter listens for Unix Domain Socket connections. Defaults to
  the same port number as for TCP.  This argument is only used if `embedded=false`.
* -m, --warmup: The number of warmup iterations to run before executing the actual benchmark. PGAdapter is a Java application,
  and Java uses a Just-in-Time compiler that compiles and optimizes code based on the actual usage pattern. Running a
  warmup script before the actual benchmark is therefore recommended to get results that are comparable to an actual
  application that runs for a longer period of time. The same warmup is also executed for the Spanner Node.js client
  library, as Javascript also uses a Just-in-Time compiler. Defaults to 12,000 operations per CPU core.

## Examples

Run a benchmark with 32 parallel clients each executing 5,000 operations:

```shell
npm start -- --clients=32 --operations=5000
```


Run a benchmark with 32 parallel clients each executing 1 query per second.
Each client executes 100 queries:

```shell
npm start -- --clients=32 --operations=100 --wait=1000
```


Run a benchmark with 32 parallel clients executing read/write transactions.
Each client executes 1,000 transactions:

```shell
npm start -- --clients=32 --operations=1000 --transaction=READ_WRITE
```
