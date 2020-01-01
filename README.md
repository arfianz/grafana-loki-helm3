# Using Helm3 to Install Grafana and Loki

**Helm3 is here**, and without the server side component Tiller. Let’s check it out!

## Download the binary

Check for latest beta releases on https://github.com/helm/helm/releases

I downloaded and copied it to:

```bash
$ sudo cp ~/helm /usr/local/bin/helm3
$ sudo chown root:root /usr/local/bin/helm3
$ helm3 version
version.BuildInfo{Version:"v3.0.2", GitCommit:"19e47ee3283ae98139d98460de796c1be1e3975f", GitTreeState:"clean", GoVersion:"go1.13.5"}
```

For more information about the changes in Helm3 checkout this link:

https://github.com/helm/community/blob/master/helm-v3/000-helm-v3.md

## Grafana

**Grafana** is the open source analytics & monitoring solution for every database.

```bash
$ helm3 repo add stable https://kubernetes-charts.storage.googleapis.com
NAME  	URL                                             
stable	https://kubernetes-charts.storage.googleapis.com
local 	http://127.0.0.1:8879/charts  

$ helm3 repo update
Hang tight while we grab the latest from your chart repositories...
...Unable to get an update from the "local" chart repository (http://127.0.0.1:8879/charts):
	Get http://127.0.0.1:8879/charts/index.yaml: dial tcp 127.0.0.1:8879: connect: connection refused
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```
We can just use the existing Helm2 charts (most of the time) and run:

```bash
$ kubectl create ns grafana
namespace/grafana created

$ helm3 upgrade --install grafana stable/grafana -n grafana
Release "grafana" does not exist. Installing it now.
NAME: grafana
LAST DEPLOYED: Thu Jan  2 04:35:39 2020
NAMESPACE: grafana
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:

   kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   grafana.grafana.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:

     export POD_NAME=$(kubectl get pods --namespace grafana -l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace grafana port-forward $POD_NAME 3000

3. Login with the password from step 1 and the username: admin
#################################################################################
######   WARNING: Persistence is disabled!!! You will lose your data when   #####
######            the Grafana pod is terminated.                            #####
#################################################################################
```
### Namespaces are important now

```bash helm ls``` won’t show anything, we have to specify the namespace with it:

```bash
$ helm3 -n grafana ls
NAME   	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
grafana	grafana  	1       	2020-01-02 04:35:39.638115275 +0700 WIB	deployed	grafana-4.2.2	6.5.2
```

### Access Grafana Interface

Now we can get the admin password and create a port-forward:

``` bash
$ kubectl get secret -n grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode
WKcXPtOeqm7EgVTUallU8GlqUuxB6aQylDJzLHct

$ kubectl port-forward -n grafana service/grafana 3000:80
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

Open http://localhost:3000 to access the Grafana interface. And CTRL+C to end port-forward.

## Loki

**Loki** is a horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by [Prometheus](https://prometheus.io/).

First we add the repo:

```bash
$ helm3 repo add loki https://grafana.github.io/loki/charts
"loki" has been added to your repositories

$ helm3 repo update
Hang tight while we grab the latest from your chart repositories...
...Unable to get an update from the "local" chart repository (http://127.0.0.1:8879/charts):
	Get http://127.0.0.1:8879/charts/index.yaml: dial tcp 127.0.0.1:8879: connect: connection refused
...Successfully got an update from the "loki" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
'''

Then we create a namespace and install it:

```bash
$ kubectl create ns loki-stack
namespace/loki-stack created

$ helm3 upgrade --install loki-stack -n=loki-stack loki/loki-stack
Release "loki-stack" does not exist. Installing it now.
NAME: loki-stack
LAST DEPLOYED: Thu Jan  2 04:53:20 2020
NAMESPACE: loki-stack
STATUS: deployed
REVISION: 1
NOTES:
The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.

See http://docs.grafana.org/features/datasources/loki/ for more detail.

$ helm3 -n loki-stack ls
NAME      	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART            	APP VERSION
loki-stack	loki-stack	1       	2020-01-02 04:53:20.241818342 +0700 WIB	deployed	loki-stack-0.24.0	v1.2.0     
```

Done. Helm3 looks promising!

## LICENSE

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

