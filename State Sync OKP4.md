## State Sync

```
peers="052e10ce23cce3249f61853e2ca6a63102b7bddb@5.161.97.198:26656"; \
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml

SNAP_RPC="http://5.161.97.198:26657"; \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.okpd/config/config.toml

sudo systemctl stop okp4d
okp4d tendermint unsafe-reset-all --home /root/.okp4d --keep-addr-book

sudo systemctl restart okp4d && journalctl -u okp4d -f -o cat
```

Optional - disable synchronization with State Sync after synchronization

```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.okp4d/config/config.toml
sudo systemctl restart okp4d && journalctl -u okp4d -f -o cat
```
