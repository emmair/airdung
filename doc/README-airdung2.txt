airdung2 notes - emm 2/2014

updated: 1/14/2016 - emm

--------------------------------------------------------------------------------------

airdung2 is a packet generation tool for testing the IEEE 802.11 a/b/g/n specification.

At this time, there are five different packet type/subtypes available with some fields
that are customizable. Default values for the packets are summarized at the end of this 
document, for details on all fields refer to airdung.h.

airdung2 is dependent on the wifi chipset and drivers used. Notes below outline 'best 
practices' to date.

For Ubuntu 12.xx:

1) For best results stop network-manager (nm):

	service network-manager stop

Although, nm can run, and you can have multiple interfaces up in normal/managed mode at the
same time, funny things might happen after long durations of sending test packets.

For Ubuntu 14.xx:

1) In /etc/NetworkManager/NetworkManager.conf, add the following line:

unmanaged-devices=mac:<hwaddr>;mac=<hwaddr2> ... etc

for as many devices as necessary. Then restart NetworkManager (i.e. ps aux, kill PID #...) 

Watch proximity of send/receive devices, if they are too close, errors may occur resulting
in dropped packets. Attenuate as necessary, this is easily done by removing one of the tx/rx
devices antennas.

2) Cards that have been tested to work:

Alfa AWUS036NH b/g/n - uses rt2800usb driver. Note: do *not* use the rt2870sta driver since
                                                    this driver is known not to work
Alfa AWUS036H b/g - uses rtl8187 driver.

Atheros 9k based cards - uses ath9k driver (both usb and PCIe)

Cards not working at this time:

Rosewill N600UBE a/b/g/n - driver issues.

3) Use airmon-ng (located in the scripts directory) to enumerate available wifi interfaces or
use:
	iw list

4) Useful iw commands:

Set card in monitor mode:
	iw dev wlan1 set monitor none
	
Set bitrate mask:
	iw dev wlan1 set bitrates mcs-2.4 0
	
Set channel and HT channel (with debug):
	iw --debug dev wlan1 set channel 10 HT40-
	
without debug:
	iw  dev wlan1 set channel 10 HT40-

Other notes

-some usb cards have had issues with buffer overruns (not sure if this is kernel or driver 
realted). These issues have not been present with PCIe devices. A hack was added to airdung2 to
deal with this until the problem is resolved. 

-delay between packets (-q option) will be measured by tcpdump/wireshark depending on the data
rate as well as packet length. Shorter packets at higher rates will approach the specified delay 
while longer packets and lower rates will add latency to the measurement. This is in addition to the
clock resolution of the capture machine.

-tcpdump is the preferred method for measuring accurate results as it incurs the least CPU 
overhead. Wireshark is primarily used for troublshooting and "sanity checks".

Command line examples:

./airdung2 -i wlan1 -t 1000 -m 10 -o 5 -x 10 -n 18 -s bb:bb:bb:bb:bb:bb

airdung - ver 2.0

Opening kernel socket ...done.
Setting up interface: wlan1 ...Retries set to: 0
done.

Current channel: 10 (2457 MHz)
Current Bit Rate: 18.0 Mb/s

Mode: 5
Source address:      bb:bb:bb:bb:bb:bb
Destination address: ff:ff:ff:ff:ff:ff
BSSID:               ff:ff:ff:ff:ff:ff
Data (Large)

Sent 10 packet(s).

./airdung2 -i wlan1 -t 1000 -m 10 -o 1 -x 10 -n 18 -s bb:bb:bb:bb:bb:bb -e JaY

airdung - ver 2.0

Opening kernel socket ...done.
Setting up interface: wlan1 ...Retries set to: 0
done.

Current channel: 10 (2457 MHz)
Current Bit Rate: 1.0 Mb/s

Mode: 1
Source address:      bb:bb:bb:bb:bb:bb
Destination address: ff:ff:ff:ff:ff:ff
BSSID:               ff:ff:ff:ff:ff:ff
Beacon 
ssid: JaY ssid_len: 3

Sent 10 packet(s).


-example packet captures are in airdung-captures.pcap.

Useful Wireshark/tcpdump filters:
---------------------------------

*Wireshark Filters*

-filter on Source MAC address (display/keep only packets with the source MAC
 address equal to 00:11:22:33:44:55):

wlan.sa==00:11:22:33:44:55

-filter on Destination MAC address (display/keep only packets with the 
 destination MAC address equal to 00:11:22:33:44:55):

wlan.da==00:11:22:33:44:55

-filter on Source IP address (display/keep only packets with the 
 source IP address equal to 192.168.1.1):

ip.src==192.168.1.1

-filter on BSSID (display/keep only packets with the BSSID  address 
 equal to bb:bb:bb:bb:bb:bb):

wlan.bssid==bb:bb:bb:bb:bb:bb

*Tcpdump Examples*

-filter on Source MAC address (display/keep only packets with the source MAC
 address equal to 00:11:22:33:44:55):

tcpdump -i wlan0 -s 4096 -ttt -q ether src 00:11:22:33:44:55

-filter on Source IP address (display/keep only packets with the 
 source IP address equal to 192.168.1.1):

tcpdump -i wlan0 -s 4096 -ttt -q src 192.168.1.1

-filter on BSSID (display/keep only packets with the BSSID  address 
 equal to bb:bb:bb:bb:bb:bb):

tcpdump -i wlan0 -s 4096 -ttt -q wlan addr3 bb:bb:bb:bb:bb:bb

-filter on all data packets:

tcpdump -i wlan0 -s 4096 -ttt -q -nne 'wlan type data subtype data'

<<<<<<< HEAD

Useful tshark examples
----------------------

=======
Useful tshark examples
----------------------


>>>>>>> 04ffe8e82d9e70c483ea2f16f69ea23ce7c1f0f9
tshark -i wlan0 -q ether src 00:11:22:33:44:55 -z io,stat,1.2,"AVG(frame.time_delta)frame.time_delta"

-filter on Source MAC address 00:11:22:33:44:55
-keeps stats on average delta time between pkts for 1200ms intervals

tshark -i wlan0 -q ether src 00:11:22:33:44:55 -e frame.time_delta -Tfields

-filter on Source MAC address 00:11:22:33:44:55
-display delta time between pkts

Version 2.0.x of tshark:

tshark -i wlan0 -f "ether src bb:bb:bb:bb:bb:bb:bb" -z io,stat,1.2,"AVG(frame.time_delta)frame.time_delta"


Appendix A:

Data rate mappings (Note: this is specified in airdung.h with the bitrates array)
------------------

option -n

1	-	1.0 mb/s
2	-	2.0 mb/s
5	-	5.5 mb/s
6	-	6.0 mb/s
9	-	9.0 mb/s
11	-	11.0mb/s
12	-	12.0mb/s
18	-	18.0mb/s
24	-	24.0mb/s
36	-	36.0mb/s
48	-	48.0mb/s
54	-	54.0mb/s

Packet Types
------------

option -o

1 - Beacon (default SSID=test) - pkt len: 294
2 - Probe Request (default SSID= test) - pkt len: 42
3 - Null Data - pkt len: 24
4 - Data (Medium)- pkt len: 717
5 - Data (Large)- pkt len: 1498


Alfa AWUS036H Rx sensitivity - rtl8187 chipset (From on-line documentation)
----------------------------

 54mb/s (64QAM)		-76dBm (worst case docs spec -68dBm)  
 48mb/s (64QAM)		-71dBm
 36mb/s (16QAM)		-78dBm
 24mb/s (16QAM)		-80dBm
 18mb/s (QPSK)		-81dBm
 12mb/s (QPSK)		-82dBm
 11mb/s (CCK)           -86dBm (PER < 8 percent)
  9mb/s (BPSK)		-85dBm
  6mb/s (BPSK)		-91dBm
5.5mb/s (CCK)		-89dBm (PER < 8 percent)	
  2mb/s (BPSK)		-92dBm (PER < 8 percent)
  1mb/s (BPSK)		-94dBm (PER < 8 percent)    

*all values reflect PER < 10 percent, pkt size 1024 bytes @25Deg. C, unless
 otherwise indicated.  
 
Alfa AWUS036NH Rx sensitivity - rt2870 chipset (From on-line documentation)
-----------------------------

 54mb/s (64QAM)		-76dBm (worst case docs spec -68dBm)  
 48mb/s (64QAM)		-71dBm
 36mb/s (16QAM)		-78dBm
 24mb/s (16QAM)		-80dBm
 18mb/s (QPSK)		-81dBm
 12mb/s (QPSK)		-82dBm
 11mb/s (CCK)           -86dBm (PER < 8 percent)
  9mb/s (BPSK)		-85dBm
  6mb/s (BPSK)		-91dBm
5.5mb/s (CCK)		-89dBm (PER < 8 percent)	
  2mb/s (BPSK)		-92dBm (PER < 8 percent)
  1mb/s (BPSK)		-94dBm (PER < 8 percent)    

*all values reflect PER < 10 percent, pkt size 1024 bytes @25Deg. C, unless
 otherwise indicated.  

