---
---

# How to check Oracle corruption

(Tip given from MOS - myoracle support)


Please check with RMAN if there are datablock corruptions :

```
% rman target /
RMAN> run {
allocate channel d1 type disk;
allocate channel d2 type disk;
backup check logical validate filesperset 10 database;
}
```

RMAN is populating the view V$DATABASE_BLOCK_CORRUPTION with the above commands, if corrupted blocks are found

Use `sqlplus` and connect as `sysdba` user `sqlplus "/ as sysdba"` and run the following queries
```sql
SQL> select * from v$database_block_corruption ;

SQL> SELECT e.owner, e.segment_type, e.segment_name, e.partition_name, c.file#
, greatest(e.block_id, c.block#) corr_start_block#
, least(e.block_id+e.blocks-1, c.block#+c.blocks-1) corr_end_block#
, least(e.block_id+e.blocks-1, c.block#+c.blocks-1)
- greatest(e.block_id, c.block#) + 1 blocks_corrupted
, null description
FROM dba_extents e, v$database_block_corruption c
WHERE e.file_id = c.file#
AND e.block_id <= c.block# + c.blocks - 1
AND e.block_id + e.blocks - 1 >= c.block#
UNION
SELECT s.owner, s.segment_type, s.segment_name, s.partition_name, c.file#
, header_block corr_start_block#
, header_block corr_end_block#
, 1 blocks_corrupted
, 'Segment Header' description
FROM dba_segments s, v$database_block_corruption c
WHERE s.header_file = c.file#
AND s.header_block between c.block# and c.block# + c.blocks - 1
UNION
SELECT null owner, null segment_type, null segment_name, null partition_name, c.file#
, greatest(f.block_id, c.block#) corr_start_block#
, least(f.block_id+f.blocks-1, c.block#+c.blocks-1) corr_end_block#
, least(f.block_id+f.blocks-1, c.block#+c.blocks-1)
- greatest(f.block_id, c.block#) + 1 blocks_corrupted
, 'Free Block' description
FROM dba_free_space f, v$database_block_corruption c
WHERE f.file_id = c.file#
AND f.block_id <= c.block# + c.blocks - 1
AND f.block_id + f.blocks - 1 >= c.block#
order by file#, corr_start_block#;
```