---
---

# Elasticsearch Tips and Tricks
---
date: 04/02/2025
---


* Set user `elastic` password to `elastic`
```sh
echo -e "elastic\r\nelastic\r\n"|elasticsearch-reset-password -u elastic -b -i -s
```
