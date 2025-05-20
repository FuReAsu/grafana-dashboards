# Ceph RGW OPs Logs

Data source backend is loki from ceph.
Promtail from ceph automatically collects ceph logs.

## Set Up OPs Logs in Ceph

```bash
ceph config set client.rgw.<name> rgw_enable_ops_logs true
```

Now, ops logs will be in /var/log/ceph/\<fsid\>/
This will be collected by promtail and sent to loki
