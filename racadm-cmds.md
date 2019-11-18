# disable host -> BMC config update
racadm config -g cfgRacTuning -o cfgRacTuneLocalConfigDisable 1
# disable dell proprietary admin protocol
# (this won't disable sshd or the internal racadm tool on the ssh shell)
racadm config -g cfgRacTuning -o cfgRacTuneRemoteRacadmEnable 0
# disable http/https server
racadm config -g cfgRacTuning -o cfgRacTuneWebServerEnable 0
# disable IPMI Serial Over LAN
racadm config -g cfgIpmiSol -o cfgIpmiSolEnable 0
racadm config -g cfgIpmiPef -o cfgIpmiPefEnable 0 -i 1
# disable ipmi trap
racadm config -g cfgIpmiPet -o cfgIpmiPetAlertEnable 0 -i 1
# disable snmpd
racadm config -g cfgOobSnmp -o cfgOobSnmpAgentEnable 0
# disable telnetd
racadm config -g cfgSerial  -o cfgSerialTelnetEnable 0
# enable sshd
racadm config -g cfgSerial -o cfgSerialSshEnable 1
# disable IPMI over LAN (globally, hopefully disables the ipmi server)
racadm config -g cfgIpmiLan -o cfgIpmiLanAlertEnable 0
racadm config -g cfgIpmiLan -o cfgIpmiLanEnable 0
# reset the drac to make sure all settings apply
racadm racreset
