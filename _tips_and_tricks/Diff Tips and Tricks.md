---
---

# Diff Tips and Tricks

##How to create patches using diff
<note>The original patch has the //new// changes</note>
To create a patch for a single file, use the form:
<code>
diff -u original.c new.c > original.patch
</code>

To create a patch for an entire source tree, make a copy of the tree:
<code>
cp -R original new
</code>

Make any changes required in the directory new/. Then create a patch with the following command:
<code>
diff -rupN original/ new/ > original.patch
</code>