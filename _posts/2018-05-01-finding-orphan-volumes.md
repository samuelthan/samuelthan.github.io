---
title: 'Finding Orphaned AWS snapshots or volumes'
date: 2018-05-01 00:00:00
featured_image: '/images/post/AWS_EBS.jpeg'
excerpt: There was no straightforward way to figure out which AWS snapshot belongs to which instance...
---

Sometimes there is no straightforward way to find out which AWS snapshot belongs to which instance, especially if it’s not managed properly for a long time. Often, and instance may have been deleted but the snapshots got forgotten. Especially the ones that are created manually.

**Goal:** Create an excel/csv list of AWS snapshots and linked it to it’s AWS instance and Volume of origin

I’ve taken the liberty of putting together a few PowerShell to make this happen, some of this are modification from other contributors

* ListProdEC2.ps1
* ListProdEC2Snapshots.ps1
* ListProdEC2Volumes.ps1
* ListProdEC2InstancesDetails.ps1

[Available reference at Github](https://github.com/samuelthan/AWS-Powershell)
