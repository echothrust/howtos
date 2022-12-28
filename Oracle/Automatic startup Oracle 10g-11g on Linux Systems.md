# How to auto startup Oracle 10g/11g on Linux Systems

If you have installed 11g using the Grid Infrastructure (either for a single node or a RAC environment), then you do not need to worry for autostartup, as Grid Infrastructure installs a service called Oracle Restart, which does exactly that, the automatic startup of Oracle processes whenever they fail or the system reboots.

If you have installed 10g or 11g (Without using the Grid Infrastructure option then you need to):
http://www.oracle-base.com/articles/linux/AutomatingDatabaseStartupAndShutdownOnLinux.php