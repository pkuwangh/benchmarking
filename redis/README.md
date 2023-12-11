# Redis

## Build Redis

```bash
cd redis/
make

# to build with TLS support
make BUILD_TLS=yes
# to build with systemd support
make USE_SYSTEMD=yes

# to fix problem with dependencies
make distclean
```

## Run Redis

```bash
# server
./redis/src/redis-server --port 7777 --loglevel notice
# to bind on one specific core
numactl --physcpubind=1,65 ./redis/src/redis-server --port 7777 --loglevel notice

# cli
# stats
./redis/src/redis-cli -p 7777 -i 5 --stat
# reset stats
./redis/src/redis-cli -p 7777 flushdb
# list all keys
./redis/src/redis-cli -p 7777 keys \*
```

## benchmarking

### redis-benchmark

```bash
# simple
./redis/src/redis-benchmark -h 127.0.0.1 -p 7777
# addtional args
# -r <key space range>, default to rand_int
# -q <quiet>, only QPS
# -c <# clients>
# -n <# reqs>
# -d <size>
# -P <pipelined # reqs per client req>
# -t <commands>, e.g. `SET,GET`
# -l <loop forever>

# sweep connections
./redis/src/redis-benchmark -h 127.0.0.1 -p 7777 -c <1/5/10/50/100> -n 2000000 -t SET -d 100 -q
# sweep packet size
./redis/src/redis-benchmark -h 127.0.0.1 -p 7777 -c 50 -n 2000000 -t SET -d <2/100/1000/10000/50000> -q
```

#### Key take-aways

- QPS improves with number of clients but saturate quickly after 10s of connections.
- QPS is stable for 1~10KB data size, after that QPS decreases with increasing data size.
  - MTU size is a factor here; `ifconfig | grep -i MTU`
  - LLC usage & miss increases as data size increase.
- QPS scales with pipeline depth almost linearly with small data size.
  - Again MTU size is a factor.

### performance profiling

#### LLC and BW monitoring

```bash
cd ../tools/intel-cmt-cat/
make
sudo LD_LIBRARY_PATH=lib ./pqos/pqos -m 'all:1,65' -i 50 -o uarch-mon.txt
```
