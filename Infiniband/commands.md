[toc]

# 命令工具

## ibqueryerrors - 查看ib错误计数器

* 此命令显示所有超过指定阈值的端口计数器的信息
* 此外，此命令可以重置错误计数器

## ibdiagnet - 检查和诊断整个网络拓扑



## iblinkinfo - 查看ib连接信息

* 可查询整个ib网所有端口的链接信息

```
CA: m01 HCA-3:
      0x0000000000000123      2    1[  ] ==( 4X      25.78125 Gbps Active/  LinkUp)==>       1    1[  ] "m02 HCA-1" ( )
CA: m02 HCA-1:
      0x0002c90300212111      1    1[  ] ==( 4X      25.78125 Gbps Active/  LinkUp)==>       2    1[  ] "m01 HCA-3" ( )
```

- CA: m01 HCA-3 CA代表该设备是网卡, m01 HCA-3 是 m01 这台机器上的 HCA 卡 3号.
- 0x0000000000000123 是设备(网卡或交换机) 的GID, 用于全局标记设备, 全局唯一.
- 2 是 lid, 用于在子网内标记设备, 子网内唯一, 是 OpenSM 程序分配的.
- 1 是设备的端口号, 我们的网卡只有一个端口, 序号是 1.
- 4 X 25.78125 Gbps, 由于是 QSFP28 链路, 该规范可以把端口拆分成 4 个 SFP28 链路, SFP28 是 25Gbps, 我们的端口是 QSFP28, (Q代表Quad是4个xxx的意思), 所以链路是 4 X 25 Gbps, 总计 100Gbps. 你也许会问为啥小数点后面还有一堆? 这跟光传输网络有关, 想详细了解可以去查询 wikipedia.
- 刚才说到了可以拆分成 4 个 SFP28, 所以如果买一条 QSFP28 转 4 个 SFP28 的电缆, 的确可以一拖四, 交换机上也可以用这种电缆来实现更高的密度接入.
- Active/ LinkUp, Active 代表 IB 在协议层运行正常, LinkUp 代表物理层(电缆, 硬件设备)连接正常.
- ==> 1 1[ ] "m02 HCA-1" ( ) 表示当前设备连接的对端设备, 第一个是 lid, 第二个是端口号, 后面是机器名和设备号.

## ib_send_bw -- 测试ib吞吐

```
[root@m01 ~]# ib_send_bw -a -c UD -d mlx5_0 -i 1
 Max msg size in UD is MTU 4096
 Changing to this MTU
---------------------------------------------------------------------------------------
                    Send BW Test
 Dual-port       : OFF    Device         : mlx5_0
 Number of qps   : 1    Transport type : IB
 Connection type : UD   Using SRQ      : OFF
 RX depth        : 1000
 CQ Moderation   : 100
 Mtu             : 4096[B]
 Link type       : IB
 Max inline data : 0[B]
 rdma_cm QPs   : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x02 QPN 0x0090 PSN 0x57780b
 remote address: LID 0x01 QPN 0x008e PSN 0xec96be
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[MB/sec]    BW average[MB/sec]   MsgRate[Mpps]
 2          1000             0.00               6.53         3.424226
 4          1000             0.00               18.11        4.747918
 8          1000             0.00               31.74        4.160209
 16         1000             0.00               93.23        6.109701
 32         1000             0.00               177.60       5.819656
 64         1000             0.00               346.08       5.670225
 128        1000             0.00               705.65       5.780665
 256        1000             0.00               1481.36      6.067664
 512        1000             0.00               2956.48      6.054879
 1024       1000             0.00               5258.59      5.384796
 2048       1000             0.00               9157.67      4.688726
 4096       1000             0.00               9582.67      2.453164
---------------------------------------------------------------------------------------
[root@m01 ~]# 
```

可以看到在 4096bytes 情况下 BW average[MB/sec] 9582.67, 折合约 78.5Gbps. MsgRate[Mpps] 2.453164, 每秒传输2.45百万数据包. 增大包大小(也要提升MTU), 会得到更大带宽.

## ib_send_lat - **测试 IB 延时性能**

```
[root@m02 ~]# ib_send_lat -a -c UD -d mlx5_0 -i 1 192.168.123.1
 Max msg size in UD is MTU 4096
 Changing to this MTU
---------------------------------------------------------------------------------------
                    Send Latency Test
 Dual-port       : OFF    Device         : mlx5_0
 Number of qps   : 1    Transport type : IB
 Connection type : UD   Using SRQ      : OFF
 TX depth        : 1
 Mtu             : 4096[B]
 Link type       : IB
 Max inline data : 188[B]
 rdma_cm QPs   : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x01 QPN 0x0091 PSN 0xe2af32
 remote address: LID 0x02 QPN 0x0093 PSN 0x98b544
---------------------------------------------------------------------------------------
 #bytes #iterations    t_min[usec]    t_max[usec]  t_typical[usec]    t_avg[usec]    t_stdev[usec]   99% percentile[usec]   99.9% percentile[usec] 
 2       1000          0.79           5.30         0.81              0.82         0.12      0.84        5.30   
 4       1000          0.77           3.43         0.79              0.79         0.03      0.82        3.43   
 8       1000          0.76           6.64         0.78              0.79         0.26      0.80        6.64   
 16      1000          0.75           6.95         0.78              0.79         0.27      0.80        6.95   
 32      1000          0.82           29.77        0.84              0.87         0.43      1.31        29.77  
 64      1000          0.82           7.17         0.84              0.85         0.21      0.87        7.17   
 128     1000          0.84           3.89         0.87              0.87         0.08      0.90        3.89   
 256     1000          1.16           5.92         1.17              1.19         0.23      1.21        5.92   
 512     1000          1.22           5.03         1.25              1.25         0.13      1.28        5.03   
 1024    1000          1.34           6.69         1.37              1.38         0.19      1.44        6.69   
 2048    1000          1.58           6.86         1.60              1.62         0.19      1.63        6.86   
 4096    1000          2.04           7.88         2.07              2.10         0.29      2.23        7.88   
---------------------------------------------------------------------------------------
[root@m02 ~]# 
```

可以看到, 64 字节的包 99% 都在 0.87 微秒完成了传输, 4096 字节的包 99% 都在 2.23 微秒完成了传输. 可以说是相当强悍, 这个数据比高频交易用的 solarflare 的大部分网卡都要强. 当然, 这是 IB 协议的情况下, 具体到以太网情况数据是没这么好看的.



## mlnx_tune - 性能瓶颈原因检查

```
[root@m02 ~]# mlnx_tune -r 
2019-07-12 08:18:40,831 INFO Collecting node information
2019-07-12 08:18:40,831 INFO Collecting OS information
2019-07-12 08:18:40,835 WARNING Unknown OS [Linux,3.10.0-957.el7.x86_64,('centos', '7.6.1810', 'Core')]. Tuning might be non-optimized.
...
2019-07-12 08:18:41,278 INFO Collecting IP forwarding information
2019-07-12 08:18:41,285 INFO Collecting hyper threading information
2019-07-12 08:18:41,285 INFO Collecting IOMMU information
2019-07-12 08:18:41,288 INFO Collecting driver information
2019-07-12 08:18:41,915 INFO Collecting Mellanox devices information

Mellanox Technologies - System Report

Operation System Status
UNKNOWN
3.10.0-957.el7.x86_64

CPU Status
GenuineIntel Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz Ivy Bridge
Warning: Frequency 2800.0MHz

Memory Status
Total: 125.67 GB
Free: 122.07 GB

Hugepages Status
On NUMA 1:
Transparent enabled: always
Transparent defrag: always

Hyper Threading Status
ACTIVE

IRQ Balancer Status
NOT PRESENT

Firewall Status
NOT PRESENT
IP table Status
NOT PRESENT
IPv6 table Status
NOT PRESENT

Driver Status
OK: MLNX_OFED_LINUX-4.6-1.0.1.1 (OFED-4.6-1.0.1)

ConnectX-4 Device Status on PCI 03:00.0
FW version 12.25.1020
OK: PCI Width x16
Warning: PCI Speed 5GT/s >>> PCI width status is below PCI capabilities. Check PCI configuration in BIOS.
PCI Max Payload Size 256
PCI Max Read Request 512
Local CPUs list [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]

enp3s0 (Port 1) Status
Link Type eth
OK: Link status Up
Speed 100GbE 
MTU 9000
OK: TX nocache copy 'off'

ConnectX-3 Device Status on PCI 04:00.0
FW version 2.42.5000
OK: PCI Width x8
Warning: PCI Speed 5GT/s >>> PCI width status is below PCI capabilities. Check PCI configuration in BIOS.
PCI Max Payload Size 256
PCI Max Read Request 512
Local CPUs list [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]

ib0 (Port 1) Status
Link Type ib
Warning: Link status Down >>> Check your port configuration (Physical connection, SM, IP).
Speed N/A 
MTU 4092

ib1 (Port 2) Status
Link Type ib
Warning: Link status Down >>> Check your port configuration (Physical connection, SM, IP).
Speed N/A 
MTU 4092

ConnectX-3 Device Status on PCI 06:00.0
FW version 2.42.5000
Warning: PCI Width x4 >>> PCI width status is below PCI capabilities. Check PCI configuration in BIOS.
Warning: PCI Speed 5GT/s >>> PCI width status is below PCI capabilities. Check PCI configuration in BIOS.
PCI Max Payload Size 128
PCI Max Read Request 512
Local CPUs list [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]

enp6s0 (Port 1) Status
Link Type eth
OK: Link status Up
Speed 10GbE 
MTU 1500
OK: TX nocache copy 'off'

enp6s0d1 (Port 2) Status
Link Type eth
Warning: Link status Down >>> Check your port configuration (Physical connection, SM, IP).
Speed N/A 
MTU 1500
OK: TX nocache copy 'off'

2019-07-12 08:18:47,246 INFO System info file: /tmp/mlnx_tune_190712_081839.log
```

## ibv_devinfo - 查看设备信息和状态

```
[root@fn91 qos]# ibv_devinfo 
hca_id: mlx4_0
        transport:                      InfiniBand (0)
        fw_ver:                         2.42.5000
        node_guid:                      0002:c903:003e:2930
        sys_image_guid:                 0002:c903:003e:2933
        vendor_id:                      0x02c9
        vendor_part_id:                 4099
        hw_ver:                         0x1
        board_id:                       MT_1090120019
        phys_port_cnt:                  2
        Device ports:
                port:   1
                        state:                  PORT_ACTIVE (4)
                        max_mtu:                4096 (5)
                        active_mtu:             4096 (5)
                        sm_lid:                 64
                        port_lid:               13
                        port_lmc:               0x00
                        link_layer:             InfiniBand

                port:   2
                        state:                  PORT_DOWN (1)
                        max_mtu:                4096 (5)
                        active_mtu:             4096 (5)
                        sm_lid:                 0
                        port_lid:               0
                        port_lmc:               0x00
                        link_layer:             InfiniBand
```

## ibswitches -- 查看交换机

```
ibswitches 
src/query_smp.c:199; umad (DR path slid 0; dlid 0; 0,1,20,34,4,22,1,5,5 Attr 0x11:0) bad status 110; Connection timed out
Switch  : 0xe41d2d0300169560 ports 36 "SwitchX -  Mellanox Technologies" base port 0 lid 481 lmc 0
Switch  : 0xe41d2d03001692e0 ports 36 "SwitchX -  Mellanox Technologies" base port 0 lid 496 lmc 0
Switch  : 0xe41d2d03001ceb40 ports 36 "SwitchX -  Mellanox Technologies" base port 0 lid 473 lmc 0
```

## sminfo -- 查看运行 opensm 的节点

```
[root@fn91 qos]# sminfo 
sminfo: sm lid 64 sm guid 0x2c9030037a2f1, activity count 524649304 priority 14 state 3 SMINFO_MASTER
```

```
ibnetdiscover | grep "lid 64"
src/query_smp.c:199; umad (DR path slid 0; dlid 0; 0,1,19,5,18 Attr 0x11:0) bad status 110; Connection timed out
[1]     "H-0002c9030037a2f0"[1](2c9030037a2f1)          # "mds91 HCA-1" lid 64 4xFDR
[1](2c9030037a2f1)      "S-7cfe900300fbbcc0"[1]         # lid 64 lmc 0 "MF0;BSCC-M-SX6536-O01-3-1:SX6536/L33/U1" lid 56 4xFDR
```

> 注：此方式只能查看当前作为 opensm master 的节点，即使当前集群中有多个opensm

