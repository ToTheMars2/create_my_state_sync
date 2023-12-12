# create_my_state_sync


* RPC
* Get peer
* state sync

```
bin="hid-noded"
config=".hid-node"
service="hypersingd"
```

## Create RPC
```
sed -i '91 s/127.0.0.1/0.0.0.0/' ~/$config/config/config.toml
rpc_port=$(sed -n "91 s/^.*://p" ~/$config/config/config.toml | sed -n 's/"$//p')
ufw allow $rpc_port 
service $service restart

```
<b>Get `ip:port` for rpc check</b>
```
rpc_port=$(sed -n "91 s/^.*://p" ~/$config/config/config.toml | sed -n 's/"$//p')
echo $(curl ifconfig.me):$rpc_port
```
```
config='.hid-node'
```
```
rpc_port=$(sed -n "91 s/^.*://p" ~/$config/config/config.toml | sed -n 's/"$//p')
echo http://localhost:$rpc_port
```

## Create peer
```
rpc_port=$(sed -n "202 s/^.*://p" ~/$config/config/config.toml | sed -n 's/"$//p')
echo $($bin tendermint show-node-id)@$(curl ipinfo.io/ip):$rpc_port
```

## Start RPC on your server

```
SNAP_RPC="http://65.109.108.152:24557"
bin="hid-noded"
config=".hid-node"
service="hypersingd"
peers="20e40949206d9d991274bfa388af4f77b7da0de1@65.109.108.152:24556"
```

```
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/$config/config/config.toml
```
```
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/$config/config/config.toml

```

```
sudo systemctl stop $service && $bin tendermint unsafe-reset-all --home $HOME/$config --keep-addr-book
sudo systemctl restart $service
journalctl -fu $service -o cat

```
