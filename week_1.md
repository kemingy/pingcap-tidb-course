1. PD

Clone the repo: `git clone https://github.com/pingcap/pd.git`

Build: `make`

Run:
```sh
./bin/pd-server --name="pd" \
                --data-dir="pd" \
                --client-urls="http://localhost:2379" \
                --peer-urls="http://localhost:2380" \
                --log-file=pd.log
```

2. TiKV

Clone the repo: `git clone https://github.com/tikv/tikv.git`

Install `cmake`: `conda install -c anaconda cmake`

If you are using your laptop, open the air conditioner.

Build: `make`

Change the max number of file descriptors: `sudo launchctl limit maxfiles 100000 100000` for the current session (macOS 10.15.6).

Run:
```sh
./target/release/tikv-server --pd-endpoints="127.0.0.1:2379" \
                             --addr="127.0.0.1:20160" \
                             --data-dir=tikv1 \
                             --log-file=tikv1.log

./target/release/tikv-server --pd-endpoints="127.0.0.1:2379" \
                             --addr="127.0.0.1:20161" \
                             --data-dir=tikv2 \
                             --log-file=tikv2.log

./target/release/tikv-server --pd-endpoints="127.0.0.1:2379" \
                             --addr="127.0.0.1:20162" \
                             --data-dir=tikv3 \
                             --log-file=tikv3.log
```

Check the store status: `./bin/pd-ctl store -u http://127.0.0.1:2379`

3. TiDB

Clone the repo: `git clone https://github.com/pingcap/tidb.git`

Add log when creates a new transaction.

Check [commits](https://github.com/pingcap/tidb/commit/1e78e1b6d4bfbe99a662236fdbd0c9ee9383e809)

Build: `make`

Install MySQL cli: `pip install mycli`

Run:
```sh
./bin/tidb-server --store tikv \
                  --log-file tidb.log \
                  --path localhost:2379
```

Connect to TiDB: `mycli -h 127.0.0.1 -P 4000 -u root test`


## Refs:

* https://www.youtube.com/watch?v=OlO3R5ESFuU
* https://github.com/pingcap/tidb-docker-compose/blob/master/config/tidb.toml
