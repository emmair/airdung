1 How to use packet injection with mac80211
2 =========================================
3 
4 mac80211 now allows arbitrary packets to be injected down any Monitor Mode
5 interface from userland.  The packet you inject needs to be composed in the
6 following format:
7 
8  [ radiotap header  ]
9  [ ieee80211 header ]
10  [ payload ]
11 
12 The radiotap format is discussed in
13 ./Documentation/networking/radiotap-headers.txt.
14 
15 Despite many radiotap parameters being currently defined, most only make sense
16 to appear on received packets.  The following information is parsed from the
17 radiotap headers and used to control injection:
18 
19  * IEEE80211_RADIOTAP_FLAGS
20 
21    IEEE80211_RADIOTAP_F_FCS: FCS will be removed and recalculated
22    IEEE80211_RADIOTAP_F_WEP: frame will be encrypted if key available
23    IEEE80211_RADIOTAP_F_FRAG: frame will be fragmented if longer than the
24 			      current fragmentation threshold.
25 
26  * IEEE80211_RADIOTAP_TX_FLAGS
27 
28    IEEE80211_RADIOTAP_F_TX_NOACK: frame should be sent without waiting for
29 				  an ACK even if it is a unicast frame
30 
31 The injection code can also skip all other currently defined radiotap fields
32 facilitating replay of captured radiotap headers directly.
33 
34 Here is an example valid radiotap header defining some parameters
35 
36 	0x00, 0x00, // <-- radiotap version
37 	0x0b, 0x00, // <- radiotap header length
38 	0x04, 0x0c, 0x00, 0x00, // <-- bitmap
39 	0x6c, // <-- rate
40 	0x0c, //<-- tx power
41 	0x01 //<-- antenna
42 
43 The ieee80211 header follows immediately afterwards, looking for example like
44 this:
45 
46 	0x08, 0x01, 0x00, 0x00,
47 	0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
48 	0x13, 0x22, 0x33, 0x44, 0x55, 0x66,
49 	0x13, 0x22, 0x33, 0x44, 0x55, 0x66,
50 	0x10, 0x86
51 
52 Then lastly there is the payload.
53 
54 After composing the packet contents, it is sent by send()-ing it to a logical
55 mac80211 interface that is in Monitor mode.  Libpcap can also be used,
56 (which is easier than doing the work to bind the socket to the right
57 interface), along the following lines:
58 
59 	ppcap = pcap_open_live(szInterfaceName, 800, 1, 20, szErrbuf);
60 ...
61 	r = pcap_inject(ppcap, u8aSendBuffer, nLength);
62 
63 You can also find a link to a complete inject application here:
64 
65 http://wireless.kernel.org/en/users/Documentation/packetspammer
66 
67 Andy Green <andy@warmcat.com>

