https://github.com/nicocha30/ligolo-ng

```bash
sudo ip tuntap add user $USER mode tun ligolo

sudo ip link set ligolo up

then you can start ligolo with 
./proxy -selfcert -laddr 0.0.0.0:9001

connect the client with 
.\agent.exe -connect 10.10.14.165:9001 -ignore-cert

go into ligolo select the session

enter start after selecting the session

sudo ip route add 10.0.0.0/24 dev ligolo
```