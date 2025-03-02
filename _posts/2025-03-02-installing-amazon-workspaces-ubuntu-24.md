---
layout: post
title: Installing Amazon Workspaces on Ubuntu 24.04 (noble)
date: '2025-03-02T11:37:00.000+00:00'
author: Craig Watcham
tags:
- Amazon Workspaces
- gpg
modified_time: '2025-03-02T11:37:00.000+00:00'
---

There is currently no Ubuntu 24.04 client for Amazon Workspaces, fortunately the 22.04 client can be installed via apt with only a minor tweak.

Following the Linux install instructions [here](https://clients.amazonworkspaces.com/linux-install) will lead to a signature error when performing 'apt update':
```
Failed to fetch https://d3nt0h4h6pmmc4.cloudfront.net/ubuntu/dists/jammy/InRelease  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 04B0588859EF5026
```
\
The reason for this is the key from Amazon is not in the binary format apt is expecting. Changing the key retrieval command from
```
wget -q -O - https://workspaces-client-linux-public-key.s3-us-west-2.amazonaws.com/ADB332E7.asc | sudo tee /etc/apt/trusted.gpg.d/amazon-workspaces-clients.gpg > /dev/null
```
To:
```
wget -q -O - https://workspaces-client-linux-public-key.s3-us-west-2.amazonaws.com/ADB332E7.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/amazon-workspaces-clients.gpg
```
Writes the key in the expected format and allows apt to verify the source signatures and install the package correctly. Note that the 22.04 client only supports DCV, you will need to modify your Workspace to switch to DCV if it is provisioned as PCoIP.