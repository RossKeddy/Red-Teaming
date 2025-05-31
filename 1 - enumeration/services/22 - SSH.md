
```bash
# SSH VIA Private Key
ssh -i /path/to/private/key user@hostname

# Get ports from remote machine
ssh <gateway> -L <local_port_to_listen_to>:<remote_host>:<remote_port>

# Send ports
ssh <gateway> -R <remote_port>:<local_host>:<local_port>

# Dynamic Port Forwarding
ssh -D <local proxy port> -p <remote port> <target>

-o PreferredAuthentications=password
```