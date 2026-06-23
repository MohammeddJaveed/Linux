# Linux Networking - DevOps and Full-Stack Essentials

> A practical networking command reference for DevOps engineers and full-stack developers.

---

## Why These Commands Matter

As a DevOps engineer or full-stack developer, you usually do not need every networking command. You need the commands that help you answer practical questions quickly:

- Does this server have the right IP address?
- Can the server reach the internet?
- Is DNS resolving correctly?
- Is my app listening on the right port?
- Can another machine reach this service?
- Is a firewall blocking traffic?
- Is the HTTP API or website responding correctly?

This guide focuses only on those useful commands.

---

## Quick Command Index

| Command                    | What it helps with                                  |
| -------------------------- | --------------------------------------------------- |
| `ip addr`                  | Check server IP addresses                           |
| `ip -br addr`              | Show IP addresses in a clean short format           |
| `ip route`                 | Check default gateway and routing                   |
| `ping -c 4 host`           | Test basic connectivity                             |
| `curl -I URL`              | Check if a website or API responds                  |
| `curl -v URL`              | Debug HTTP, TLS, redirects, and connection problems |
| `dig domain`               | Check DNS resolution                                |
| `dig +short domain`        | Get only the resolved IP address                    |
| `nslookup domain`          | Simple DNS lookup                                   |
| `ss -tulnp`                | Check listening ports and processes                 |
| `lsof -i :PORT`            | Find which process is using a port                  |
| `nc -vz host port`         | Test if a TCP port is reachable                     |
| `ssh user@host`            | Connect to a remote Linux server                    |
| `scp file user@host:/path` | Copy files to a remote server                       |
| `ufw status`               | Check Ubuntu firewall status                        |

---

## IP Address and Route Checks

### `ip addr` - Show IP Addresses

```bash
ip addr
```

Shows all network interfaces and their IP addresses.

Shorter version:

```bash
ip -br addr
```

Example:

```text
lo               UNKNOWN        127.0.0.1/8
ens33            UP             192.168.1.25/24
```

What to check:

| Field             | Meaning                      |
| ----------------- | ---------------------------- |
| `ens33`           | Network interface name       |
| `UP`              | Interface is enabled         |
| `192.168.1.25/24` | Server IP address and subnet |

If the server has no IP address on its main interface, it may have a DHCP, cloud networking, or interface configuration issue.

---

### `ip route` - Check Gateway

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev ens33
192.168.1.0/24 dev ens33 proto kernel scope link src 192.168.1.25
```

The important line is:

```text
default via 192.168.1.1 dev ens33
```

This means traffic to the internet goes through gateway `192.168.1.1`.

If there is no `default` route, the server may reach local machines but not the internet.

---

## Connectivity Testing

### `ping` - Test Basic Network Reachability

```bash
ping -c 4 8.8.8.8
```

Tests whether your machine can reach an external IP address.

Test a domain:

```bash
ping -c 4 google.com
```

How to interpret:

| Result                           | Meaning                                       |
| -------------------------------- | --------------------------------------------- |
| IP ping works, domain ping fails | DNS problem                                   |
| Both fail                        | Network, route, firewall, or internet problem |
| Domain ping works                | Basic network and DNS are working             |

> Some servers block ping, so a failed ping does not always mean the service is down.

---

### `curl` - Test Websites, APIs, and Load Balancers

Check only response headers:

```bash
curl -I https://example.com
```

Debug connection details:

```bash
curl -v https://example.com
```

Send a request to an API:

```bash
curl -X GET https://api.example.com/users
```

Send JSON to an API:

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Ali"}'
```

Common results:

| Result                   | Meaning                                    |
| ------------------------ | ------------------------------------------ |
| `200 OK`                 | Request succeeded                          |
| `301` / `302`            | Redirect                                   |
| `401`                    | Authentication required or invalid token   |
| `403`                    | Access denied                              |
| `404`                    | Route or resource not found                |
| `500`                    | Server-side error                          |
| `Connection refused`     | Nothing is listening on that port          |
| `Connection timed out`   | Firewall, routing, or network path problem |
| `Could not resolve host` | DNS problem                                |

For full-stack work, `curl` is one of the most important tools because it tests the backend without needing a browser or frontend.

---

## DNS Checks

### `dig` - Check DNS Resolution

```bash
dig example.com
```

Short output:

```bash
dig +short example.com
```

Check a specific DNS record:

```bash
dig A example.com
dig CNAME www.example.com
dig MX example.com
dig TXT example.com
```

Query a specific DNS server:

```bash
dig @8.8.8.8 example.com
```

Use this when a domain is not resolving or when DNS changes have been made and you need to verify them.

---

### `nslookup` - Simple DNS Lookup

```bash
nslookup example.com
```

This is a simpler DNS check. It is useful when you just need to confirm which IP address a domain resolves to.

---

## Ports and Running Services

### `ss -tulnp` - Check Listening Ports

```bash
sudo ss -tulnp
```

This shows which services are listening for connections.

Options:

| Option | Meaning              |
| ------ | -------------------- |
| `-t`   | TCP ports            |
| `-u`   | UDP ports            |
| `-l`   | Listening ports only |
| `-n`   | Show port numbers    |
| `-p`   | Show process name    |

Example:

```text
tcp LISTEN 0 128 0.0.0.0:22      0.0.0.0:* users:(("sshd",pid=700))
tcp LISTEN 0 128 127.0.0.1:3000  0.0.0.0:* users:(("node",pid=1200))
```

Important difference:

| Address             | Meaning                             |
| ------------------- | ----------------------------------- |
| `127.0.0.1:3000`    | Only available from the same server |
| `0.0.0.0:3000`      | Available on all network interfaces |
| `192.168.1.25:3000` | Available only on that specific IP  |

If your app listens on `127.0.0.1`, other machines cannot connect to it directly.

---

### `lsof -i :PORT` - Find the Process Using a Port

```bash
sudo lsof -i :3000
```

Useful when a dev server, backend, or database cannot start because the port is already in use.

Examples:

```bash
sudo lsof -i :80
sudo lsof -i :443
sudo lsof -i :5432
sudo lsof -i :3000
```

---

### `nc` - Test Whether a Port Is Reachable

```bash
nc -vz example.com 443
```

Examples:

```bash
nc -vz server-ip 22
nc -vz server-ip 80
nc -vz server-ip 443
nc -vz database-host 5432
nc -vz redis-host 6379
```

This is useful for checking SSH, web servers, PostgreSQL, Redis, and internal services.

How to interpret:

| Result      | Meaning                                             |
| ----------- | --------------------------------------------------- |
| `succeeded` | Port is reachable                                   |
| `refused`   | Host reached, but service is not listening          |
| `timed out` | Firewall, security group, routing, or network issue |

---

## SSH and File Copying

### `ssh` - Connect to a Server

```bash
ssh username@server-ip
```

Using a private key:

```bash
ssh -i ~/.ssh/key.pem username@server-ip
```

Common SSH ports:

| Port        | Purpose                                            |
| ----------- | -------------------------------------------------- |
| `22`        | Default SSH port                                   |
| Custom port | Some servers use a different SSH port for security |

Connect using a custom port:

```bash
ssh -p 2222 username@server-ip
```

---

### `scp` - Copy Files to or from a Server

Copy local file to server:

```bash
scp app.log username@server-ip:/tmp/
```

Copy file from server to local machine:

```bash
scp username@server-ip:/var/log/syslog .
```

Copy a folder:

```bash
scp -r ./project username@server-ip:/home/username/
```

---

## Firewall Basics

### `ufw` - Ubuntu Firewall

Check status:

```bash
sudo ufw status
```

Allow SSH:

```bash
sudo ufw allow 22/tcp
```

Allow HTTP and HTTPS:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Allow a development port:

```bash
sudo ufw allow 3000/tcp
```

> Before enabling or changing firewall rules on a remote server, make sure SSH is allowed.

---

## Practical Troubleshooting

### Website or API Is Not Working

Run:

```bash
curl -I https://example.com
curl -v https://example.com
dig +short example.com
```

Check:

| Symptom                  | Likely issue                              |
| ------------------------ | ----------------------------------------- |
| `Could not resolve host` | DNS issue                                 |
| `Connection timed out`   | Firewall, routing, or load balancer issue |
| `Connection refused`     | Service is not listening                  |
| `500` response           | Application/backend error                 |
| `404` response           | Wrong route, path, or deployment          |

---

### Backend App Is Not Reachable

On the server:

```bash
sudo ss -tulnp
sudo lsof -i :3000
sudo ufw status
```

From another machine:

```bash
nc -vz server-ip 3000
curl -v http://server-ip:3000
```

Check:

| Symptom                        | Likely issue                  |
| ------------------------------ | ----------------------------- |
| App not shown in `ss`          | App is not running            |
| App listens on `127.0.0.1`     | App is only available locally |
| Port blocked in `ufw`          | Firewall is blocking traffic  |
| `nc` succeeds but `curl` fails | App-level or HTTP issue       |

---

### Server Has No Internet

Run:

```bash
ip -br addr
ip route
ping -c 4 8.8.8.8
ping -c 4 google.com
```

Check:

| Symptom                             | Likely issue                        |
| ----------------------------------- | ----------------------------------- |
| No IP address                       | Network interface or DHCP issue     |
| No default route                    | Gateway issue                       |
| IP ping works but domain ping fails | DNS issue                           |
| Both pings fail                     | Network, firewall, or gateway issue |

---

## Most Important Commands to Remember

```bash
ip -br addr
ip route
ping -c 4 8.8.8.8
dig +short example.com
curl -Iv https://example.com
sudo ss -tulnp
sudo lsof -i :3000
nc -vz host 443
ssh user@host
sudo ufw status
```
