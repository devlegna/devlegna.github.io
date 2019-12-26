---


---

<p>Let’s make Galera Cluster with docker</p>
<ol>
<li>What’s the Galera Cluster? -&gt; Ask to google.</li>
<li>What’s docker? -&gt; Ask to google.</li>
</ol>
<p><strong>This article will not contain technical information.</strong><br>
<strong>I’m not good at English.</strong></p>
<p>Galera is a great solution for Multi-Master replication. But I think that you will face fear when you try setup Galera. There are so many unfamiliar words in the manual. So, I’ll explain easy way make Galera with docker.</p>
<p><strong>Requires</strong></p>
<pre><code>docker
docker-compose
</code></pre>
<p><strong>Config Files</strong></p>
<p>docker-compose.yml</p>
<pre><code>version: '2'
services:
  mariadb:
image: mariadb:10.4
container_name: mariadb
restart: always
environment:
  - TZ=Asia/Seoul
  - MYSQL_ROOT_PASSWORD=[PASSWORD]
ports:
  - 3306:3306
  - 4444:4444
  - 4567:4567
  - 4568:4568
volumes:
  - {/docker/mariadb/data}:/var/lib/mysql
  - {/docker/mariadb/conf}:/etc/mysql/conf.d
</code></pre>
<p><strong>!! Change variables with your environment</strong></p>
<p>conf/my.cnf</p>
<pre><code>[mysqld]
port=3306
</code></pre>
<p>conf/cluster.cnf [ for <strong>node1</strong> ]</p>
<pre><code>[galera]
binlog_format='ROW'
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_on = ON
wsrep_cluster_name=my-galera-cluster
wsrep_cluster_address='gcomm://[192.168.0.1,192.168.0.2]'
wsrep_sst_method=mariabackup
wsrep_sst_auth=[sstUser]:[sstPass]
wsrep_node_address=[192.168.0.1]
wsrep_sst_receive_address=[192.168.0.1]
wsrep_node_name=[db-01]
</code></pre>
<p><strong>!! Change variables with your environment</strong></p>
<p>conf/cluster.cnf [ for <strong>node2</strong> ]</p>
<pre><code>[galera]
binlog_format='ROW'
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_on = ON
wsrep_cluster_name=my-galera-cluster
wsrep_cluster_address='gcomm://[192.168.0.1,192.168.0.2]'
wsrep_sst_method=mariabackup
wsrep_sst_auth=[sstUser]:[sstPass]
wsrep_node_address=[192.168.0.2]
wsrep_sst_receive_address=[192.168.0.2]
wsrep_node_name=[db-02]
</code></pre>
<p><strong>!! Change variables with your environment</strong></p>
<p><strong>Start Node1</strong></p>
<ol>
<li>
<p>Rename cluster.cnf to cluster.cnf.x</p>
<pre><code> mv conf/cluster.cnf conf/cluster.cnf.x
</code></pre>
</li>
<li>
<p>Start docker for initialize Mariadb</p>
<pre><code> docker-compose up
</code></pre>
</li>
<li>
<p>Wait until Mariadb initialize complete</p>
</li>
<li>
<p>Stop container with Press Ctrl+C<br>
4-1. Restore your data if you need (Optional)</p>
</li>
<li>
<p>Rename cluster.cnf.x to cluster.cnf</p>
<pre><code> mv conf/cluster.cnf.x conf/cluster.cnf
</code></pre>
</li>
<li>
<p>Start docker</p>
<pre><code> docker-compose up -d
</code></pre>
</li>
<li>
<p>Initialize Galera Primary Node</p>
<pre><code> docker exec mariadb galera_new_cluster
</code></pre>
</li>
</ol>

