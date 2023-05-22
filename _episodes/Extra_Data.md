---
title: "Moving Data"
teaching: 15
exercises: 15
questions:
- "Where/how can I store data?"
- "How can I move data from A to B?"
objectives:
- "Understand the range of options available for data moving and storage"
keypoints:
- "Use a storage solution that is fit for purpose"
- "Moving data can be a significant part of your workflow"
---

## Where to store data
- Local machine
- Online repo
  - dropbox
  - google drive, one drive
  - zenodo
- HPC
  - Hot storage (group/scratch/project)
  - warm storage (buckets)
  - cold storage (tape)

few large vs many small files
- archiving

## How to move data
- drag n drop / web site
- wget
- scp
- `tar cvf - /data | ssh otherhost tar xvf -`
- rsync
- minio client (mc)
- pigeon