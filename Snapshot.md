Stop node

```
systemctl stop okp4d
```

Delete the .data directory and create an empty directory

```
rm -rf $HOME/.okp4d/data/
mkdir $HOME/.okp4d/data/
```

Download the archive

```
cd $HOME
wget http://5.161.97.198:8000/okp4data.tar.gz
```

Unpack the archive without the subfolder
```
tar -C $HOME/ -zxvf okp4data.tar.gz --strip-components 1
```

Delete archive

```
cd $HOME
rm okp4data.tar.gz
```

Restart node

```
systemctl restart okp4d && journalctl -u okp4d -f -o cat
```
