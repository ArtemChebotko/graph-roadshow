# Hands-on Lab at DataStax Graph Workshop

This repo contains a _KillrVideoGraph_ data set and DataStax Studio notebooks that were used for the hands-on session of DataStax Graph Workshop in New York, Washington D.C., Philadelphia, etc. You are welcome to transfer these artifacts into your own environment to continue working with the hands-on examples and problems.

## Data

- `data/killr_video_graph.json` is a pre-generated _KillrVideoGraph_ data set that can be imported into DataStax Graph 6.8 using Gremlin IO.
- `data/csv/*.csv` is a raw _KillrVideoGraph_ data set consisting of CSV files that can be loaded into DataStax Graph 6.8 using DataStax Bulk Loader _dsbulk_, DataStax Graph Frames or other methods.

## Notebooks

- `notebooks/Hands-On_Lab_on_Gremlin_Graph_Traversals.studio-nb.tar` is a hands-on notebook with Gremlin examples and problems that can be imported into DataStax Studio 6.8.
- `notebooks/Hands-On_Lab_on_Gremlin_Graph_Traversals_with_answers.studio-nb.tar` is a hands-on notebook with Gremlin examples,  problems and solutions that can be imported into DataStax Studio 6.8.

## All-Inclusive Image for Docker
[This Docker image](https://hub.docker.com/r/datastaxacademy/graph-roadshow) can quickly get you a running environment complete with DataStax Graph 6.8, DataStax Studio 6.8, data set, and notebooks. _Minimum resource requirements_ for [Docker](https://docs.docker.com) Engine: `6` CPUs and `8.0` GiB Memory (can be conveniently set in Docker Desktop > Preferences > Advanced).

1. Create a container based on the image 
```
   docker run -p 80:80 datastaxacademy/graph-roadshow:6.8
```

(Note that we chose to map container port 80 to host machine port 80. If port 80 is not available on your host, you can choose to map to a different port, e.g. `-p 8888:80`)

2. Access the notebooks in DataStax Studio by going to `localhost` or `localhost:80` in a web browser (preferably Chrome)

3. Get the container ID to be able to stop/start the container
```
   docker ps --all
     CONTAINER ID        IMAGE                                ...
     b4dd02fe403e        datastaxacademy/graph-roadshow:6.8   ...
   
   docker stop b4dd02fe403e
   
   docker start b4dd02fe403e
```

4. Remove the container when no longer needed
```
   docker stop b4dd02fe403e

   docker rm b4dd02fe403e
```

## Requirements for Your Own Deployment

At the Workshop, we used DataStax Graph 6.8 and DataStax Studio 6.8 that are available at [DataStax Labs](https://downloads.datastax.com/#labs).

1. Download and extract the DataStax Graph (DSG) archive from [DataStax Labs](https://downloads.datastax.com/#labs) - it contains both DataStax Graph 6.8 and DataStax Studio 6.8 distributions.
2. Modify _dse.yaml_ (e.g. `resources/dse/conf/dse.yaml`) to reflect the following settings:
  - Enable AlwaysOn SQL to be able to execute SQL queries
```
  alwayson_sql_options:
    # Set to true to enable the node for AlwaysOn SQL. Only an Analytics node
    # can be enabled as an AlwaysOn SQL node.
    enabled: true

    # AlwaysOn SQL Thrift port
    thrift_port: 10000

    # AlwaysOn SQL WebUI port
    web_ui_port: 9077

    # The waiting time to reserve the Thrift port if it's not available
    reserve_port_wait_time_ms: 100

    # The waiting time to check AlwaysOn SQL health status
    alwayson_sql_status_check_wait_time_ms: 500

    # The work pool name used by AlwaysOn SQL
    workpool: alwayson_sql

    # Location in DSEFS of the log files
    log_dsefs_dir: /spark/log/alwayson_sql

    # The role to use for internal communication by AlwaysOn SQL if authentication is enabled
    auth_user: alwayson_sql

    # The maximum number of errors that can occur during AlwaysOn SQL service runner thread
    # runs before stopping the service. A service stop requires a manual restart.
    runner_max_errors: 10

    # The interval in seconds to update heartbeat of AlwaysOn SQL. If heartbeat is not updated
    # for more than the period of three times of the interval, AlwaysOn SQL malfunctions.
    # AlwaysOn SQL automatically restarts.
    heartbeat_update_interval_seconds: 30
```
  - Increase the OLTP timeout setting to 300 to be able to load the data set
```
graph:
    # Maximum time to wait for an OLAP analytic (Spark) traversal to evaluate.
    # When not set, the default is 10080 minutes (168 hours).
    # analytic_evaluation_timeout_in_minutes: 10080

    # Maximum time to wait for an OLTP real-time traversal to evaluate.
    # When not set, the default is 30 seconds.
    realtime_evaluation_timeout_in_seconds: 300
```
  - Disable the Gremlin sandbox to be able to read the data set files
```
    gremlin_server:
        # port: 8182

        # Size of the worker thread pool. Should generally not exceed 2 * number of cores.
        # A worker thread performs non-blocking read and write for one or more Channels.
        # threadPoolWorker: 2

        # The number of "Gremlin" threads available to execute scripts in a ScriptEngine as well as bytecode requests.
        # This pool represents the workers available to handle blocking operations in Gremlin Server. When unset or set to zero,
        # this value will be defaulted to 10 times the value of the JVM property "cassandra.available_processors" (if set)
        # or to 10 times the value of Runtime.getRuntime().availableProcessors() (otherwise).
        # gremlinPool: 0

        # The gremlin-groovy script engine will always be added even if the configuration option is not present.
        # Additional imports may be added in the configuration for that script engine.
         scriptEngines:
            gremlin-groovy:
                 config:
                     # To disable the gremlin groovy sandbox entirely
                     sandbox_enabled: false
#                     sandbox_rules:
```
3. Start DataStax Graph with Search and Analytics enabled: e.g. `bin/dse cassandra -s -k -g`.
4. Start DataStax Studio: e.g. `bin/server.sh`.
5. Open a Web browser and navigate to `localhost:9091` to access DataStax Studio. Import the notebooks from this repo. The notebooks expect the data to be located at `/var/lib/graph/killr_video_graph.json`. Enjoy!
