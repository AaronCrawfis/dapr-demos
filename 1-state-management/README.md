# Dapr State Management Demo

This demo will walk you through getting Dapr up and running on your system and trying out state management.

## Step 1: Initialize Dapr

Using the Dapr CLI, initialize the Dapr runtime to install the Dapr Docker containers:

```bash
dapr init
```

Verify the Dapr containers have been initialized by running `docker ps`:

```
$ docker ps

CONTAINER ID   IMAGE               COMMAND                  CREATED             STATUS                       PORTS                              NAMES
3a231973748e   daprio/dapr         "./placement"            About an hour ago   Up About an hour             0.0.0.0:6050->50005/tcp            dapr_placement
2fffd1266c88   redis               "docker-entrypoint.s…"   About an hour ago   Up About an hour             0.0.0.0:6379->6379/tcp             dapr_redis
8fa8de1afad7   openzipkin/zipkin   "start-zipkin"           About an hour ago   Up About an hour (healthy)   9410/tcp, 0.0.0.0:9411->9411/tcp   dapr_zipkin
```

Verify the default Dapr component have been created by opening the Dapr components folder:
- **Windows**: %UserProfile%\.dapr\components
- **Linux/macOS**: ~/.dapr/components

```
$ ls -l ~/.dapr/components

-rw-r--r-- 1 vscode vscode 193 Mar 16 12:35 pubsub.yaml
-rw-r--r-- 1 vscode vscode 240 Mar 16 12:35 statestore.yaml
```

Open `statestore.yaml` to see the connection information for the previously initialized Redis container:

```
$ ~/.dapr/components/statestore.yaml

apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
  - name: actorStateStore
    value: "true"
```

## Step 2: Run a Dapr sidecar

Create a new Dapr sidecar without an accompanying application to test out HTTP API calls on port 3500:

```bash
dapr run --app-id sidecar --dapr-http-port 3500
```

In a new terminal window verify the container is running with `dapr list`:

```
$ dapr list

APP ID   HTTP PORT  GRPC PORT  APP PORT  COMMAND  AGE  CREATED              PID
sidecar  3500       40285      0                  13s  2021-03-16 12:38.15  1538
```

## Step 3: Save state with Dapr

Now that you have a sidecar up and running you can use the Dapr sidecar's HTTP API to save and get state.

Begin by saving state using the following command in a terminal or command prompt:

```http
curl -X POST -H "Content-Type: application/json" -d '[{ "key": "name", "value": "Bruce Wayne"}]' http://localhost:3500/v1.0/state/statestore
```

## Step 4: Retrieve state with Dapr

Verify you have successfully saved state by retrieving it from the state store:

```http
curl http://localhost:3500/v1.0/state/statestore/name
```

## Step 5: See how state is stored in Redis

You can look at how Dapr stores application state within Redis by dumping the keys the redis CLI with `redis-cli`:

```
$ redis-cli

keys *