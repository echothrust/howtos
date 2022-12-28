# Restore a RAC from scratch 
  * First reformat the vote and crs disks <code sh>
$ORA_CRS_HOME/install/rootdelete.sh
$ORA_CRS_HOME/install/rootdeinstall.sh
$ORA_CRS_HOME/root.sh (on all nodes)
oifcfg setif -global bond0/255.255.255.0:public
oifcfg setif -global bond1/255.255.255.0:cluster_interconnect
</code>

  * On all nodes <code sh>
/etc/oratab +ASM entry
rm -rf admin/+ASM/* 
rm -rf product/10.2.0/asm_1/dbs/* 
</code>
  * On first node<code sh>
su - oracle
. oraenv
+ASM1
