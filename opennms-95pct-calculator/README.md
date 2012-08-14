# 95th percentile calculator for OpenNMS servers using JRobin

## Description

calc95 allows a user running OpenNMS+JRobin to calculate 95th percentiles for
one or more SNMP interfaces. It aggregates the traffic and will try to
automatically choose the correct traffic flow (in/out) when combining
server interfaces (e.g. eth0) with switch ports (e.g. Gi0/1).

JRobin is a Java port of RRDtool, and has been the default storage mechanism
in OpenNMS since 3.1.2.

Feel free to send any questions, suggestions or comments to me on
github -- at -- segfault.no

## Credits

This Groovy script is originally based on the following script:
http://www.opennms.org/wiki/Groovy\_Scripting#Graphing\_percentiles
(Added to the wiki by User:Daniellawson at 05:40, 17 February 2009)

## Limitations

calc95 currently requires the snmpinterface.id values from the OpenNMS
database, since there are already numerous alternatives for generating
graphs based on a single interface.

In other words, it's more suited to generate the 95pct in situations
where you have customers with multiple interfaces, or where you for any other
reason need to calculate aggregated 95th percentile values.

It probably requires slight modifications in order to work with very old
versions of Groovy (due to the CliBuilder), but has been tested and found to
be successfully working with Groovy 2.0.1 and OpenNMS 1.9.0

## Usage examples

### Help menu
<pre>
$ ./calc95 -h
usage: calc95 [options] <snmp interface id> [,<more snmp interface ids>]
 -d,--debug              Debug output
 -g,--graph <filename>   Create png graph
 -h,--help               This help menu
 -m,--month <M|MM>       Month (defaults to the previous month)
 -y,--year <YYYY>        Year (defaults to the current year)
</pre>

### 95pct for multiple switch interfaces + corresponding graph
<pre>
$ ./calc95 -g multipledemo 4120,5290,5418,23741
2012-07 1466 Mbps
</pre>

In addition to the output above, the following file will be created:
&lt;pngroot&gt;/2012-07/multipledemo.png

### 95pct for switchports + server interfaces + debug output

The following command will find the 95pct for two switch ports plus two
server interfaces for a particular period (April 2012).

Since the traffic IN to the switch equals the traffic OUT from the server,
all JRobin-data that belongs to NET-SNMP will automatically be reversed in
order to be consistent with the rest of the traffic flow.

Debugging is enabled in this example to make that more clear.

<pre>
$ ./calc95 -d -g vg -y 2012 -m 4 31135,39125,83721,83659,83524
Sun Apr 01 00:00:00 CEST 2012
Mon Apr 30 23:59:59 CEST 2012
SELECT snmpinterface.nodeid,node.nodesysoid,snmpinterface.snmpifname,snmpinterface.snmpphysaddr 
 FROM node,snmpinterface WHERE node.nodeid=snmpinterface.nodeid AND id IN (31135,39125,83721,83659,83524)
 ORDER BY snmpinterface.nodeid ASC, snmpinterface.snmpifname ASC
(1) 1437: Gi3\_12-001e624a5a42
(2) 1437: Te1\_4-001e52a937c4
[1644 => eth4]:	We found a NET-SNMP interface - reversing the in/out octets for this interface.
(3) 1644: eth4-0050244567eb
[1659 => eth0]:	We found a NET-SNMP interface - reversing the in/out octets for this interface.
(4) 1659: eth0-00233a742b14
[1659 => eth1]:	We found a NET-SNMP interface - reversing the in/out octets for this interface.
(5) 1659: eth1
/var/www/95pct/png/2012-04/vg.png
Creating folder '/var/www/calc95/png/2012-04/'
/usr/share/opennms/share/rrd/snmp/1437/Gi3\_12-001e624a5a42/ifHCInOctets.jrb
/usr/share/opennms/share/rrd/snmp/1437/Gi3\_12-001e624a5a42/ifHCOutOctets.jrb
/usr/share/opennms/share/rrd/snmp/1437/Te1\_4-001e52a937c4/ifHCInOctets.jrb
/usr/share/opennms/share/rrd/snmp/1437/Te1\_4-001e52a937c4/ifHCOutOctets.jrb
/usr/share/opennms/share/rrd/snmp/1644/eth4-0050244567eb/ifHCOutOctets.jrb
/usr/share/opennms/share/rrd/snmp/1644/eth4-0050244567eb/ifHCInOctets.jrb
/usr/share/opennms/share/rrd/snmp/1659/eth0-00233a742b14/ifHCOutOctets.jrb
/usr/share/opennms/share/rrd/snmp/1659/eth0-00233a742b14/ifHCInOctets.jrb
/usr/share/opennms/share/rrd/snmp/1659/eth1/ifHCOutOctets.jrb
/usr/share/opennms/share/rrd/snmp/1659/eth1/ifHCInOctets.jrb
octIn1,octIn2,+,octIn3,+,octIn4,+,octIn5
octOut1,octOut2,+,octOut3,+,octOut4,+,octOut5
95th percentiles (in : out): 3.2962776925493126E9 : 6.43075640903236E7. 95pct: 3296 Mbps
2012-04 3296 Mbps
</pre>

(I originally needed to combine servers and switches in order to get a
total 95pct for some servers that were behind switches outside our own
network)
