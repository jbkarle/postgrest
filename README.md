### Purpose
Created this image in order to quickly deploy a pod service with standard `stable/postgresql` helm chart in the cluster including postgres *and* postgrest services together for dev/test purposes.

These services can be used in multiple ways, for example a test populating the postgres DB via `psql` or via `postgrest` api and other tests being able to retrieve the data via `REST calls`

If data need to be persisted, a PVC storage can be added into the helm chart for this purpose.

### Features
* Using standard `stable/postgresql` helm chart but changing to a custom docker image
* Both `postgrest`(3000) and `postgres`(5432) tcp ports exposed to the cluster
* Demo database included in the `postgres-postgrest` image with multiple tables (citiy, country, countrylanguage)
* Docker image running as a non root user and based on postgres:9.6 image
* anon/postgres users created with all privileges (readonly recommended once deployed in non dev environments)
* Default pvc configured for persistent storage

### Get started on minikube
```
eval $(minikube docker-env)
docker build . -t postgres-postgrest:latest
helm install stable/postgresql --name postgres-postgrest --namespace mynamespace -f values-prod.yaml
```

Just note both postgres and postgrest ports will be opened in order to interact with other pods in the cluster:

```kubectl -n mynamespace get services```
```
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
postgres-postgrest   ClusterIP   10.105.192.105   <none>        5432/TCP,3000/TCP   7m1s
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

(from another pod)

```curl -I http://postgres-postgrest.mynamespace.svc.cluster.local:3000```
```
HTTP/1.1 200 OK
Date: Sat, 07 Mar 2020 14:12:35 GMT
Server: postgrest/6.0.0 (dd86fe3)
Content-Type: application/openapi+json; charset=utf-8
```

```curl "http://postgres-postgrest.mynamespace.svc.cluster.local:3000/country?select=name,continent,region&name=eq.Estonia"```
```
[[{"name":"Estonia","continent":"Europe","region":"Baltic Countries"}]
```