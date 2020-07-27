# EC2-Prometheus-Graphana

### Spin up EC2 instances which need to be monitored

Install and Run NodeExporters on these instances. (NodeExporter acts as an agent for Prometheus to poll resources)

* Download Node Exporter
```
sudo su root

wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
````

* Extract the package 
```
tar xvfz node_exporter-1.0.1.linux-amd64.tar.gz
````

* Run NodeExporter as a service
```
nohup ./node_exporter-1.0.1.linux-amd64/node_exporter &
````

* Verify if the NodeExporter is running by checking the url on a browser.
```
http://<ec2-instance-ip>:9100
````


### Setup Prometheus as docker container on a different EC2 instance

* Update packages and install docker
```
sudo yum update -y

sudo amazon-linux-extras install docker
````
* Start docker service
```
sudo service docker start
````
* Add ec2-user as a docker user
```
sudo usermod -a -G docker ec2-user
````
* Pull docker image
```
docker pull prom/prometheus
````
* Create prometheus.yml file at location ~/
```
vi ~/prometheus.yml
````
* Add IPs of ec2 instances that need to be monitored
```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
  external_labels:
    monitor: 'codelab-monitor'
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    scrape_interval: 10s
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['localhost:9090']
  
  - job_name: 'node1'
    scrape_interval: 5s
    static_configs:
    - targets: ['<NodeExporterInstanceIP>:9100']
  - job_name: 'node2'
    scrape_interval: 5s
    static_configs:
    - targets: ['<NodeExporterInstanceIP>:9100']
````

* Run prometheus docker container
```
docker run -d -p 9090:9090 -v ~/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
````
