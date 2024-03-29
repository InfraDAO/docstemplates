# 🔗 CIP Linea Baremetal Guide

_Note: The Linea archive node was 524 GB on February 27.2024_

## System Requirements

| CPU                      | OS           | RAM       | DISK |
| ------------------------ | ------------ | --------- | ---- |
| A fast CPU with 4+ cores | Ubuntu 22.04 | 16GB+ RAM | 1TB+ |

## Pre-Requisites

<pre class="language-bash"><code class="lang-bash">sudo apt update -y &#x26;&#x26; sudo apt upgrade -y &#x26;&#x26; sudo apt autoremove -y

<strong>sudo apt install -y build-essential bsdmainutils aria2 dtrx screen clang cmake curl httpie jq nano wget
</strong>
</code></pre>

### Install Go

```bash
wget https://golang.org/dl/go1.21.6.linux-amd64.tar.gz

sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz

echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc

source ~/.bashrc

#check Go version

go version
```

### Setup the Geth client to run Linea

```bash
git clone https://github.com/ethereum/go-ethereum.git

cd go-ethereum

#Reportedly most stable version to run Linea node 1.13.4-stable-3f907d6a

git checkout 3f907d6a

make geth

cd

#Create directory for database

mkdir linea-datadir

cd linea-datadir

#Download Genesis file

wget https://docs.linea.build/files/genesis.json

#Bootstrap the node:

cd /root/go-ethereum

./build/bin/geth --datadir /root/linea-datadir/ init /root/linea-datadir/genesis.json
```

### Create service to run Linea Node

<pre class="language-bash"><code class="lang-bash"><strong>sudo echo "[Unit]
</strong>Description=Linea Node
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/go-ethereum
ExecStart=/root/go-ethereum/build/bin/geth \
--datadir /root/linea-datadir \
--networkid 59144 \
--rpc.allow-unprotected-txs \
--txpool.accountqueue 50000 \
--txpool.globalqueue 50000 \
--txpool.globalslots 50000 \
--txpool.pricelimit 1000000 \
--txpool.pricebump 1 \
--txpool.nolocals \
--http --http.addr '127.0.0.1' --http.port 8545 --http.corsdomain '*' --http.api 'web3,eth,txpool,net' --http.vhosts='*' \
--ws --ws.addr '127.0.0.1' --ws.port 8546 --ws.origins '*' --ws.api 'web3,eth,txpool,net' \
--bootnodes "enode://ca2f06aa93728e2883ff02b0c2076329e475fe667a48035b4f77711ea41a73cf6cb2ff232804c49538ad77794185d83295b57ddd2be79eefc50a9dd5c48bbb2e@3.23.106.165:30303,enode://eef91d714494a1ceb6e06e5ce96fe5d7d25d3701b2d2e68c042b33d5fa0e4bf134116e06947b3f40b0f22db08f104504dd2e5c790d8bcbb6bfb1b7f4f85313ec@3.133.179.213:30303,enode://cfd472842582c422c7c98b0f2d04c6bf21d1afb2c767f72b032f7ea89c03a7abdaf4855b7cb2dc9ae7509836064ba8d817572cf7421ba106ac87857836fa1d1b@3.145.12.13:30303" \
--syncmode full \
--metrics \
--verbosity 3 \
--gcmode archive
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/linea-node.service
</code></pre>

### Run Linea

```bash
sudo systemctl daemon-reload
sudo systemctl start linea-node
sudo systemctl enable linea-node
```

## Monitor logs

```bash
sudo journalctl -fu linea-node
```



## References

{% embed url="https://docs.linea.build/build-on-linea/run-a-node#step-3-1" %}

