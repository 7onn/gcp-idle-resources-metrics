# gcp-idle-resources-metrics
Identify unused resources at Google Cloud Platform through Prometheus' metrics


## Usage

Set up a service account on the project you want to monitor. To comprehend all collectors' required permissions, you have to grant: 
- `roles/compute.viewer`
- `roles/dataproc.viewer`

You can authenticate by setting the [Application Default Credentials](https://developers.google.com/accounts/docs/application-default-credentials) (i.e: Placing the service account's JSON key and setting the environment variable `GOOGLE_APPLICATION_CREDENTIALS=path-to-credentials.json`) or letting the application automatically load the credentials from metadata ([Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) is recommended).

You must set at least the project ID and the regions you want to monitor. Either by: 
- Specifying through command args `--project_id  --regions us-east1,us-central1`  
- Specifying through environment variables `GCP_PROJECT_ID= GCP_REGIONS=us-east1,us-central1` (if authenticating through metadata, the project doesn't need to be specified)


## Development building and running
Prerequisites:
* [Go compiler](https://golang.org/dl/)

Building:
```bash
make build
```

Running:
```bash
./server -h
./server --project-id=x --regions=us-central1,us-east1
```

Running tests
```bash
make test
```
## Collectors

Current supported APIs:
- Google Compute Engine
  - [Instances](https://console.cloud.google.com/compute/instances)
  - [Disks](https://console.cloud.google.com/compute/disks)
  - [Snapshots](https://console.cloud.google.com/compute/snapshots)
- Dataproc
  - [Clusters](https://console.cloud.google.com/dataproc/clusters)

To enable only some specific collector(s):
```bash
./server --collector.disable-defaults --collector.gce_is_disk_attached --collector.gce_is_old_snapshot
```


### Docker
```bash
cp ~/.config/gcloud/application_default_credentials.json ./credentials.json

chmod 444 credentials.json

docker build -t gcp-idle-resources-metrics . 

docker run -it --rm --network=host \
  -v $(pwd)/credentials.json:/credentials.json \
  -e GOOGLE_APPLICATION_CREDENTIALS=/credentials.json \
  -e GCP_PROJECT_ID= \
  -e GCP_REGIONS=us-east1,us-central1,southamerica-east1 \
  gcp-idle-resources-metrics
```
Check the exported [metrics](http://localhost:5000/metrics).


### Kubernetes
Add the Chart repository
```bash
helm repo add 7onn https://www.7onn.dev/helm-charts
helm search repo 7onn
```

Export its default values
```bash
helm show values 7onn/gcp-idle-resources-metrics > values.yaml
```

Edit the values according to your needs then install the application
```bash
helm upgrade -i gcp-idle-resources-metrics --values values.yaml 7onn/gcp-idle-resources-metrics
```
