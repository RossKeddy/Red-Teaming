# LLMNR/NTB-NS Poisoning
https://github.com/SpiderLabs/Responder

[Responder](https://github.com/lgandx/Responder-Windows) is a tool built to listen, analyze, and poison `LLMNR`, `NBT-NS`, and `MDNS` requests and responses. It has many more functions, Analyze mode will passively listen to the network and not send any poisoned packets.

```bash
# -A is analyze mode
sudo responder -I tun0 -A 


sudo responder -I tun0 -wrf
```