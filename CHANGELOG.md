# Changelog

## 1.0.2 (2026-11-10)

Globus: the "ARCTIC Transfer" flows no longer work. The reference now describes the File Manager's two-pane view, the supported way to move files between your computer and ARCTIC, and names the collection for each destination (ARCTIC-SCRATCH, ARCTIC-HOME-NEW, S3-ARCTIC).

Removed links to three wiki pages that aren't live yet: `/en/data-transfer/scp-rsync`, `/en/backup/backup-policy` and `/en/software/load-modules`. The facts stay; add the links back when the pages are published.

## 1.0.1 (2026-10-28)

qGPU partition memory corrected to about 8,500 GB (it was computed in decimal units, unlike the other partitions).

## 1.0.0 (2026-10-28)

First public version. Built from the ARCTIC wiki as updated in October 2026 and from test jobs run on the cluster that month (CUDA compiler locations, the `pytorch/2.7.0` V100 limitation, the R torch install, `$SCRATCH` creation, the conda shell hook).
