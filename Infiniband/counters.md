[toc]





**SymbolErrors**: The total number of minor link errors detected on one or more physical lanes. This includes 8B/10B coding violations and is typically an indication of a bit error on the line.

**LinkRecovers**: The total number of times the Port Training state machine has successfully completed the link error recovery process.

**LinkDowned**: The total number of times the Port Training state machine has failed the link error recovery process and downed the link.

**RcvErrors**: The total number of packets containing an error that were received on the port.

These errors include:

- Local physical errors (ICRC, VCRC, FCCRC, and all physical errors that cause entry into the BAD PACKET or BAD PACKET DISCARD states of the packet receiver state machine)
- Malformed data packet errors (LVer, length, VL)
- Malformed link packet errors (operand, length, VL)
- Packets discarded due to buffer overrun

**RcvRemotePhysErrors**: The total number of packets marked with the EBP (End of Bad Packet) delimiter received on the port. This is typically due to a physical error that was detected and marked by an upstream port.

**RcvSwRelayErrors**: The total number of packets received on the port that were discarded because they could not be forwarded by the switch relay.

**XmtDiscards**: The total number of packets dropped because the port is down or congested.

**XmtConstraintErrors**: The total number of packets not transmitted from the switch physical port for the following reasons:

- FilterRawOutbound is true and packet is raw.
- PartitionEnforcementOutbound is true and packet fails partition key check or IP version check.

**RcvConstraintErrors**: The total number of packets received on the switch physical port that were discarded for the following reasons:

- FilterRawInbound is true and packet is raw.
- PartitionEnforcementInbound is true and packet fails partition key check or IP version check.

**LinkIntegrityErrors**: The number of times that the count of local physical errors exceeded the threshold specified by LocalPhyErrors.

**ExcBufOverrunErrors**: The number of times that OverrunErrors consecutive flow control update periods occurred, each having at least one overrun error.

**VL15Dropped**: The number of incoming VL15 packets dropped due to resource limitations (for example, lack of buffers) in the port.

**XmtData**: The total number of data octets, divided by 4, (counting in double words, 32 bits), transmitted on all VLs from the port.

**RcvData**: The total number of data octets, divided by 4, (counting in double words, 32 bits), received on all VLs from the port.

**XmtPkts**: The total number of packets transmitted on all VLs from the port.

**RcvPkts**: The total number of packets received on all VLs from the port.



The counters are located under **/sys/class/infiniband/** path as follows:

 

## Counter Groups

There are two sets of counters

1. Port Counters under the **counters** folder

2. HW counters, under the **hw_counters** folder

3. Debug Counters

 

### Port Counters

> \# ll /sys/class/infiniband/mlx5_0/ports/1/counters
> total 0
> -r--r--r-- 1 root root 4096 Aug 29 12:10 excessive_buffer_overrun_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 link_downed
> -r--r--r-- 1 root root 4096 Aug 29 12:10 link_error_recovery
> -r--r--r-- 1 root root 4096 Aug 29 12:10 local_link_integrity_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_rcv_constraint_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_rcv_data
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_rcv_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_rcv_packets
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_rcv_remote_physical_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_rcv_switch_relay_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_xmit_constraint_errors
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_xmit_data
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_xmit_discards
> -r--r--r-- 1 root root 4096 Aug 29 12:10 port_xmit_packets
> -r--r--r-- 1 root root 4096 Aug 29 12:10 symbol_error
> -r--r--r-- 1 root root 4096 Aug 29 12:10 VL15_dropped



### HW Counters (RDMA diagnostics)

> [root@l-csi-1016l ~]# ls -l /sys/class/infiniband/mlx5_0/ports/1/hw_counters/
> total 0
> -r--r--r-- 1 root root 4096 Mar 9 12:10 duplicate_request
> -r--r--r-- 1 root root 4096 Mar 9 12:10 implied_nak_seq_err
> -rw-r--r-- 1 root root 4096 Mar 9 12:10 lifespan
> -r--r--r-- 1 root root 4096 Mar 9 12:10 local_ack_timeout_err
> -r--r--r-- 1 root root 4096 Mar 9 12:10 np_cnp_sent
> -r--r--r-- 1 root root 4096 Mar 9 12:10 np_ecn_marked_roce_packets
> -r--r--r-- 1 root root 4096 Mar 9 12:10 out_of_buffer
> -r--r--r-- 1 root root 4096 Mar 9 12:10 out_of_sequence
> -r--r--r-- 1 root root 4096 Mar 9 12:10 packet_seq_err
> -r--r--r-- 1 root root 4096 Mar 9 12:10 req_cqe_error
> -r--r--r-- 1 root root 4096 Mar 9 12:10 req_cqe_flush_error
> -r--r--r-- 1 root root 4096 Mar 9 12:10 req_remote_access_errors
> -r--r--r-- 1 root root 4096 Mar 9 12:10 req_remote_invalid_request
> -r--r--r-- 1 root root 4096 Mar 9 12:10 resp_cqe_error
> -r--r--r-- 1 root root 4096 Mar 9 12:10 resp_cqe_flush_error
> -r--r--r-- 1 root root 4096 Mar 9 12:10 resp_local_length_error
> -r--r--r-- 1 root root 4096 Mar 9 12:10 resp_remote_access_errors
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rnr_nak_retry_err
> -r--r--r-- 1 root root 4096 Mar 9 12:10 roce_adp_retrans
> -r--r--r-- 1 root root 4096 Mar 9 12:10 roce_adp_retrans_to
> -r--r--r-- 1 root root 4096 Mar 9 12:10 roce_slow_restart
> -r--r--r-- 1 root root 4096 Mar 9 12:10 roce_slow_restart_cnps
> -r--r--r-- 1 root root 4096 Mar 9 12:10 roce_slow_restart_trans
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rp_cnp_handled
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rp_cnp_ignored
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rx_atomic_requests
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rx_dct_connect
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rx_icrc_encapsulated
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rx_read_requests
> -r--r--r-- 1 root root 4096 Mar 9 12:10 rx_write_requests


## Counter Description

### Port Counters Description

| **Counter**                           | **Description**                                              | **InfiniBand Spec Name**    | **Group**   |
| :------------------------------------ | :----------------------------------------------------------- | :-------------------------- | :---------- |
| port_rcv_data                         | The total number of data octets, divided by 4, (counting in double words, 32 bits), received on all VLs from the port. | PortRcvData                 | Informative |
| port_rcv_packets                      | Total number of packets (this may include packets containing Errors. This is 64 bit counter. | PortRcvPkts                 | Informative |
| port_multicast_rcv_packets            | Total number of multicast packets, including multicast packets containing errors. | PortMultiCastRcvPkts        | Informative |
| port_unicast_rcv_packets              | Total number of unicast packets, including unicast packets containing errors. | PortUnicastRcvPkts          | Informative |
| port_xmit_data                        | The total number of data octets, divided by 4, (counting in double words, 32 bits), transmitted on all VLs from the port. | PortXmitData                | Informative |
| port_xmit_packetsport_xmit_packets_64 | Total number of packets transmitted on all VLs from this port. This may include packets with errors.This is 64 bit counter. | PortXmitPkts                | Informative |
| port_rcv_switch_relay_errors          | Total number of packets received on the port that were discarded because they could not be forwarded by the switch relay. | PortRcvSwitchRelayErrors    | Error       |
| port_rcv_errors                       | Total number of packets containing an error that were received on the port. | PortRcvErrors               | Informative |
| port_rcv_constraint_errors            | Total number of packets received on the switch physical port that are discarded. | PortRcvConstraintErrors     | Error       |
| local_link_integrity_errors           | The number of times that the count of local physical errors exceeded the threshold specified by LocalPhyErrors. | LocalLinkIntegrityErrors    | Error       |
| port_xmit_wait                        | The number of ticks during which the port had data to transmit but no data was sent during the entire tick (either because of insufficient credits or because of lack of arbitration). | PortXmitWait                | Informative |
| port_multicast_xmit_packets           | Total number of multicast packets transmitted on all VLs from the port. This may include multicast packets with errors. | PortMultiCastXmitPkts       | Informative |
| port_unicast_xmit_packets             | Total number of unicast packets transmitted on all VLs from the port. This may include unicast packets with errors. | PortUnicastXmitPkts         | Informative |
| port_xmit_discards                    | Total number of outbound packets discarded by the port because the port is down or congested. | PortXmitDiscards            | Error       |
| port_xmit_constraint_errors           | Total number of packets not transmitted from the switch physical port. | PortXmitConstraintErrors    | Error       |
| port_rcv_remote_physical_errors       | Total number of packets marked with the EBP delimiter received on the port. | PortRcvRemotePhysicalErrors | Error       |
| symbol_error                          | Total number of minor link errors detected on one or more physical lanes. | SymbolErrorCounter          | Error       |
| VL15_dropped                          | Number of incoming VL15 packets dropped due to resource limitations (e.g., lack of buffers) of the port. | VL15Dropped                 | Error       |
| link_error_recovery                   | Total number of times the Port Training state machine has successfully completed the link error recovery process. | LinkErrorRecoveryCounter    | Error       |
| link_downed                           | Total number of times the Port Training state machine has failed the link error recovery process and downed the link. | LinkDownedCounter           | Error       |

 

 

### HW Counters Description

| **Counter**                | **Description**                                              | **Group**   |
| :------------------------- | :----------------------------------------------------------- | :---------- |
| duplicate_request          | Number of received packets. A duplicate request is a request that had been previously executed. | Error       |
| implied_nak_seq_err        | Number of time the requested decided an ACK. with a PSN larger than the expected PSN for an RDMA read or response. | Error       |
| lifespan                   | The maximum period in ms which defines the aging of the counter reads. Two consecutive reads within this period might return the same values | Informative |
| local_ack_timeout_err      | The number of times QP's ack timer expired for RC, XRC, DCT QPs at the sender side.The QP retry limit was not exceed, therefore it is still recoverable error. | Error       |
| np_cnp_sent                | The number of CNP packets sent by the Notification Point when it noticed congestion experienced in the RoCEv2 IP header (ECN bits).The counters was added in MLNX_OFED 4.1 | Informative |
| np_ecn_marked_roce_packets | The number of RoCEv2 packets received by the notification point which were marked for experiencing the congestion (ECN bits where '11' on the ingress RoCE traffic) .The counters was added in MLNX_OFED 4.1 | Informative |
| out_of_buffer              | The number of drops occurred due to lack of WQE for the associated QPs. | Error       |
| out_of_sequence            | The number of out of sequence packets received.              | Error       |
| packet_seq_err             | The number of received NAK sequence error packets. The QP retry limit was not exceeded. | Error       |
| req_cqe_error              | The number of times requester detected CQEs completed with errors.The counters was added in MLNX_OFED 4.1 | Error       |
| req_cqe_flush_error        | The number of times requester detected CQEs completed with flushed errors.The counters was added in MLNX_OFED 4.1 | Error       |
| req_remote_access_errors   | The number of times requester detected remote access errors.The counters was added in MLNX_OFED 4.1 | Error       |
| req_remote_invalid_request | The number of times requester detected remote invalid request errors.The counters was added in MLNX_OFED 4.1 | Error       |
| resp_cqe_error             | The number of times responder detected CQEs completed with errors.The counters was added in MLNX_OFED 4.1 | Error       |
| resp_cqe_flush_error       | The number of times responder detected CQEs completed with flushed errors.The counters was added in MLNX_OFED 4.1 | Error       |
| resp_local_length_error    | The number of times responder detected local length errors.The counters was added in MLNX_OFED 4.1 | Error       |
| resp_remote_access_errors  | The number of times responder detected remote access errors.The counters was added in MLNX_OFED 4.1 | Error       |
| rnr_nak_retry_err          | The number of received RNR NAK packets. The QP retry limit was not exceeded. | Error       |
| rp_cnp_handled             | The number of CNP packets handled by the Reaction Point HCA to throttle the transmission rate.The counters was added in MLNX_OFED 4.1 | Informative |
| rp_cnp_ignored             | The number of CNP packets received and ignored by the Reaction Point HCA. This counter should not raise if RoCE Congestion Control was enabled in the network. If this counter raise, verify that ECN was enabled on the adapter. See [HowTo Configure DCQCN (RoCE CC) values for ConnectX-4 (Linux)](https://community.mellanox.com/s/article/howto-configure-dcqcn--roce-cc--values-for-connectx-4--linux-x).The counters was added in MLNX_OFED 4.1 | Error       |
| rx_atomic_requests         | The number of received ATOMIC request for the associated QPs. | Informative |
| rx_dct_connect             | The number of received connection request for the associated DCTs. | Informative |
| rx_read_requests           | The number of received READ requests for the associated QPs. | Informative |
| rx_write_requests          | The number of received WRITE requests for the associated QPs. | Informative |
| rx_icrc_encapsulated       | The number of RoCE packets with ICRC errors.This counter was added in MLNX_OFED 4.4 and kernel 4.19 | Error       |
| roce_adp_retrans           | Counts the number of adaptive retransmissions for RoCE trafficThe counter was added in MLNX_OFED rev 5.0-1.0.0.0 and kernel v5.6.0 | Informative |
| roce_adp_retrans_to        | Counts the number of times RoCE traffic reached timeout due to adaptive retransmissionThe counter was added in MLNX_OFED rev 5.0-1.0.0.0 and kernel v5.6.0 | Informative |
| roce_slow_restart          | Counts the number of times RoCE slow restart was usedThe counter was added in MLNX_OFED rev 5.0-1.0.0.0 and kernel v5.6.0 | Informative |
| roce_slow_restart_cnps     | Counts the number of times RoCE slow restart generated CNP packetsThe counter was added in MLNX_OFED rev 5.0-1.0.0.0 and kernel v5.6.0 | Informative |
| roce_slow_restart_trans    | Counts the number of times RoCE slow restart changed state to slow restartThe counter was added in MLNX_OFED rev 5.0-1.0.0.0 and kernel v5.6.0 | Informative |

### What to do with PortRcvSwitchRelayErrors ?

What to do with port PortRcvSwitchRelayErrors? PortRcvSwitchRelayErrors are the total number of packets received on the port that were discarded because they could not be forwarded by the switch relay. Reasons for this include: DLID mapping , VL mapping , Looping (output port = input port) This counter can increase due to valid event occurring in the network. For instance if a MC (multicast) group is created by a host application, in some cases the SM may not be fast enough to create the routes to the new group, thus the switch chip (ASIC) could get a packet destined to as of yet non-existent MC group member. This could cause a PortRcvSwitchRelayError on the given switch chip (ASIC). Additional Information A network administrator defines an IPoIB link by setting up an IB partition and assigning it a unique P_Key. Since a full-duplex communication is required among IP nodes, full-membership P_Keys, that is, those with the high-order bit set to 1 shall be used. An IB partition may or may not span multiple IB subnets; and whether it does or not is mostly transparent to IPoIB. Each node attached to an IB partition must have one of its HCAs assigned the P_Key to use. Note that the P_key table of an HCA port may contain many P_Keys. It is up to the implementation to define the method by which the P_Key relevant to a particular IPoIB subnet is determined and conveyed to the IPoIB stack. By default, most SM’s create a full partition pkey (ffff, or high ordered bit enabled 7fff) that all nodes are members of. Once an IB partition is created with link attributes identified for an IPoIB link, the network administrator must create a special IB all-node multicast group (henceforth referred to as the broadcast group) with these link attributes for every node on the IPoIB link to join. The creation of an IB multicast group is through the use of the "MCMemberRecord" SA attribute as described in the IBA specification. By default, most SM’s dynamically create the multicast group for the nodes. This gives the seamless appearance of a “broadcast” domain for the IP stack. Rcvswitchrelayerrors: Required counter that records the total packets received on a port discarded because they could not be forwarded by the switch chip. Reasons for this include + DLID error on Forwarding Table Lookup. + Packets’s LRH:SL SLtoVL map lookup resulted in invalid VL + Looping error. Forwarding Table lookup selected the same port that packet arrived on. DLID errors would be easily diagnosed and eliminated by using IB ping. Corruption of the switch chip LFT table would be apparent with all IB traffic and cause complete communication failures. SLtoVL mapping issues would only occur if an IB application specifically references a Service Level that is defined on the host and SM. Then an invalid lookup table would be sent to the switch chips to cause this error. Most installations do not have applications that use Service Levels and do not implement SLtoVL mapping.

PortRcvSwitchRelayErrors是端口上接收到的由于不能被交换机中继转发而被丢弃的数据包的总数。原因包括:DLID映射、VL映射、loop(输出端口=输入端口)由于网络中发生的有效事件，计数器可能会增加。例如，如果一个MC(多播)组是由一个主机应用程序创建的，在某些情况下，SM可能不够快，无法创建到新组的路由，因此交换芯片(ASIC)可能会得到一个包，该包的目的地是尚未存在的MC组成员。这可能在给定的开关芯片(ASIC)上导致一个PortRcvSwitchRelayError。网络管理员通过设置IB分区并为其分配唯一的P_Key来定义IPoIB链路。因为IP节点之间需要全双工通信，所以full-membership P_Keys，即高阶位设为1应使用的那些



IB Error Counter Definitions and Examples
LinkDowned
Just like it says. Usually associated with a node reboot.

If not associated with a reboot, could be a failing connection. (like port flapping)


Linkspeed not at maximum
Link is not operating at full speed. (i.e. 2.5 Gbps, 5.0Gbps, 10.0Gbps)

Usually a reseat of cable/card resolves the issue.

Linkwidth not at maximum
Link is not operating at full width. (i.e. 4x, 8x, 12x)

Usually a reseat of cable/card resolves the issue.


PortRcvErrors
These errors can be due to local physical errors, local buffer overruns, or receiving a malformed packet.

If a malformed packet is received - this indicates a problem somewhere else on the fabric. Somebody is putting bad messages on the wire.


PortRcvRemotePhysicalErrors
Similar to !PortRcvErrors, the end bad packet EBP flag is set. Usually a problem between the physical and logical layers.


PortRcvSwitchRelayErrors
This field counts the number of packets that could not be forwarded by the switch.

The reasons for this include

1. VL mapping errors. (LANL has not implemented VLs (yet)).

2. Looping; input port and output port are the same

3. DLID errors; It is a Multicast DLID (0xC000 to 0xFFFE) not configured for this CA, or DLID is outside the LFTS range or greater than the LinearFDBTop, or Port associated with this DLID in the LFTS file does not exist.

Usually this is due to the poor implementation of multicast on IB and therefore can be ignored.


Port[Rcv|Xmit]ConstraintErrors
This is the number of packets [ received and discarded on | not transmitted by ] a port in the fabric.

There are 2 general reasons for this.

1. The filter for raw packets [ inbound | outbound ] is turned on and these are raw packets

2. The partition key or IP version check has failed.


PortXmitWait
This field counts the number of packets that had to wait before being transmitted.

It is almost always non-zero.

Really large numbers indicate congestion. If the congestion gets really bad, you will see !XmitDiscards.


RcvRemotePhys(ical)Errors
This field counts "Total number of packets marked with the EBP delimiter received on the port."

The idea is that an "End Bad Packet" can be used instead of EGP (End Good Packet) whenever you know there is something wrong with the packet. So, if a packet is passing through the fabric and some port notices a problem (i.e. bad CRC), it will end it with EBP instead of EGP. If the packet progress requires store-and-forward, an option would be to just drop it and not waste bandwidth sending EBP packets. The CA that reports this error is NOT where the corruption occurred. It occurred elsewhere in the fabric.


SymbolErrors
The interpretation of symbols within the packet is done on the HCA/CA. If the translation or interpretation fails, it creates a minor event called a symbol error. 99% of all !SymbolErrors are hardware related. If the counts are small (small being a relative term that is up for interpretation) they can be ignored. If the numbers are large and/or the same CA is reporting this error regularly it should be looked into. On a node, the HCA and/or cable should be reseated. If the reseat is unsuccessful it should be replaced. On a switch, reseat the cable or replace the cable.


VL15Drop
VL15 is the default virtual lane for management packets. They are the first to be dropped when there are resource limitations on the port. This is usually related to not enough space in the buffers. In many instances these errors can be ignored. There have been instances, however, when these messages were very closely correlated to user problems in time and fabric space. Obviously, if they are being dropped the buffers are being kept very busy with other data and therefore could indicate congestion.

XmtDiscards
This counter tracks packets that were discarded instead of transmitted. This usually indicates congestion in the fabric. The CA this packet was supposed to be sent to cannot accept it. After so many retries and/or too many incoming packets, the packet to be transmitted gets dropped. If the fabric is being routed well, without deadlocks or credit loops, these should be transient.
