# Consul

## Tips
* [Consul 手册](https://www.consul.io/docs/guides/)

```bash
# 启动单节点
# 开发模式
consul agent -dev -client 0.0.0.0

consul agent -bootstrap -server -data-dir $PWD/data -advertise 127.0.0.1 -ui
# ==> WARNING: Bootstrap mode enabled! Do not enable unless necessary
# ==> Starting Consul agent...
# ==> Starting Consul agent RPC...
# ==> Consul agent running!
#            Version: 'v0.7.3'
#            Node ID: 'f5bdf8b9-3e67-8b88-ddce-4052a5ae6bec'
#          Node name: 'wener'
#         Datacenter: 'dc1'
#             Server: true (bootstrap: true)
#        Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
#       Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
#     Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
#              Atlas: <disabled>


# 在容器中启动集群, 指定网卡是关键
docker run --rm -it \
  --networ service \
  -e CONSUL_CLIENT_INTERFACE=eth0 \
  -e CONSUL_BIND_INTERFACE=eth0 \
  consule agent -server -bootstrap-expect 3 \
  -retry-join consul_consul_1 -retry-join consul_consul_2 -retry-join consul_consul_3

# 查看成员信息
docker exec -it consul_consul_3 consul members -http-addr=http://consul_consul_3:8500

# 再启动一个用于做对外暴露, 启动 ui, 允许所有来源客户端访问
docker run -it --rm --network multi-host-network \
  -e CONSUL_BIND_INTERFACE=eth0 -p 8500:8500 \
  --name consul consul agent -server -retry-join consul_consul_1 -ui



# 启用 ACL
docker run -it --rm -p 8500:8500  -v $PWD/consul/data:/consul/data \
  -e 'CONSUL_LOCAL_CONFIG={"datacenter":"dc1","acl_datacenter": "dc1","acl_master_token": "b1gs33cr3t","acl_default_policy": "deny","acl_down_policy": "extend-cache"}' \
  -e CONSUL_CLIENT_INTERFACE=eth0 \
  -e CONSUL_BIND_INTERFACE=eth0 \
  --name consul consul agent -server -bootstrap-expect 1 -ui

# 客户端指定 Token
consul members -token b1gs33cr3t
CONSUL_HTTP_TOKEN=b1gs33cr3t consul members

curl -v --request PUT http://127.0.0.1:8500/v1/acl/bootstrap

docker run -d --name=dev-consul consul
docker run -d consul agent -dev -join=172.17.0.2
docker exec -t dev-consul consul members

docker run -d --net=host consul agent -server -bind=<external ip> -retry-join=<root agent ip> -bootstrap-expect=<number of server agents>

# Start cluster server
docker run -d --name=node0 consul agent -server -client=0.0.0.0 -node=node0 -bootstrap-expect=1 -bind=172.17.0.2 -data-dir=/tmp/consul

# Start client
docker run -d --name=node1 consul agent -client=0.0.0.0 -node=node1 -bind=172.17.0.3 -data-dir=/tmp/consul -join=172.17.0.2
```


## ACL
* ACL 可控制的资源主要有
  * agent	Utility operations in the Agent API, other than service and check registration
  * event	Listing and firing events in the Event API
  * key	Key/value store operations in the KV Store API
  * keyring	Keyring operations in the Keyring API
  * node	Node-level catalog operations in the Catalog API, Health API, Prepared Query API, Network Coordinate API, and Agent API
  * operator	Cluster-level operations in the Operator API, other than the Keyring API
  * query	Prepared query operations in the Prepared Query API
  * service	Service-level catalog operations in the Catalog API, Health API, Prepared Query API, and Agent API
  * session	Session operations in the Session API

```hcl
# Agent Token
node "" {
  policy="write"
}
service "" {
  policy="read"
}

# Client Token
node "" {
  policy="read"
}

# Traefik
key "traefik" {
  policy = "write"
}
session "" {
  policy = "write"
}
```

### Config

```js
{
  "datacenter": "east-aws",
  "data_dir": "/opt/consul",
  "log_level": "INFO",
  "node_name": "foobar",
  "server": true,
  "watches": [
    {
        "type": "checks",
        "handler": "/usr/bin/health-check-handler.sh"
    }
  ],
  "telemetry": {
     "statsite_address": "127.0.0.1:2180"
  }
}
```

### Helps

```
$ consul help
Usage: consul [--version] [--help] <command> [<args>]

Available commands are:
    agent          Runs a Consul agent
    catalog        Interact with the catalog
    event          Fire a new event
    exec           Executes a command on Consul nodes
    force-leave    Forces a member of the cluster to enter the "left" state
    info           Provides debugging information for operators.
    join           Tell Consul agent to join cluster
    keygen         Generates a new encryption key
    keyring        Manages gossip layer encryption keys
    kv             Interact with the key-value store
    leave          Gracefully leaves the Consul cluster and shuts down
    lock           Execute a command holding a lock
    maint          Controls node or service maintenance mode
    members        Lists the members of a Consul cluster
    monitor        Stream logs from a Consul agent
    operator       Provides cluster-level tools for Consul operators
    reload         Triggers the agent to reload configuration files
    rtt            Estimates network round trip time between nodes
    snapshot       Saves, restores and inspects snapshots of Consul server state
    validate       Validate config files/directories
    version        Prints the Consul version
    watch          Watch for changes in Consul
```

```
$ consul agent --help
==> Usage: consul agent [options]

  Starts the Consul agent and runs until an interrupt is received. The
  agent represents a single node in a cluster.

Options:

  -advertise=addr                  Sets the advertise address to use
  -advertise-wan=addr              Sets address to advertise on wan instead of
                                   advertise addr
  -atlas=org/name                  Sets the Atlas infrastructure name, enables
                                   SCADA.
  -atlas-join                      Enables auto-joining the Atlas cluster
  -atlas-token=token               Provides the Atlas API token
  -atlas-endpoint=1.2.3.4          The address of the endpoint for Atlas
                                   integration.
  -bootstrap                       Sets server to bootstrap mode
  -bind=0.0.0.0                    Sets the bind address for cluster
                                   communication
  -http-port=8500                  Sets the HTTP API port to listen on
  -bootstrap-expect=0              Sets server to expect bootstrap mode.
  -client=127.0.0.1                Sets the address to bind for client access.
                                   This includes RPC, DNS, HTTP and HTTPS (if
                                   configured)
  -config-file=foo                 Path to a JSON file to read configuration
                                   from. This can be specified multiple times.
  -config-dir=foo                  Path to a directory to read configuration
                                   files from. This will read every file ending
                                   in ".json" as configuration in this
                                   directory in alphabetical order. This can be
                                   specified multiple times.
  -data-dir=path                   Path to a data directory to store agent
                                   state
  -dev                             Starts the agent in development mode.
  -recursor=1.2.3.4                Address of an upstream DNS server.
                                   Can be specified multiple times.
  -dc=east-aws                     Datacenter of the agent (deprecated: use
                                   'datacenter' instead).
  -datacenter=east-aws             Datacenter of the agent.
  -encrypt=key                     Provides the gossip encryption key
  -join=1.2.3.4                    Address of an agent to join at start time.
                                   Can be specified multiple times.
  -join-wan=1.2.3.4                Address of an agent to join -wan at start
                                   time. Can be specified multiple times.
  -retry-join=1.2.3.4              Address of an agent to join at start time
                                   with retries enabled. Can be specified
                                   multiple times.
  -retry-interval=30s              Time to wait between join attempts.
  -retry-max=0                     Maximum number of join attempts. Defaults to
                                   0, which will retry indefinitely.
  -retry-join-ec2-region           EC2 Region to use for discovering servers to
                                   join.
  -retry-join-ec2-tag-key          EC2 tag key to filter on for server
                                   discovery
  -retry-join-ec2-tag-value        EC2 tag value to filter on for server
                                   discovery
  -retry-join-gce-project-name     Google Compute Engine project to discover
                                   servers in
  -retry-join-gce-zone-pattern     Google Compute Engine region or zone to
                                   discover servers in (regex pattern)
  -retry-join-gce-tag-value        Google Compute Engine tag value to filter
                                   for server discovery
  -retry-join-gce-credentials-file Path to credentials JSON file to use with
                                   Google Compute Engine
  -retry-join-wan=1.2.3.4          Address of an agent to join -wan at start
                                   time with retries enabled. Can be specified
                                   multiple times.
  -retry-interval-wan=30s          Time to wait between join -wan attempts.
  -retry-max-wan=0                 Maximum number of join -wan attempts.
                                   Defaults to 0, which will retry
                                   indefinitely.
  -log-level=info                  Log level of the agent.
  -node=hostname                   Name of this node. Must be unique in the
                                   cluster
  -node-meta=key:value             An arbitrary metadata key/value pair for
                                   this node.
                                   This can be specified multiple times.
  -protocol=N                      Sets the protocol version. Defaults to
                                   latest.
  -rejoin                          Ignores a previous leave and attempts to
                                   rejoin the cluster.
  -server                          Switches agent to server mode.
  -syslog                          Enables logging to syslog
  -ui                              Enables the built-in static web UI server
  -ui-dir=path                     Path to directory containing the Web UI
                                   resources
  -pid-file=path                   Path to file to store agent PID
```
