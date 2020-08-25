Environments:

* Vagrant + VirtualBox
* CPU: Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz
* Memory: DDR4 32GB * 8
* Disk: ext4


## TiUP

* doc: https://docs.pingcap.com/zh/tidb/stable/production-deployment-using-tiup

* version: v1.0.9

## Config

```vagrant
Vagrant.configure("2") do |config|

  config.vm.box = "hashicorp/bionic64"

  config.vm.define :pd do |pd|
    pd.vm.hostname = "pingcap.pd"
    pd.vm.network :private_network, ip: "192.168.0.1"
    pd.vm.network "forwarded_port", guest: 2379, host: 2379, host_ip: "127.0.0.1"
    pd.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"
    pd.vm.network "forwarded_port", guest: 3000, host: 9093, host_ip: "127.0.0.1"
    pd.vm.network "forwarded_port", guest: 3000, host: 8249, host_ip: "127.0.0.1"
    pd.vm.provider :virtualbox do |vb|
      vb.memory = "10240"
      vb.cpus = 4
    end
  end

  config.vm.define :tikv1 do |kv|
    kv.vm.hostname = "pingcap.kv1"
    kv.vm.network :private_network, ip: "192.168.0.2"
    kv.vm.network "forwarded_port", guest: 20160, host: 20160, host_ip: "127.0.0.1"
    kv.vm.provider :virtualbox do |vb|
      vb.memory = "40960"
      vb.cpus = 6
    end
  end

  config.vm.define :tikv2 do |kv|
    kv.vm.hostname = "pingcap.kv2"
    kv.vm.network :private_network, ip: "192.168.0.3"
    kv.vm.network "forwarded_port", guest: 20161, host: 20161, host_ip: "127.0.0.1"
    kv.vm.provider :virtualbox do |vb|
      vb.memory = "40960"
      vb.cpus = 6
    end
  end

  config.vm.define :tikv3 do |kv|
    kv.vm.hostname = "pingcap.kv3"
    kv.vm.network :private_network, ip: "192.168.0.4"
    kv.vm.network "forwarded_port", guest: 20162, host: 20162, host_ip: "127.0.0.1"
    kv.vm.provider :virtualbox do |vb|
      vb.memory = "40960"
      vb.cpus = 6
    end
  end

  config.vm.define :tidb1 do |db|
    db.vm.hostname = "pingcap.db1"
    db.vm.network :private_network, ip: "192.168.0.5"
    db.vm.network "forwarded_port", guest: 4000, host: 4000, host_ip: "127.0.0.1"
    db.vm.provider :virtualbox do |vb|
      vb.memory = "40960"
      vb.cpus = 6
    end
  end

  config.vm.define :tidb2 do |db|
    db.vm.hostname = "pingcap.db2"
    db.vm.network :private_network, ip: "192.168.0.6"
    db.vm.network "forwarded_port", guest: 4001, host: 4001, host_ip: "127.0.0.1"
    db.vm.provider :virtualbox do |vb|
      vb.memory = "40960"
      vb.cpus = 6
    end
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo echo "fs.file-max=200000" >> /etc/sysctl.conf
    sudo sysctl -p
    sudo echo "* soft nofile 200000" >> /etc/security/limits.conf
    sudo echo "* hard nofile 200000" >> /etc/security/limits.conf
    sudo echo "session required pam_limits.so" >> /etc/pam.d/common-session
  SHELL
end

```

topology:

```yaml
global:
    user: "tidb"
    ssh_port: 22
    deploy_dir: "/tidb-deploy"
    data_dir: "/tidb-data"

pd_servers:
    - host: pd

tidb_servers:
    - host: tidb1
      port: 4001
    - host: tidb2
      port: 4001

tikv_servers:
    - host: tikv1
      port: 20160
    - host: tikv2
      port: 20161
    - host: tikv3
      port: 20162

monitoring_servers:
    - host: pd

grafana_servers:
    - host: pd

alertmanager_servers:
    - host: pd
```

generate ssh config: `vagrant ssh-config >> ~/.ssh/config`

use TiUP to build the cluster: `tiup cluster deploy tidb-test v4.0.0 ./topology.yaml --user vagrant --native-ssh`

run the cluster: `tiup cluster start --native-ssh tidb-test`

error

```
2020-08-25T10:21:01.710+0800›   INFO›   Execute command finished›   {"code": 1, "error": "failed to start pd: \tpd pd:
2379 failed to start: timed out waiting for port 2379 to be started after 2m0s, please check the log of the instance:
timed out waiting for port 2379 to be started after 2m0s", "errorVerbose": "timed out waiting for port 2379 to be started
after 2m0s\ngithub.com/pingcap/tiup/pkg/cluster/module.(*WaitFor).Execute\n\tgithub.com/pingcap/tiup@/pkg/cluster/module/
wait_for.go:90\ngithub.com/pingcap/tiup/pkg/cluster/spec.PortStarted\n\tgithub.com/pingcap/tiup@/pkg/cluster/spec/
instance.go:99\ngithub.com/pingcap/tiup/pkg/cluster/spec.(*instance).Ready\n\tgithub.com/pingcap/tiup@/pkg/cluster/spec/
instance.go:130\ngithub.com/pingcap/tiup/pkg/cluster/operation.startInstance\n\tgithub.com/pingcap/tiup@/pkg/cluster/
operation/action.go:477\ngithub.com/pingcap/tiup/pkg/cluster/operation.StartComponent.func1\n\tgithub.com/pingcap/tiup@/
pkg/cluster/operation/action.go:513\ngolang.org/x/sync/errgroup.(*Group).Go.func1\n\tgolang.org/x/sync@v0.0.0-
20190911185100-cd5d95a43a6e/errgroup/errgroup.go:57\nruntime.goexit\n\truntime/asm_amd64.s:1357\n\tpd pd:2379 failed to
start: timed out waiting for port 2379 to be started after 2m0s, please check the log of the instance\nfailed to start
pd"}
```

