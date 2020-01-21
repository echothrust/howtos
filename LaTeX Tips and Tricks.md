# LaTeX Tips & Tricks

## Add new font
```sh
kpsewhich --var-value TEXMFLOCAL
mktexlsr
updmap-sys --enable Map gfsneohellenic.map
```

The output of `updmap` is similar to the following.
```
updmap: This is updmap, version $Id: updmap 14402 2009-07-23 17:09:15Z karl $
updmap: using transcript file `/usr/local/share/texmf-var/web2c/updmap.log'
updmap: initial config file is `/usr/local/share/texmf-config/web2c/updmap.cfg'
updmap: configuration (updmap.cfg) unchanged. Map files will not be recreated.
```
