# PostgreSQL High-Availability Cluster :elephant: :sparkling_heart:

---

### Deploy a Production Ready PostgreSQL High-Availability Cluster (based on "Patroni" and "DCS(etcd)"). Automating with Ansible.

使用Ansible实现自动化部署一个生产就绪的PostgreSQLHigh-Availability集群（基于“Patenti”和“DCS（etcd）”。

本Ansible playbook设计用于在生产环境的专用物理服务器上部署PostgreSQL高可用性集群。Сluster可以部署在测试环境和小型项目的虚拟机中。

除了部署新的集群外，本手册还支持在现有和正在运行的PostgreSQL上部署集群。您可以将基本的PostgreSQL安装转换为高可用性集群。只需在清单文件中指定变量 postgresql_exists='true'。

**注意!** > :heavy_exclamation_mark: 在以群集模式运行之前，您的PostgreSQL将被停止（请计划短暂的数据库停机时间）。

> :heavy_exclamation_mark: 在用于生产之前，请在您的测试环境中进行测试。

**您有两种部署选项（“A型”和“B型”):**

### [TypeA] 带负载平衡的PostgreSQLHigh-Availability
![TypeA](images/TypeA.png)
> 要使用此方案，请在变量文件vars/main.yml中指定with_haproxy_load_balancing: true
> 该方案提供了在读取时分配负载的能力。这还允许我们扩展集群（使用read-only个副本）。

-	端口5000（读/写）主机
-	端口5001（只读）所有副本

> 如果变量 "synchronous_mode" 值为 'true' (vars/main.yml):

-	端口5002（只读）仅同步副本
-	端口5003（只读）仅异步副本

**注意:** 应用程序必须支持将读取请求发送到自定义端口（例如5001）和写入请求（例如5000）。

##### 高可用性组件：

[**Patroni**](https://github.com/zalando/patroni) 是一个模板，您可以使用Python和分布式配置存储（如ZooKeeper、etcd、Consor或Kubernetes）创建自己的定制high-availability解决方案，以实现最大的可访问性。用于自动管理PostgreSQL实例和自动故障切换。

[**etcd**](https://github.com/etcd-io/etcd) 是分布式系统中最关键数据的分布式可靠key-value存储。etcd是用Go编写的，并使用Raft共识算法来管理highly-available复制日志。Patenti使用它来存储有关集群状态和PostgreSQL配置参数的信息。

[什么是分布式共识?](http://thesecretlivesofdata.com/raft/)

##### 负载平衡组件:

[**HAProxy**](http://www.haproxy.org/) 是一个免费、非常快速和可靠的解决方案，为TCP和HTTP-based应用程序提供高可用性、负载平衡和代理。

[**confd**](https://github.com/kelseyhightower/confd) 使用来自etcd或Consor的模板和数据管理本地应用程序配置文件。用于自动化HAProxy配置文件管理。

[**Keepalived**](https://github.com/acassen/keepalived) 为数据库访问提供了虚拟high-availableIP地址（VIP）和单个入口点。在Linux上实现VRRP（虚拟路由器冗余协议）。在我们的配置中，keepalive检查HAProxy服务的状态，如果出现故障，则将VIP委托给集群中的另一台服务器。

[**PgBouncer**](https://pgbouncer.github.io/features.html) 是PostgreSQL的连接池。


### [Type B] 仅限PostgreSQLHigh-Availability
![TypeB](images/TypeB.png)

这是一个没有负载平衡Used by default的简单方案。

使用"vip-manager"为数据库访问提供单一入口点（VIP）。

vip-manager是一种在所有集群节点上启动并连接到DCS的服务。如果本地节点拥有leader-key，则vip-manager启动配置的VIP。在故障转移的情况下，vip-manager删除旧节点上的VIP，新节点上的相应服务再启动。

---
## 兼容性
基于RedHat和Debian的发行版(x86_64）

###### 支持的 Linux 版本:
- **Debian**: 9, 10, 11
- **Ubuntu**: 18.04, 20.04, 22.04
- **CentOS**: 7, 8
- **Oracle Linux**: 7, 8
- **Rocky Linux**: 8
- **AlmaLinux**: 8

###### 支持的 PostgreSQL 版本:
所有支持的PostgreSQL版本

:white_check_mark: tested, works fine: PostgreSQL 10, 11, 12, 13, 14

集群部署日常自动化测试结果表:
| Distribution | Test result |
|--------------|:----------:|
| Debian 9     | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(Debian%209))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28Debian+9%29%22) |
| Debian 10    | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(Debian%2010))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28Debian+10%29%22) |
| Debian 11    | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(Debian%2011))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28Debian+11%29%22) |
| Ubuntu 18.04 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(Ubuntu%2018.04))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28Ubuntu+18.04%29%22) |
| Ubuntu 20.04 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(Ubuntu%2020.04))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28Ubuntu+20.04%29%22) |
| Ubuntu 22.04 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(Ubuntu%2022.04))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28Ubuntu+22.04%29%22) |
| CentOS 7     | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(CentOS%207))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28CentOS+7%29%22) |
| CentOS 8     | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(CentOS%208))](https://github.com/vitabaks/postgresql_cluster/actions?query=workflow%3A%22scheduled+PostgreSQL+%28CentOS+8%29%22) |
| Oracle Linux 7 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(OracleLinux%207))](https://github.com/vitabaks/postgresql_cluster/actions/workflows/schedule_pg_oracle_linux7.yml) |
| Oracle Linux 8 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(OracleLinux%208))](https://github.com/vitabaks/postgresql_cluster/actions/workflows/schedule_pg_oracle_linux8.yml) |
| Rocky Linux 8 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(RockyLinux%208))](https://github.com/vitabaks/postgresql_cluster/actions/workflows/schedule_pg_rockylinux8.yml) |
| AlmaLinux 8 | [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/vitabaks/postgresql_cluster/scheduled%20PostgreSQL%20(AlmaLinux%208))](https://github.com/vitabaks/postgresql_cluster/actions/workflows/schedule_pg_almalinux8.yml) |

###### Ansible 版本
已经在Ansible 2.7、2.8、2.9、2.10、2.11上进行了测试。

## 前提要求
此playbook需要root权限或sudo。

## 端口号
必须为数据库群集打开的所需TCP端口列表:

- `5432` (postgresql)
- `6432` (pgbouncer;postgresql连接池，用于外部客户端能够缓存和PostgreSQL的连接)
- `8008` (patroni rest api,监控页面http://ip:8008/health)
- `2379`, `2380` (etcd)

此外，对于具有负载平衡的方案“[类型A]PostgreSQLHigh-Availability”:

-	`5000`（haproxy-（读/写）主机，可写入和读取主库）
-	`5001`（haproxy-（只读）所有副本,所有节点库可提供读取的负载均衡）
-	`5002`（haproxy-（只读）仅同步副本）
-	`5003`（仅限haproxy-（只读）异步副本）
-	`7000`（可选，haproxy stats）

## 推荐

- linux（操作系统）：

在部署之前更新目标服务器上的操作系统；

确保已配置时间同步（NTP）。如果要安装和配置ntp服务，`vars/system.yml`中请指定ntp_enabled: 'true'和ntp_servers。

-	DCS（分布式配置存储）：

快速的驱动器和可靠的网络是影响etcd集群性能和稳定性的最重要因素。

避免将etcd数据与密集使用磁盘子系统资源的其他进程（如数据库）存储在同一驱动器上！将etcd和postgresql数据存储在不同的磁盘上（请参阅etcd_data_dir变量），如果可能，请使用ssd驱动器。请参阅硬件建议和调整指南。
过载（高负载）数据库集群可能需要在与数据库服务器分离的专用服务器上安装etcd集群。

-	在不同数据中心放置集群成员：

如果您更喜欢cross-data中心设置，其中复制数据库位于不同的数据中心，那么etcd成员的放置就变得至关重要。

如果您想创建一个真正健壮的etcd集群，有很多事情需要考虑，但有一条规则：不要将所有etcd成员都放在主数据中心。请参阅一些示例。

-	如何在自动故障转移(synchronous_modes和pg_rewind)的情况下防止数据丢失：

由于性能原因，默认情况下禁用同步复制。
为了最大限度地降低自动故障切换时丢失数据的风险，您可以按以下方式配置设置：

> - synchronous_mode: 'true'
> -	synchronous_mode_strict: 'true'
> -	synchronous_commit: 'on' (or 'remote_apply')
> -	use_pg_rewind：“false”（默认启用）

---

## 部署：快速启动
1. 在一个控制节点上安装Ansible

   `sudo apt update`

   `sudo apt install python3-pip sshpass -y`

   `sudo pip3 install ansible`

2. 转到playbook目录

   `cd ansible-postgresql-cluster/`

3. 编辑inventory文件

 节点私有IP地址和连接配置 (`ansible_user`, `ansible_ssh_pass` or `ansible_ssh_private_key_file` 变量值)
   `vim inventory`

4. 编辑变量文件(./vars/main.yml)

   `vim vars/main.yml`

-	proxy_env  #如果需要（用于下载软件包）

    example:
```
   proxy_env:
       http_proxy: http://proxy_server_ip:port
       https_proxy: http://proxy_server_ip:port
```
-	cluster_vip#用于客户端访问集群中的数据库（可选）
-	patroni_cluster_name
-	patroni_superuser_username: "postgres"
-	patroni_superuser_password: "postgres-pass"  # please change password
-	with_haproxy_load_balancing'true'（类型A）或'false'/default（类型B）
-	postgresql_version
-	postgresql_data_dir

5. 尝试连接到主机

   `ansible all -m ping`

6. 执行playbook:
```
   echo "`who|head -1|awk '{print $1}'|tr -d ['\n']`  ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers.d/user

   ansible-playbook deploy_pgcluster.yml
```

---

## 	组变量

请参阅vars/[main.yml](./vars/main.yml)。有关详细信息，请参阅[system.yml](./vars/system.yml)和[Debian.yml](./vars/Debian.yml)或[RedHat.yml](./vars/RedHat.yml)文件。

## 集群拓展

#### 将新的postgresql节点添加到现有集群

成功部署PostgreSQL HA集群后，可能需要进一步扩展。为此，请使用`add_pgnode.yml`剧本。

**注意:** 此playbook不会扩展etcd集群和haproxy平衡器。
在运行这个剧本的过程中，新节点的准备方式将与第一次部署集群时相同。但与初始部署不同，所有必要的配置文件都将从主服务器复制。

#####	准备:

> 在集群中所有节点的`pg_hba.conf`文件中添加一个新节点（或子网）。

> 对所有PostgreSQL应用pg_hba.conf（请参见patronictl reload --help）。

###### 添加新节点的步骤：

> 转到playbook目录postgresql_cluster

      `ansible-postgresql-cluster`

> 编辑inventory文件
在[master]组中指定集群的一个节点的ip地址，并在[reploca]组中指定新节点（您要添加的节点）。


> 编辑变量文件

所有集群节点上应该相同的变量：with_haproxy_load_balancing， postgresql_version,postgresql_data_dir， postgresql_conf_dir。

> 运行 playbook:  `ansible-playbook add_pgnode.yml`

#### 添加新的haproxy平衡器节点

为此，请使用`add_balancer.yml`剧本。

在运行该playbook的过程中，新的平衡器节点将以与第一次部署集群时相同的方式进行准备。但与初始部署不同，所有必要的配置文件都将从[master]组中指定的服务器复制。

**注意:**在用于生产之前，请在您的测试环境中进行测试.


>	转到playbook目录

  `cd ansible-postgresql-cluster`

> 编辑inventory文件

  指定[master]组中一个现有平衡器节点的ip地址，以及[balancers]组中的新平衡器节点（您要添加的）。

**注意:**防火墙端口列表根据指定主机的组动态确定。如果您向[etcd_cluster]或[master]/[replica]组中的现有节点之一添加新的haproxy平衡器节点，则可以重写iptables规则！请参阅system.yml文件中的firewall_allowed_tcp_ports_for.balancers变量。

> 编辑 `main.yml` 变量文件

设置 `with_haproxy_load_balancing: true`

4. 运行playbook:

`ansible-playbook add_balancer.yml`

## 恢复和克隆
使用pgbackback或WAL-GPoint-In-Time-Recovery从现有备份创建新集群。

##### 使用PgBackRest创建集群：
> 编辑main.yml变量文件
```
patroni_cluster_bootstrap_method: "pgbackrest"

patroni_create_replica_methods:
  - pgbackrest
  - basebackup

postgresql_restore_command: "pgbackrest --stanza={{ pgbackrest_stanza }} archive-get %f %p"

pgbackrest_install: true
pgbackrest_stanza: "stanza_name"  # specify your --stanza
pgbackrest_repo_type: "posix"  # or "s3"
pgbackrest_repo_host: "ip-address"  # dedicated repository host (if repo_type: "posix")
pgbackrest_repo_user: "postgres"  # if "repo_host" is set
pgbackrest_conf:  # see more options https://pgbackrest.org/configuration.html
  global:  # [global] section
    - {option: "xxxxxxx", value: "xxxxxxx"}
    ...
  stanza:  # [stanza_name] section
    - {option: "xxxxxxx", value: "xxxxxxx"}
    ...

pgbackrest_patroni_cluster_restore_command:
  '/usr/bin/pgbackrest --stanza={{ pgbackrest_stanza }} --type=time "--target=2020-06-01 11:00:00+03" --delta restore'
```

> 运行 playbook:

`ansible-playbook deploy_pgcluster.yml`

##### 使用WAL-G创建集群:

>	编辑main.yml变量文件
```
patroni_cluster_bootstrap_method: "wal-g"

patroni_create_replica_methods:
  - wal_g
  - basebackup

postgresql_restore_command: "wal-g wal-fetch %f %p"

wal_g_install: true
wal_g_ver: "v0.2.15"  # version to install
wal_g_json:  # see more options https://github.com/wal-g/wal-g#configuration
  - {option: "xxxxxxx", value: "xxxxxxx"}
  - {option: "xxxxxxx", value: "xxxxxxx"}
  ...

```
> 运行 playbook:

`ansible-playbook deploy_pgcluster.yml`


##### Point-In-Time-Recovery:
您可以为PITR运行现有Patenti集群的自动恢复，在main.yml变量文件中指定所需参数，并使用标记运行playbook:
```
ansible-playbook deploy_pgcluster.yml --tags point_in_time_recovery
```
pgBackRest恢复过程:
```
1. Stop patroni service on the Replica servers (if running);
2. Stop patroni service on the Master server;
3. Remove patroni cluster "xxxxxxx" from DCS (if exist);
4. Run "/usr/bin/pgbackrest --stanza=xxxxxxx --delta restore" on Master;
5. Run "/usr/bin/pgbackrest --stanza=xxxxxxx --delta restore" on Replica (if patroni_create_replica_methods: "pgbackrest");
6. Waiting for restore from backup (timeout 24 hours);
7. Start PostgreSQL for Recovery (master and replicas);
8. Waiting for PostgreSQL Recovery to complete (WAL apply);
9. Stop PostgreSQL instance (if running);
10. Disable PostgreSQL archive_command (if enabled);
11. Start patroni service on the Master server;
12. Check PostgreSQL is started and accepting connections on Master;
13. Make sure the postgresql users (superuser and replication) are present, and password does not differ from the specified in vars/main.yml;
14. Update postgresql authentication parameter in patroni.yml (if superuser or replication users is changed);
15. Reload patroni service (if patroni.yml is updated);
16. Start patroni service on Replica servers;
17. Check that the patroni is healthy on the replica server (timeout 10 hours);
18. Check postgresql cluster health (finish).
```

**为什么禁用archive_command？**

这对于在归档WAL时避免归档日志存储中的冲突是必要的。当多个集群尝试将WAL发送到同一存储时。例如，当您从一个备份中创建群集的多个克隆时。
还原后，可以使用`patronictl edit-config`更改此参数。或者将`disable_archive_command: false`设置为在还原后不禁用archive_command。



## 维护

请注意，本剧本的最初设计目标更关注PostgreSQL HA集群的初始开发，因此它目前并不关注集群的持续维护。
您应该了解集群的每个组件，以便进一步维护。

>	- [教程：使用Patenti管理High-AvailabilityPostgreSQL集群](https://pgconf.ru/en/2018/108567)

> - [Patroni documentation](https://patroni.readthedocs.io/en/latest/)

> - [etcd操作指南](https://etcd.io/docs/v3.3.12/op-guide/)


## 灾难恢复

高可用性群集提供了一种自动故障切换机制，并不能覆盖所有灾难恢复场景。您必须自己负责备份数据。
#####	etcd
每次更改配置时，用户节点都会将DCS选项的状态转储到磁盘上，并将其放入Postgres数据目录中的文件patroni.dynamic.json。如果DCS中完全没有这些选项或这些选项无效，则允许主（用户领导）从on-disk转储恢复这些选项。
但是，我建议您阅读etcd群集的灾难恢复指南：

> -	[etcd灾难恢复](https://etcd.io/docs/v3.3.12/op-guide/recovery)
##### PostgreSQL (databases)

推荐以下备份和恢复工具：
* [pgbackrest](https://github.com/pgbackrest/pgbackrest)
* [pg_probackup](https://github.com/postgrespro/pg_probackup)
* [wal-g](https://github.com/wal-g/wal-g)

不要忘记验证您的备份(例如: [pgbackrest auto](https://github.com/vitabaks/pgbackrest_auto))。

## 卸载集群
如果你需要从头开始，使用剧本`remove_cluster.yml`。

为了防止脚本在生产环境中意外使用，请编辑`remove_cluster.yml`并删除*安全pin*。相应地更改这些变量：

> - remove_postgres: true
> - remove_etcd: true

运行脚本，所有数据都会消失。

`ansible-playbook remove_cluster.yml`

`yum remove postgresql*` 或者 `apt remove postgresql*`

`rm -rf /var/lib/postgresql`

现在可以从头开始安装。

**注意:** 小心不要在没有*安全pin*的情况下将此脚本复制到生产环境中。
