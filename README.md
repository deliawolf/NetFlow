# NetFlow
NetFlow is an embedded Cisco IOS Software tool that reports the usage statistics of measured resources within the network, giving network managers clear insight into the traffic for analysis.

NetFlow requires three components, as shown in the preceding figure:

1. Flow Exporter: The router or network device in charge of collecting flow information and exporting it to a flow collector.
2. Flow Collector: A server that receives the exported flow information.
3. Flow Analyser: An application that analyzes flow information collected by the flow collector.

# Configuration

![Creating a NetFlow](https://raw.githubusercontent.com/deliawolf/NetFlow/main/NetFlow.png)

NetFlow configuraiton consist of four step:
1. Configure Exporter
2. Configure the flow record
3. Configure the flow monitor
4. Apply to an interface


## Configure Exporter

On R1, configure SRV1 (10.1.1.10) as the NetFlow Collector to which the NetFlow traffic will be exported.

```
R1(config)# flow exporter EXPORTER1
R1(config-flow-exporter)# description Stealthwatch 
R1(config-flow-exporter)# source loopback0
R1(config-flow-exporter)# destination 10.1.1.10
R1(config-flow-exporter)# transport udp 2055
R1(config-flow-exporter)# exit
```
In this example, the NetFlow information is sent to a Stealthwatch server acting as a NetFlow collector at 10.1.1.10 using UDP port 2055. Each flow exporter supports only one destination. You can create multiple exporters if you have multiple Stealthwatch collectors.

Using the loopback interface as the source ensures that all NetFlow packets source from the same IP address on the router.

## Configure NetFlow Record

A NetFlow record is a combination of key and nonkey fields that are used to identify flows. There are both predefined and user-defined records that can be configured.

You must specify a series of match and collect commands that tell the router which fields to include in the outgoing NetFlow PDU. The match fields are the key fields: they are used to determine the uniqueness of the flow. The collect fields are just extra info (nonkey) that you include to provide more detail to the collector for reporting and analysis.

```
R1(config)# flow record LANCOPE1
R1(config-flow-record)# match ipv4 tos
R1(config-flow-record)# match ipv4 protocol 
R1(config-flow-record)# match ipv4 source address 
R1(config-flow-record)# match ipv4 destination address 
R1(config-flow-record)# match transport source-port 
R1(config-flow-record)# match transport destination-port 
R1(config-flow-record)# match interface input 
R1(config-flow-record)# collect routing next-hop address ipv4
R1(config-flow-record)# collect ipv4 dscp 
R1(config-flow-record)# collect interface output 
R1(config-flow-record)# collect counter bytes 
R1(config-flow-record)# collect counter packets
R1(config-flow-record)# collect timestamp sys-uptime first 
R1(config-flow-record)# collect timestamp sys-uptime last
R1(config-flow-record)# exit
```

An explanation of the flow record commands used in the example is as follows:

Required Key Fields<br>

match ipv4 tos<br>
match ipv4 protocol<br> 
match ipv4 source address <br>
match ipv4 destination address<br> 
match transport source-port <br>
match transport destination-port<br> 
match interface input <br>

Others<br>

collect routing next-hop address ipv4 --- required<br>
collect ipv4 dscp  --- optional; used to generate QoS reports <br>
collect interface output --- required; used for computing bps rates<br> 
collect counter bytes --- required; used for bps calculation <br>
collect counter packets --- required; used for pps calculation <br>
collect timestamp sys-uptime first --- required; for calculating duration<br> 
collect timestamp sys-uptime last --- required; for calculating duration<br>

## Configure NetFlow Monitor

The monitor represents the memory-resident NetFlow database of the router. Flexible NetFlow allows you to create multiple independent monitors.
```
R1(config)# flow monitor MONITOR1
R1(config-flow-monitor)# description Main Cache 
R1(config-flow-monitor)# record LANCOPE1                    
R1(config-flow-monitor)# exporter EXPORTER1                 
R1(config-flow-monitor)# cache timeout active 60
R1(config-flow-monitor)# cache timeout inactive 60
R1(config-flow-monitor)# exit
```

## Apply to the Interfaces

```
R1(config)# interface Ethernet0/0
R1(config-if)# ip flow monitor MONITOR1 input             
R1(config-if)# interface Ethernet0/2
R1(config-if)# ip flow monitor MONITOR1 input      
R1(config-if)# end
```

## Verifying the Configuration

```
R1# show flow monitor
Flow Monitor MONITOR1:
  Description:       Main Cache 
  Flow Record:       LANCOPE1
  Flow Exporter:     EXPORTER1
  Cache:
    Type:              normal
    Status:            allocated
    Size:              4096 entries / 311316 bytes
    Inactive Timeout:  60 secs
    Active Timeout:    60 secs
    Update Timeout:    1800 secs
```
```
R1# show flow monitor MONITOR1 cache
```
