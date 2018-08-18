# cicd-3tier

Use Gitlab to create a full CI-CD pipeline for a standard 3 Tier, Core > Distribution > Access, topology.  The Core is composed of 2 IOS XE routers, Distribution and Access are Open NX-OS switches.  

## Prerequisites

This lab requires you to have the [gitlab stack](../gitlab) running on your devbox


## Network topology

The network topology for both test and prod is provided by virl as defined in [./virl/test/topology.virl](./virl/test/topology.virl) and [./virl/prod/topology.virl](./virl/prod/topology.virl)

![network](./static/topology.png)

## Pipeline Technology Stack

![pipeline](./static/pipeline.png "CICD Pipeline")

## setup

This following script should cover all the initial installation

```
./setup.sh
```


## Getting started

You should be able to see your infrastructure as code repository at http://10.10.20.20/developer/cicd-3tier


## Known Issues

### VIRL node boot error

Occasionally, a node may not boot up correctly and you will see an error similar to
this

```
sync-result {
    device test-dist2
    result false
    info Failed to connect to device test-dist2: connection refused: No route to host (Host unreachable)
}
```

To fix this you can use virlutils to stop/start the node e.g

```
cd virl/test
virl stop test-dist2
virl start test-dist2
```

### Initial Configuration Pushes from GitLab may fail

Occasionally the intial configuration pushes from GitLab to the networks may start before the nodes are fully ready, and one or more may timeout.  This can result in the initial configuration and connectivity in the topology from not coming up automatically after running `setup`.  

The most common operational issue is the VPC domain not coming up correctly between the dist nodes.  Troubleshoot using CLI, but these steps tend to work.  (Maybe needed on test, prod, or both)

1. SSH to both `dist1` and `dist2` and shutdown the VPC peerlink and VPC to `access1`

    ```
    conf t
    int po1
    shut
    int po11
    shut
    ```

1. Wait about a minute for everything to settle, and then reenable the peer-link on both switches.  

    ```
    int po1
    no shut
    ```

1. Once `show vpc` shows adjaceny healthy on both dist switches, bring up the link to access1

    ```
    int po11
    no shut
    ```

1. If VPC is not healthy on the distribution layer, the pyATS test will fail many of the tests as they are written to expect a healthy network.  

## Verification / Troubleshooting

If all goes well, you should see output similar to This

```
[developer@devbox cicd-3tier]$./setup.sh
Launching VIRL simulations (prod+test) ...
Creating default environment from topology.virl
Waiting 10 minutes for nodes to come online....
  [####################################]  100%             
Launching NSO ...
Importing Test network to NSO ..
Updating NSO....
Successfully added VIRL devices to NSO
Importing Prod network to NSO
Updating NSO....
Successfully added VIRL devices to NSO
Performing initial sync of devices...
sync-result {
    device access1
    result true
}
sync-result {
    device core1
    result true
}
sync-result {
    device core2
    result true
}
sync-result {
    device dist1
    result true
}
sync-result {
    device dist2
    result true
}
sync-result {
    device test-access1
    result true
}
sync-result {
    device test-core1
    result true
}
sync-result {
    device test-core2
    result true
}
sync-result {
    device test-dist1
    result true
}
sync-result {
    device test-dist2
    result true
}
Creating Repo on Gitlab
Configure Git
Initalizing Local Repository
Initialized empty Git repository in /home/developer/sbx_multi_ios/cicd-3tier/.git/
Switched to a new branch 'test'
[test (root-commit) ba69843] Initial commit
 24 files changed, 3540 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .gitlab-ci.yml
 create mode 100644 README.md
 create mode 100755 cleanup.sh
 create mode 100644 group_vars/all.yaml
 create mode 100644 host_vars/access1.yaml
 create mode 100644 host_vars/core1.yaml
 create mode 100644 host_vars/core2.yaml
 create mode 100644 host_vars/dist1.yaml
 create mode 100644 host_vars/dist2.yaml
 create mode 100644 inventory/prod.yaml
 create mode 100644 inventory/test.yaml
 create mode 100755 setup.sh
 create mode 100644 site.yaml
 create mode 100644 static/pipeline.png
 create mode 100644 tests/prod.robot
 create mode 100644 tests/prod_testbed.yml
 create mode 100644 tests/reachability/__init__.py
 create mode 100644 tests/reachability/pyats_job.py
 create mode 100644 tests/reachability/pyats_loopback_reachability.py
 create mode 100644 tests/test.robot
 create mode 100644 tests/test_testbed.yml
 create mode 100644 virl/prod/topology.virl
 create mode 100644 virl/test/topology.virl
Pushing Branches
Counting objects: 35, done.
Delta compression using up to 5 threads.
Compressing objects: 100% (30/30), done.
Writing objects: 100% (35/35), 356.63 KiB | 0 bytes/s, done.
Total 35 (delta 7), reused 0 (delta 0)
To http://developer:C1sco12345@10.10.20.20/developer/cicd-3tier.git
 * [new branch]      test -> test
Branch test set up to track remote branch test from origin.
Switched to a new branch 'production'
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: To create a merge request for production, visit:
remote:   http://gitlab/developer/cicd-3tier/merge_requests/new?merge_request%5Bsource_branch%5D=production
remote:
To http://developer:C1sco12345@10.10.20.20/developer/cicd-3tier.git
 * [new branch]      production -> production
Branch production set up to track remote branch production from origin.
Switched to branch 'test'
Test Network Summary
Running Simulations
╒═════════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation          │ Status   │ Launched                   │ Expires   │
╞═════════════════════╪══════════╪════════════════════════════╪═══════════╡
│ test_default_ldTwiU │ ACTIVE   │ 2018-08-15T04:18:26.886231 │           │
╘═════════════════════╧══════════╧════════════════════════════╧═══════════╛
Here is a list of all the running nodes
╒══════════════╤═════════════╤═════════╤═════════════╤════════════╤══════════════════════╤════════════════════╕
│ Node         │ Type        │ State   │ Reachable   │ Protocol   │ Management Address   │ External Address   │
╞══════════════╪═════════════╪═════════╪═════════════╪════════════╪══════════════════════╪════════════════════╡
│ test-dist1   │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.213        │ N/A                │
├──────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ test-access1 │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.215        │ N/A                │
├──────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ test-dist2   │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.214        │ N/A                │
├──────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ test-core2   │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.212        │ N/A                │
├──────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ test-core1   │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.211        │ N/A                │
╘══════════════╧═════════════╧═════════╧═════════════╧════════════╧══════════════════════╧════════════════════╛
Production Network Summary
Running Simulations
╒═════════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation          │ Status   │ Launched                   │ Expires   │
╞═════════════════════╪══════════╪════════════════════════════╪═══════════╡
│ prod_default_ZufsKU │ ACTIVE   │ 2018-08-15T04:18:27.136267 │           │
╘═════════════════════╧══════════╧════════════════════════════╧═══════════╛
Here is a list of all the running nodes
╒═════════╤═════════════╤═════════╤═════════════╤════════════╤══════════════════════╤════════════════════╕
│ Node    │ Type        │ State   │ Reachable   │ Protocol   │ Management Address   │ External Address   │
╞═════════╪═════════════╪═════════╪═════════════╪════════════╪══════════════════════╪════════════════════╡
│ core1   │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.221        │ N/A                │
├─────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ access1 │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.225        │ N/A                │
├─────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dist1   │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.223        │ N/A                │
├─────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ core2   │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.222        │ N/A                │
├─────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dist2   │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.224        │ N/A                │
╘═════════╧═════════════╧═════════╧═════════════╧════════════╧══════════════════════╧═══════════════════╛
```