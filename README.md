# Custom-Fluentd
Fluentd collects the nginx ingress logs and will insert the data into the BigQuery.

Folder Structure:

fluentd_docker
|
 -- Dockerfile
 -- entrypoint.sh
 -- fluent.conf
|
fluentd_gke
|
 -- bigquery_schema
 -- ds.yml
 -- fluent.conf
 -- new-rbac.yml
 -- nginx-format
|
README.md

Build Docker Image

fluentd_docker floder is used to create the custom fluentd docker image with required plugins

Here i used bigquery plugin to send data to the bigquery, you can add the plugins that are required by editing the Dockerfile.

Commands to build the and push the Docker image to the container registery

$ docker build -t fluentd-custom:latest .

$ docker push fluentd-custom:latest


Deploy Fluentd On Kubernetes

Before deploying the fluentd on cluster, make sure to change the log_format on nginx.conf file.

log_format json '{ '
    '"time": "$time_iso8601", '
    '"remote_addr": "$remote_addr", '
    '"remote_user": "$remote_user", '
    '"request_url": "$request_uri", '
    '"request_method": "$request_method", '
    '"request_time": "$request_time", '
    '"response_status": "$status", '
    '"response_size": "$body_bytes_sent", '
    '"referrer": "$http_referer", '
    '"upstream_response_time": "$upstream_response_time", '
    '"agent": "$http_user_agent", '
    '"forwarded_for": "$http_x_forwarded_for", '
    '"host": "$host" '
'}';

access_log /var/log/nginx/access.log json;


Go to the google bigQuery.

Step. 1
create the project.

step. 2
create the dataset for the project.

step. 3
create the table for the dataset along with schema.

[
  {
    "name": "time",
    "type": "TIMESTAMP"
  },
  {
    "name": "remote_addr",
    "type": "STRING"
  },
  {
    "name": "remote_user",
    "type": "STRING"
  },
  {
    "name": "request_url",
    "type": "STRING"
  },
  {
    "name": "request_method",
    "type": "STRING"
  },
  {
    "name": "request_time",
    "type": "FLOAT"
  },
  {
    "name": "upstream_response_time",
    "type": "FLOAT"
  },
  {
    "name": "response_status",
    "type": "INTEGER"
  },
  {
    "name": "response_size",
    "type": "INTEGER"
  },
  {
    "name": "referrer",
    "type": "STRING"
  },
  {
    "name": "agent",
    "type": "STRING"
  },
  {
    "name": "forwarded_for",
    "type": "STRING"
  },
  {
    "name": "host",
    "type": "STRING"
  }
]


***
Create Service account for bigQuery and generate the json key.

Step. 1
Apply RBAC to the cluster.

$ kubectl apply -f new-rbac.yml

Step. 2
First create the secret from the BigQuery service account key.

$ kubectl create secret generic database --from-file=bigquery.json -n fd

step. 3
Create the configmap from fluent.conf file

$ kubectl create configmap fluentd-conf --from-file=fluent.conf -n fd

step. 4
Apply the fluentd daemonset, before applying make sure you change the docker image in the ds.yml file.

$ kubectl apply -f ds.yml -n fd

Now Nginx ingress logs starts to insert into the bigQuery.

***
You can use bigQuery as datasource and visualize the data.

Here i used Google Datastudio for visualization.

Refer the images folder for sample images.

 

