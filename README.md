# CoreOS Kubernetes Vagrant

**Currently broken**

Adds monitoring pods to `/home/core/monitoring`.

See https://github.com/GoogleCloudPlatform/kubernetes/tree/master/examples/monitoring

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

In this example the UX is available on 172.17.8.103:80

### Notes

It can take a long time before the pods are ready!