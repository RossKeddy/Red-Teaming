```
sudo ip tuntap add user $USER mode tun ligolo

sudo ip link set ligolo up

then you can start ligolo with 
./proxy -selfcert

connect the client with 
.\agent.exe -connect 10.10.14.220:11601 -ignore-cert

go into ligolo select the session

enter start after selecting the session

sudo ip route add 192.168.100.0/24 dev ligolo
```