### Purpose
Created this helm chart in order to quickly deploy `postgrest` and helm chart child dependencies as well (and custom config) for dev/test purposes:
* stable/postgresql
* stable/jenkins

This chart can be used in multiple ways, for example a test populating the postgres DB via `psql` or rest calls to the `postgrest` api and other tests being able to retrieve the data via `REST calls`

`postgresql` data is persisted using kubernetes pvc and configured to remain by default (instead of delete using helm delete).

### Features
* Using standard `stable/postgresql` and `stable/jenkins` as helm chart child dependency
* Both `postgrest`(3000) and `postgres`(5432) tcp ports exposed to the cluster
* Helm 3 compatible
* Demo database included in the `postgrest` helm chart initdb scripts (citiy, country, countrylanguage)
* Postgrest vanilla docker image
* anon/postgres users created with all privileges (recommended changing permissions once deployed in non dev environments)
* pvc configured for postgresql persistent storage

### Get started on minikube and helm3
```
eval $(minikube docker-env)
docker build . -t postgrest:latest
helm install postgrest . --namespace mynamespace
```

Just note both postgres and postgrest ports will be opened in order to interact with other pods in the cluster:

```kubectl -n mynamespace get services```
```
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
postgrest                       ClusterIP   10.98.28.168     <none>        3000/TCP    10m
postgrest-jenkins               ClusterIP   10.110.35.16     <none>        8080/TCP    10m
postgrest-jenkins-agent         ClusterIP   10.106.63.76     <none>        50000/TCP   10m
postgrest-postgresql            ClusterIP   10.104.167.184   <none>        5432/TCP    10m
postgrest-postgresql-headless   ClusterIP   None             <none>        5432/TCP    10m
```

```
postgres-postgrest-helm % kubectl -n mynamespace exec -it postgres-postgrest-f8ffdf864-n8b4n psql
postgres=# \dt
              List of relations
 Schema |      Name       | Type  |  Owner   
--------+-----------------+-------+----------
 public | city            | table | postgres
 public | country         | table | postgres
 public | countrylanguage | table | postgres
 ```
### Testing
(from another pod)

```curl -I http://postgrest.mynamespace.svc.cluster.local:3000```
```
HTTP/1.1 200 OK
Date: Sat, 07 Mar 2020 14:12:35 GMT
Server: postgrest/6.0.0 (dd86fe3)
Content-Type: application/openapi+json; charset=utf-8
```

```curl "http://postgrest.mynamespace.svc.cluster.local:3000/country?select=name,continent,region&name=eq.Estonia"```
```
[[{"name":"Estonia","continent":"Europe","region":"Baltic Countries"}]
```
