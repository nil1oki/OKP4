## Setting `app.toml`

```
sed -i.bak -e "s/^pruning *=.*/pruning = \""custom"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \""100"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^pruning-keep-every *=.*/pruning-keep-every = \""1000"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \""10"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \""1000"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^snapshot-keep-recent *=.*/snapshot-keep-recent = \""2"\"/" $HOME/.okp4d/config/app.toml
```

## Setting `config.toml`

```
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:26657\"%" $HOME/.okp4d/config/config.toml
sed -i.bak -E "s|^(pex[[:space:]]+=[[:space:]]+).*$|\1true|" $HOME/.okp4d/config/config.toml
```

## Restart

```
systemctl restart okp4d && journalctl -u okp4d -f -o cat
```

## You shoud know 4 things:

`Your_server_IP` - find out ip of your server with State Sync--> `wget -qO- eth0.me`

`Your_rpc_port` (default is 26657)

`Your_p2p_port` (default is 26656)

`Your_node_id`: find out node_id of the RPC server--> `curl localhost:26657/status | jq '.result.node_info.id'` 

`Your_interval` - it is value of `snapshot-interval`/`pruning-keep-every`

```
echo "$(okp4d tendermint show-node-id)@$(curl ifconfig.me):26656"
echo "$(okp4d tendermint show-node-id)@$(curl ifconfig.me):26657"
```

### Now just insert the values:
Adding public RPC node to `persistance_peer` in `config.toml`.
Here you need `<Your_node_id>`, `<Your_server_IP>`, `<Your_p2p_port>`.

```
peers="<Your_node_id>@<Your_server_IP>:<Your_p2p_port>"; \
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml
```

Adding variables.

```
SNAP_RPC="http://<Your_server_IP>:<Your_rpc_port>"; \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```

Entering all the datat to `config.toml`

```
sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.okpd/config/config.toml
```

Deleting all downloaded date by unsafe-reset-all and resterting `axelard.service`

```
sudo systemctl stop okp4d && \
icad tendermint unsafe-reset-all --home $HOME/.okp4 && \
sudo systemctl restart okp4d && journalctl -u okp4d -f -o cat
```

```
peers="052e10ce23cce3249f61853e2ca6a63102b7bddb@5.161.97.198:26656"; \
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml
```

```
SNAP_RPC="http://5.161.97.198:26657"; \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
