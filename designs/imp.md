just go to raw mode and everything would be visible

Note: understand usage of Message-queue v/s pub-sub v/s map-reduce usage
Note: understand differnce b/w web server, applicarion server and client

*Problem solving Round*
1. Product requirements: Ask questions as PM to see future requirements (relate the problem with already existing real-world problem and solutions) 
2. Tech requirements: Ask questions as EM to check extended feature scope (logs, analytics, highly available system with low latency apis), API requirements and all possible cases
3. Information flow: Explain all possible use cases, each with all steps from end to end
4. APIs - request and response
5. Database schema tables and relationship
6. SQL queries




*SQL Write-Master-Slave and read-replica architecture* 

1. Write happens in master and read happens from slave replicas.
2. master-slave replication is lazy but eventual consistent.
2. shard-slave replication for same geo is lazy and will always have 80/20 data ratio.
3. Also, If the master shard fails, an automatic failover happens and the slave shard is promoted to be the new master shard.
4. Read requests gets load balanced across all shard-slaves for the geo from where request is being made.
5. All shard slave for same geo will be accesible via load balancer with consistent hashing algo for read requests.
6. First read from geo cache (cache_ind), if not available read from geo shard-slave (shard_slave_india) using consistent hashing at load balancer, then update geo_cache (cache_ind). 
 
 
 
 
                                                     API GATEWAY ----------------DNS--------CDN
                                                  /      |       \
                                                /        |          \
       SLB<-----------------CLB<------------READ_API    WRITE_ASYNC    WRITE_SYNC-----> active_master_global
(LB for geo-shards)   (LB for geo-caches)               (Schedulers)
                                                              | 
                                                       Message Queues
                                                              |
                                                        active_master_global & object_store(S3)

passive_master_local-------------------------active_master_local
(stand_by 1-1000)                                 (write 1-1000)
                                               /         |        \
                                         /               |              \
                                    /                    |                    \
                    shard_master_india           shard_master_uk             shard_master_usa
                     (lazy write 1-333)          (lazy write 334-666)            (lazy write 667-1000)
                           /    \                        /    \                           /    \
                          /      \                                                       /      \
          shard_slave_india.......\.... .<consistent_hashing@LB1>..........shard_slave_usa      \
            (read 1-333)           \                                       (read 667-1000)        \
                                    \                                                              \
                                   shard_slave_india........<consistent_hashing@LB2>................shard_slave_usa 
                                   (read 1-333)                                                  (read 667-1000)
         
        
 
 
 
 
 
 
 
 
 
 
 
 
