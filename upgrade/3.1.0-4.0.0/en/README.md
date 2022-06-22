# Upgrade instructions from version 3.1.0 to version 4.0.0 Commercio Network Chain 

## Prerequisites

1. Have a working node with software version v3.1.0
2. Have all the tools on the server to compile the application as mentioned in the first paragraph at https://docs.commercio.network/nodes/full-node-installation.html

If you want to make a backup and you have a kms, when you stop the service to make the backup, save the status file in the kms itself

## Raccomandations

1. To speed up the upgrade and make it easy use cosmovisor tools


**Please note**: this upgrade is **mandatory**. The nodes that will not upgrade will become incompatible with the chain.

**Please note**: if you install a new node from genesis use version 3.0.0 and try to upgrade after height 3318000 to version 3.1.0. After the halt height of this upgrade the chain  will be stopped automatecally and the new version 4.0.0 will be required. If you want use this version directly downloading the chain dump from https://quicksync.commercio.network after 2022/06/(28)

## Upgrade procedure

Download the repo from GitHub **if you not already done**. If you have already the local copy of repository don't try clone it.

```bash
git clone https://github.com/commercionetwork/commercionetwork.git
```

Go to the repo folder and checkout to the v4.0.0 tag

```bash
cd commercionetwork
git pull
git checkout v4.0.0
```

Build the Application

```bash
make build
```

Check that the application is the right version

```bash
./build/commercionetworkd version --long
```

The result should be

```
name: commercionetwork
server_name: commercionetword
version: 4.0.0
commit: -------
build_tags: netgo,ledger
go: go version go1.18.1 linux/amd64
```


### Cosmovisor installation

**Warning**: you need to setup cosmovisor env, in particular way `$DAEMON_HOME` variable.
From `commercionetwork` repository folder run commands below

```bash
mkdir -p $DAEMON_HOME/cosmovisor/upgrades/v4.0.0/bin
cp ./build/commercionetworkd $DAEMON_HOME/cosmovisor/upgrades/v4.0.0/bin/.
```

Wait sitting in your armchair watching your favorite TV series for the chain upgrade: cosmovisor should do all the dirty work for you.

**WARNING**: If you use cosmovisor version 1.0.0 or earlier you need to setup backup strategies. If you don't setup `UNSAFE_SKIP_BACKUP` variable a backup of your `data` folder will be performed before the upgrade. If the `data` folder occupied 60Gb you need an equal or greater amount of free space on your disk to perform the backup. Read [here](./setup_cosmovisor.md) how to setup your cosmovisor.


### Generic installation


**Warning**: the path where the executable is installed depends on your environment. In the following it is indicated with $GOPATH.

Few hours before upgrade setup your `commercionetworkd` service changing the line

```
Restart=always
```
in
```
#Restart=always
Restart=no
```

Your service file should be in `/etc/systemd/system/commercionetworkd.service`

Reload and restart service
```bash
systemctl daemon-reload
systemctl restart commercionetworkd.service
```

Now wait sitting in your office chair, rotating your thumbs, verifying from your the logs when your node crashes.

```bash
journalctl -u commercionetworkd.service -f
```

When the node will halted you need to chenge the `commercionetworkd` program and start the service.     
From `commercionetwork` repository folder run commands below

```bash
cp ./build/commercionetworkd $GOPATH/bin/.
```

If you want make a backup coping the `data` folder in a secure place.  

Remembar change back your service configuration

```
#Restart=always
Restart=no
```
in
```
Restart=always
```


Reload and restart the service

```bash
systemctl daemon-reload
systemctl start commercionetworkd.service
```

Check if the service is working

```bash
journalctl -u commercionetworkd.service -f
```

