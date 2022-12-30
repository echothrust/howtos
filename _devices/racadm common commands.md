---
---

# racadm common commands

Some racadm commands to help in configuring our DELL servers taken from a gist that I dont recall at the moment...

### disable host -> BMC config update
```sh
racadm config -g cfgRacTuning -o cfgRacTuneLocalConfigDisable 1
```

### disable dell proprietary admin protocol
(this won't disable sshd or the internal racadm tool on the ssh shell)
```sh
racadm config -g cfgRacTuning -o cfgRacTuneRemoteRacadmEnable 0
```

### disable http/https server
```sh
racadm config -g cfgRacTuning -o cfgRacTuneWebServerEnable 0
```

### disable IPMI Serial Over LAN
```sh
racadm config -g cfgIpmiSol -o cfgIpmiSolEnable 0
racadm config -g cfgIpmiPef -o cfgIpmiPefEnable 0 -i 1
```

### disable ipmi trap
```sh
racadm config -g cfgIpmiPet -o cfgIpmiPetAlertEnable 0 -i 1
```

### disable snmpd
```sh
racadm config -g cfgOobSnmp -o cfgOobSnmpAgentEnable 0
```

### disable telnetd
```sh
racadm config -g cfgSerial  -o cfgSerialTelnetEnable 0
```

### enable sshd
```sh
racadm config -g cfgSerial -o cfgSerialSshEnable 1
```

### disable IPMI over LAN (globally, hopefully disables the ipmi server)
```sh
racadm config -g cfgIpmiLan -o cfgIpmiLanAlertEnable 0
racadm config -g cfgIpmiLan -o cfgIpmiLanEnable 0
```

### reset the drac to make sure all settings apply
```sh
racadm racreset
```
