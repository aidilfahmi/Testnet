# $${\color{lightgreen}Disk-Usage \space Optimization}$$

Customize the configuration settings to lower the disk requirements for your validator node {synopsis}

Blockchain database tends to grow over time, depending e.g. on block
speed and transaction amount.

There are few configurations that can be done to reduce the required
disk usage quite significantly. Some of these changes take full effect
only when you do the configuration and start syncing from start with
them in use.

## ${\color{orange}Indexing}$
If you do not need to query transactions from the specific node, you can
disable indexing. <br>
Replace `node_dir` with current node directory.
```toml
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.node_dir/config/config.toml
```

If you do this on already synced node, the collected index is not purged
automatically, you need to delete it manually. The index is located
under the database directory with name `data/tx_index.db/`.

## ${\color{orange}State-sync \space Snapshots}$
Replace `node_dir` with current node directory.

```toml
sed -i -e 's|^snapshot-interval *=.*|snapshot-interval = "0"|' $HOME/.node_dir/config/app.toml
```

Note that if state-sync was enabled on the network and working properly,
it would allow one to sync a new node in few minutes. But this node
would not have the history.

## ${\color{orange}Pruning}$
By default every 500th state, and the last 100 states are kept. This
consumes a lot of disk space on long run, and can be optimized with
following custom configuration:<br>
Replace `node_dir` with current node directory.

```toml
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="19"

sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.banksy/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.node_dir/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.node_dir/config/app.toml
```
Configuring `pruning-keep-recent = "0"` might sound tempting, but this
will risk database corruption if the `nibid` is killed for any reason.
Thus, it is recommended to keep the few latest states.

## ${\color{orange}Logging}$

By default the logging level is set to `info`, and this produces a lot of
logs. This log level might be good when starting up to see that the
node starts syncing properly. However, after you see the syncing is
going smoothly, you can lower the log level to `warn` (or `error`). <br>
Replace `node_dir` with current node directory.
```toml
sed -i -e 's|^log_level *=.*|log_level = "warn"|' $HOME/.node_dir/config/config.toml
```

Also ensure your log rotation is configured properly.

