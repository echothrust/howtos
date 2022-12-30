---
---

# Docker Tips and Tricks
## inspect formatted
```sh
{% raw %}
docker inspect --format='IP: {{ .NetworkSettings.Networks.ctf.IPAddress }} MAC: {{ .NetworkSettings.Networks.ctf.MacAdess }}' host
docker inspect --format='{{range $p, $conf := .Config.ExposedPorts}} {{$p}} {{end}}' host
{% endraw %}
```

## generate ansible yml config from running container
```sh
{% raw %}
echo "---";docker inspect --format=' {{ .Config.Hostname }}
ansible_host: {{ .NetworkSettings.Networks.ctf.IPAddress }}
hostname: {{ .Config.Hostname }}
fqdn: {{ .Config.Hostname }}
mac: {{ .NetworkSettings.Networks.ctf.MacAddress }}
OS: docker
ETSCTF_FINDINGS:
{{range $p, $conf := .Config.ExposedPorts}}  - { name: "Open Port", points: 100, stock: -1, protocol: "tcp", port: {{$p}}  }
{{end}}
  - { name: "Ping host", points: 100, stock: -1, protocol: "icmp", port: 0 }
' host
{% endraw %}
```

## commit all running containers with ETSCTF tag
```sh
docker ps -a|grep -v NAME|awk '{print "docker commit "$1" "$NF":ETSCTF"}'|source /dev/stdin
```
