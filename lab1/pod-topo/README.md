在原来的 topology.json 文件中删掉为每台主机定义的arp转发选项。

在原来的 sx-runtime.json文件中为每台交换机的 action_params 新增一个 srcAddr 参数，该参数的值即为该交换机的 mac 地址。