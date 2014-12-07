# CoreOS Kubernetes Vagrant + Heapster Monitoring

Adds monitoring manifests to `/home/core/monitoring`.

See: https://github.com/GoogleCloudPlatform/heapster

**Important**

At time of writing Kubernetes HEAD is required. You need to build it and put all the binaries in a folder `opt/bin` **inside** this directory.

## Usage

		$ vagrant up
		...
		$ vagrant ssh core-01 -- -A
		$ cd monitoring
		$ kubecfg -c influx-grafana-pod.json create pods
		...
		$ kubecfg -c influx-grafana-service.json create services
		...
		$ kubecfg -c heapster-pod.json create pods
		...
		$ kubecfg list pods
		Name                Image(s)                                                                            Host                Labels              Status
		----------          ----------                                                                          ----------          ----------          ----------
		influx-grafana      kubernetes/heapster_influxdb,kubernetes/heapster_grafana,dockerfile/elasticsearch   172.17.8.103/       name=influxdb       Running
		heapster            kubernetes/heapster                                                                 172.17.8.104/       name=heapster       Running

In this example the UX is available on 172.17.8.103:80 (check the ip of the node running influx-grafana).

The default credentials are **admin**:**admin**.

## Notes

It can take a long time before the pods are ready! That's why I set the config to only spawn one master and one node.

To speed things up, provided you have a local docker client running, you can pre-pull the docker images used by the pods before starting the vm:

		$ docker pull google/cadvisor
		$ docker pull kubernetes/heapster
		$ docker pull kubernetes/heapster_grafana
		$ docker pull kubernetes/heapster_influxdb
		$ docker pull dockerfile/elasticsearch

Then save them:

		$ docker save -o ./monitoring/images \
		  google/cadvisor \
		  kubernetes/heapster \
		  kubernetes/heapster_grafana \
		  kubernetes/heapster_influxdb \
		  dockerfile/elasticsearch

The nodes are configured to load the images from this archive upon starting if the file is present (still takes a bit of time).
