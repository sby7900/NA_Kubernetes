### 错误解决
- 若是集群状态一直是 notready,用下面语句查看原因:
```
journalctl -f -u kubelet.service
```
  - 若原因是： 
```
cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
mkdir -p /etc/cni/net.d                    #创建目录给flannel做配置文件
vim /etc/cni/net.d/10-flannel.conf         #编写配置文件
{
 "name":"cbr0",
 "cniVersion":"0.3.1",
 "type":"flannel",
 "deledate":{
    "hairpinMode":true,
    "isDefaultGateway":true
  }

}
```

- flannel插件问题解决:
```
mkdir -p /etc/cni/net.d/
cat <<EOF> /etc/cni/net.d/11-flannel.conf
{"name":"cbr0","type":"flannel","delegate": {"isDefaultGateway": true}}
EOF
mkdir /usr/share/oci-umount/oci-umount.d -p
mkdimkdir -p /etc/cni/net.d/
cat <<EOF> /etc/cni/net.d/11-flannel.conf
{"name":"cbr0","type":"flannel","delegate": {"isDefaultGateway": true}}
EOF
mkdir /usr/share/oci-umount/oci-umount.d -p
mkdir /run/flannel/
cat <<EOF> /run/flannel/subnet.env
FLANNEL_NETWORK=172.100.0.0/16
FLANNEL_SUBNET=172.100.1.0/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOFr /run/flannel/
cat <<EOF>
```
