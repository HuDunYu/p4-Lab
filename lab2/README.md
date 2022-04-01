原本运行 mycontroller.py 只能让 h1 和 h2 ping 通，是因为在 main 函数中，只增加了 s1，s2  两个交换机的信息，我们还需要自己添加 s3 的信息。

添加完 s3 的信息后，我们还需要修改一下 writeTunnelRules 函数，在写 Tunnel Transit Rule 时，

原本代码是：

```python
table_entry = p4info_helper.buildTableEntry(
        table_name="MyIngress.myTunnel_exact",
        match_fields={
            "hdr.myTunnel.dst_id": tunnel_id
        },
        action_name="MyIngress.myTunnel_forward",
        action_params={
            "port": SWITCH_TO_SWITCH_PORT
        })
    ingress_sw.WriteTableEntry(table_entry)
    print("Installed transit tunnel rule on %s" % ingress_sw.name)
```

在 action_params 那个地方是 "port": SWITCH_TO_SWITCH_PORT，其中 SWITCH_TO_SWITCH_PORT 这个值是2，这导致了交换机之间传递信息时只能通过端口号为 2 的端口。因此我们要根据拓扑图，自己定义交换机到各个交换机的端口常量，如下定义常量：

```python
S1_TO_S2 = 2
S2_TO_S1 = 2
S1_TO_S3 = 3
S3_TO_S1 = 2
S3_TO_S2 = 3
S2_TO_S3 = 3
```

并且在 writeTunnelRules 新增一个参数来放端口：

```python
def writeTunnelRules(p4info_helper, ingress_sw, egress_sw, tunnel_id,
                     dst_eth_addr, dst_ip_addr, port):
```

并如下修改写 Tunnel Transit Rule 的代码：将 SWITCH_TO_SWITCH_PORT 改为 port

```
table_entry = p4info_helper.buildTableEntry(
        table_name="MyIngress.myTunnel_exact",
        match_fields={
            "hdr.myTunnel.dst_id": tunnel_id
        },
        action_name="MyIngress.myTunnel_forward",
        action_params={
            "port": port
        })
    ingress_sw.WriteTableEntry(table_entry)
    print("Installed transit tunnel rule on %s" % ingress_sw.name)
```

这样子我们在 main 函数中为各个交换机 writeTunnelRules 时，就可以这样写：

```python
writeTunnelRules(p4info_helper, ingress_sw=s1, egress_sw=s2, tunnel_id=100,
                         dst_eth_addr="08:00:00:00:02:22", dst_ip_addr="10.0.2.2", 								port=S1_TO_S2)
```

这是在写交换机到交换机的信息时，就体现了端口信息。