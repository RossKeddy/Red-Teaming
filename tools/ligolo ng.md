```
sudo ip tuntap add user $USER mode tun ligolo

sudo ip link set ligolo up

then you can start ligolo with 
./proxy -selfcert

connect the client with 
.\agent.exe -connect 10.10.14.133:11601 -ignore-cert

go into ligolo select the session

enter start after selecting the session

sudo ip route add 10.129.195.229/24 dev ligolo
```