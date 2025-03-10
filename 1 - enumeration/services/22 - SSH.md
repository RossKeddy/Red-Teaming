
```bash
# SSH VIA Private Key
ssh -i /path/to/private/key user@hostname

# Local Port Forwarding
ssh <gateway> -L <local_port_to_listen_to>:<remote_host>:<remote_port>

# Remote Port Forwarding
ssh <gateway> -R <remote_port>:<local_host>:<local_port>

# Dynamic Port Forwarding
ssh -D <local proxy port> -p <remote port> <target>
```