Load Reporting Service (LRS) {#install_sandboxes_load_reporting_service}
============================

This simple example demonstrates Envoy\'s Load Reporting Service (LRS)
capability and how to use it.

Lets say Cluster A (downstream) talks to Cluster B (Upstream) and
Cluster C (Upstream). When enabling Load Report for Cluster A, LRS
server should be sending LoadStatsResponse to Cluster A with
LoadStatsResponse.Clusters to be B and C. LRS server will then receive
LoadStatsRequests (with total requests, successful requests etc) from
Cluster A to Cluster B and from Cluster A to Cluster C.

In this example, all incoming requests are routed via Envoy to a simple
goLang web server aka http_server. We scale up two containers and
randomly send requests to each. Envoy is configured to initiate the
connection with LRS Server. LRS Server enables the stats by sending
LoadStatsResponse. Sending requests to http_server will be counted
towards successful requests and will be visible in LRS Server logs.

Running the Sandbox
-------------------

### Step 3: Build the sandbox

Terminal 1 :

    $ pwd
    envoy/examples/load_reporting_service
    $ docker-compose pull
    $ docker-compose up --scale http_service=2

Terminal 2 :

    $ pwd
    envoy/examples/load_reporting_service
    $ docker-compose ps

                                Name                               Command               State                           Ports
    --------------------------------------------------------------------------------------------------------------------------------------
    load-reporting-service_http_service_1   /bin/sh -c /usr/local/bin/ ... Up      10000/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:8081->8081/tcp
    load-reporting-service_http_service_2   /bin/sh -c /usr/local/bin/ ... Up      10000/tcp, 0.0.0.0:81->80/tcp, 0.0.0.0:8082->8081/tcp
    load-reporting-service_lrs_server_1     go run main.go                   Up      0.0.0.0:18000->18000/tcp

### Step 4: Start sending stream of HTTP requests

Terminal 2 :

    $ pwd
    envoy/examples/load_reporting_service
    $ bash send_requests.sh

The script above (`send_requests.sh`) sends requests randomly to each
Envoy, which in turn forwards the requests to the backend service.

### Step 5: See Envoy Stats

You should see

Terminal 1 :

    ............................
    lrs_server_1    | 2020/02/12 17:08:20 LRS Server is up and running on :18000
    lrs_server_1    | 2020/02/12 17:08:23 Adding new cluster to cache `http_service` with node `0022a319e1e2`
    lrs_server_1    | 2020/02/12 17:08:24 Adding new node `2417806c9d9a` to existing cluster `http_service`
    lrs_server_1    | 2020/02/12 17:08:25 Creating LRS response for cluster http_service, node 2417806c9d9a with frequency 2 secs
    lrs_server_1    | 2020/02/12 17:08:25 Creating LRS response for cluster http_service, node 0022a319e1e2 with frequency 2 secs
    http_service_2  | 127.0.0.1 - - [12/Feb/2020 17:09:06] "GET /service HTTP/1.1" 200 -
    http_service_1  | 127.0.0.1 - - [12/Feb/2020 17:09:06] "GET /service HTTP/1.1" 200 -
    ............................
    lrs_server_1    | 2020/02/12 17:09:07 Got stats from cluster `http_service` node `0022a319e1e2` - cluster_name:"local_service" upstream_locality_stats:<locality:<> total_successful_requests:21 total_issued_requests:21 > load_report_interval:<seconds:1 nanos:998411000 >
    lrs_server_1    | 2020/02/12 17:09:07 Got stats from cluster `http_service` node `2417806c9d9a` - cluster_name:"local_service" upstream_locality_stats:<locality:<> total_successful_requests:17 total_issued_requests:17 > load_report_interval:<seconds:1 nanos:994529000 >
    http_service_2  | 127.0.0.1 - - [12/Feb/2020 17:09:07] "GET /service HTTP/1.1" 200 -
    http_service_1  | 127.0.0.1 - - [12/Feb/2020 17:09:07] "GET /service HTTP/1.1" 200 -
    ............................
    lrs_server_1    | 2020/02/12 17:09:09 Got stats from cluster `http_service` node `0022a319e1e2` - cluster_name:"local_service" upstream_locality_stats:<locality:<> total_successful_requests:3 total_issued_requests:3 > load_report_interval:<seconds:2 nanos:2458000 >
    lrs_server_1    | 2020/02/12 17:09:09 Got stats from cluster `http_service` node `2417806c9d9a` - cluster_name:"local_service" upstream_locality_stats:<locality:<> total_successful_requests:9 total_issued_requests:9 > load_report_interval:<seconds:2 nanos:6487000 >
