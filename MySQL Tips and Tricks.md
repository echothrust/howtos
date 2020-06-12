# MySQL Tips and Tricks

## Take full MySQL Backups
```
mysqldump -AYKER --add-drop-database --add-drop-table --hex-blob --triggers --tz-utc
```

**Legend**
```
-A All databases
-Y All table spaces
-K disable keys
-E Events
-R Routines
```

## Take only a single db


```
mysqldump -KER --add-drop-table --hex-blob --triggers --tz-utc

```

## Dump Data Only
```sh
mysqldump --complete-insert --extended-insert --hex-blob --tz-utc --no-create-db --no-create-info --skip-triggers mydb > db-data.sql
# or
mysqldump --complete-insert --extended-insert --hex-blob --tz-utc --no-create-db --no-create-info --single-transaction --lock-tables=false --skip-triggers --set-gtid-purged=OFF mydb > mydb-$(date "+%Y%m%d").sql
```

## Create new MySQL Super User
```sql
GRANT ALL PRIVILEGES ON *.* TO 'USER'@'HOST' IDENTIFIED BY 'PASSWORD'
WITH GRANT OPTION
```

## Disable mysql user ident
```sh
mysql -e "update user set plugin='mysql_native_password' where user='root'" mysql
mysql -e "flush privileges;"
```

## Configuration Parameters

```dosini
[mysqld]
# Allow maximum recursion for functions to 1 level
max_sp_recursion_depth=1
innodb_support_xa=0
innodb_lock_wait_timeout=2
default-storage-engine=innodb
skip-character-set-client-handshake
collation-server=utf8_unicode_ci
character-set-server=utf8

# Set the init script for stupid clients
# init_connect='SET collation_connection = utf8_unicode_ci'
# init_connect='SET NAMES utf8'
# Set the Timezone to UTC so that we can keep track of foreign times

default-time-zone='+00:00'
innodb_file_per_table

# Activate scheduler if you have events
event_scheduler=ON

thread_stack            = 1M
thread_cache_size       = 32

query_cache_type        = 1
query_cache_limit       = 1024M
query_cache_size        = 2048M


join_buffer_size        = 32M
key_buffer              = 64M
max_allowed_packet      = 16M
table_cache             = 512


innodb_flush_log_at_trx_commit=0
innodb_locks_unsafe_for_binlog=1
innodb_flush_method=O_DIRECT
transaction-isolation=READ-COMMITTED
low_priority_updates=1
#innodb_log_file_size=1024M
#innodb_log_files_in_group=3
innodb_buffer_pool_instances=8
innodb_file_per_table
innodb_buffer_pool_size = 2048M
innodb_additional_mem_pool_size=20M
#innodb_log_buffer_size=64M
innodb_use_sys_malloc=ON

# tune
innodb_checksums=0
innodb_doublewrite=0
innodb_support_xa=0
innodb_thread_concurrency = 8
innodb_flush_log_at_trx_commit = 2
innodb_max_dirty_pages_pct = 50
innodb_io_capacity = 2000
innodb_write_io_threads = 64
innodb_read_io_threads = 64
innodb_adaptive_flushing=1
innodb_purge_threads=1
table_open_cache             = 1024

#pkots
innodb_sync_spin_loops = 100
```



## Deadlock wrapper

```sql
DECLARE deadlock INT DEFAULT 0;
DECLARE attempts INT DEFAULT 0;

tfer_loop:WHILE (attempts<3) DO
BEGIN
	DECLARE deadlock_detected CONDITION FOR 1213;
	DECLARE EXIT HANDLER FOR deadlock_detected
	BEGIN
		SET deadlock=1;
	END;
	SET deadlock=0;
  /*
   * YOUR DEADLOCKED SQL HERE
   */
END;
IF deadlock=0 THEN  -- No deadlock occurred! bye, bye...
    	LEAVE tfer_loop;
ELSE
      CALL log_deadlock('tau_user_ranks');
 	SET attempts=attempts+1; -- deadlock occurred, try it again.
END IF;
END WHILE tfer_loop;

```


## Extract first IP match from MSG
```sql
SELECT * , preg_capture('/(([0-9]+)(?:\.[0-9]+){3})/', msg, 1, 1) FROM `archive`
WHERE preg_capture('/([0-9]+)(?:\.[0-9]+){3}/', msg, 1, 1) IS NOT NULL;
```

## Disable event scheduler
Execute the following into the mysql client...
```sql
SET GLOBAL event_scheduler = OFF;
SET @@global.event_scheduler = OFF;
SET GLOBAL event_scheduler = 0;
SET @@global.event_scheduler = 0;
```

## Using SET column operations
To remove an element, use this syntax:
```sql
update TABLE set COLUMN = COLUMN & ~NUM;
```

To add an element to an existing set, use this syntax:
```sql
update TABLE set COLUMN = COLUMN | NUM;
```

| SET Member | Decimal Value | Binary Value |
|------------|---------------|--------------|
| `a`  			 | 1 						 | 0001 				|
| `b`  			 | 2 						 | 0010 				|
| `c`  			 | 4 						 | 0100 				|
| `d`  			 | 8 						 | 1000 				|

## IP NETMASK TRICKS
Use `int unsigned` for your IP fields.

Insert IP to a table `INET_ATON('<IP_STRING>')`
```sql
insert into ip_table (ip) values (inet_aton('192.168.0.5'));
```

Select IP to string `INET_NTOA(unsigned int)`
```sql
select inet_ntoa(ip) as ip from ip_table;
```

Networks and netmasks are no different
```sql
select inet_ntoa(network) as network, inet_ntoa(netmask) as netmask from network_table;
```

List IPs from table that fall under a specific network (eg `192.168.0.0/255.255.255.128`)
```sql
select inet_ntoa(ip) as ip from ip_table where (ip & inet_aton('255.255.255.128')) = inet_aton('192.168.0.0');
```

Similarly look into a table of network/masks to find what networks an IP falls into.
```sql
select inet_ntoa(network) as network, inet_ntoa(netmask) as netmask from network_table where (inet_aton('192.168.0.1') & netmask) = network;
```

Get netmask for a given cidr (eg `/25`)
```sql
select inet_ntoa(power(2, 32) - power(2, (32 - 25))) as netmask;
																							^^^^
```

Get the network address
```sql
select inet_ntoa(inet_aton('192.168.0.5') & inet_aton('255.255.255.128')) as network;
```

Get the broadcast
```sql
select inet_ntoa(inet_aton('192.168.0.5') | (inet_aton('255.255.255.128') ^ (power(2, 32) - 1))) as broadcast;
```

Get the first and last usable IP of a network/mask
```sql
select inet_ntoa((inet_aton('192.168.0.5') & inet_aton('255.255.255.128')) + 1) as first_usable;
select inet_ntoa((inet_aton('192.168.0.5') | (inet_aton('255.255.255.128') ^ (power(2, 32) - 1))) - 1) as last_usable;
```

Get the number of usable IPs for a given netmask
```sql
select inet_aton('255.255.255.128') ^ (power(2, 32) - 2) as num_usable;
```

wildcard for a given netmask (255.255.255.128 here):
```sql
select inet_ntoa(inet_aton('255.255.255.128') ^ (power(2, 32) - 1)) as wildcard;
```


Get the cidr for a given netmask (eg `255.255.255.128`):
```sql
select 32 - bit_count(power(2, 32) - inet_aton('255.255.255.128') - 1) as cidr;
```
