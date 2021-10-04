---
title: Using Powershell to find and clean up old files
header:
  show_overlay_excerpt: false
  overlay_image: "/content/images/2016/10/file-system.jpg"
---
I was previously responsible for supporting a number of file servers that have a large number of very small files archived to them. These files are occasionally referenced from their source systems but I had long suspected that there were a number of orphaned/redundant files on these servers (e.g where we had decommissioned the source system and not cleaned up the archive) and wanted Powershell to help verify this. 

I had three wishes:

1. To identify which folders on the archive servers were orphans. Our source systems reference these via a DFS path, so its a safe bet that if there are no DFS paths with folder targets pointing to the files on the archive location, they are probably orphans.

2. To know when a file was last written/modified in these orphan locations (as a second check that the parent folders were not still being written to directly, e.g outside of a DFS path).

3. To know how large the orphaned folders were so that I could brag about how much storage saving I had made.

![](/content/images/2016/10/Aladdin-Lamp.jpg)

