# Deploying-iPerf-on-Cisco-IOS-XE-using-IOx
Network Performance Lab: Deploying iPerf on Cisco IOS XE using IOx

Network Performance Lab: Deploying iPerf on Cisco IOS XE using IOx

Overview

As a Network Engineer, you are responsible for ensuring optimal performance across your organization's infrastructure. One common challenge is measuring throughput between a server and a router.

In this lab, you'll:

* Build a custom Docker image running iPerf
* Package it as an IOx application
* Deploy it on a Cisco IOS XE router
* Test bandwidth from a remote server to verify network performance


Step-by-Step Instructions

🔧 Build a Custom Docker Image

Step 1: Open Terminal on your Student VM.

Step 2:

```bash
cd /home/student/labs/iox
```

Step 3:

```bash
code .
```

(This opens the folder in VS Code)

Step 4: Trust the folder when prompted.

Step 5: Review the `Dockerfile` to understand its contents.

Step 6:

```bash
docker build -t iperf .
```

(Create Docker image named `iperf`)

Step 7:

```bash
docker images
```

(Verify the image is created)



Create and Upload IOx Package

Step 8: Open and review `package.yaml` in VS Code.

Step 9:
```bash
ioxclient docker package --name iperf iperf .
```

Step 10:

```bash
ls -al
```

(Verify the package `iperf.tar` was created)

Step 11: Upload to router:

```bash
scp iperf.tar cisco@10.1.1.21:/bootflash:/iperf.tar
```

---

 Enable IOx and Configure Hosting

Step 12: SSH into R1:

```bash
ssh cisco@10.1.1.21
```

Step 13-14:

```bash
configure terminal
iox
```

Step 15: Configure VPG0:

```bash
interface VirtualPortGroup0
ip address 10.0.0.254 255.255.255.0
no shutdown
```

Step 16-20: App-hosting setup:

```bash
app-hosting appid iperf
vnic gateway1 virtualportgroup 0 guest-interface 0
guest-ipaddress 10.0.0.3 netmask 255.255.255.0
app-default-gateway 10.0.0.254
docker
end
```

---

Deploy IOx Application

Step 22:

```bash
app-hosting install appid iperf package bootflash:/iperf.tar
```

Step 23:

```bash
show app-hosting list
```

(Ensure state is `DEPLOYED`)
Step 24-25:

```bash
app-hosting activate appid iperf
app-hosting start appid iperf
```

Step 26:

```bash
show app-hosting detail appid iperf
```

Step 27: Connect to console:

```bash
app-hosting connect appid iperf console
```

Step 28: Exit console:

```
Control + C (3 times)
```

---

 Verify Performance

Step 29: SSH into remote desktop:

```bash
ssh cisco@10.1.1.22
```

Step 30: Run iPerf test:

```bash
iperf3 -c 10.0.0.3
```

---

 GitHub Deployment Structure

```
iox-iperf-lab/
├── Dockerfile
├── package.yaml
├── README.md  <-- Document lab steps
└── .gitignore
```

🔨 Example Files

Dockerfile

```Dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y iperf3 && \
    apt-get clean
CMD ["iperf3", "-s"]
```

package.yaml

```yaml
descriptor-schema-version: "2.0"
info:
  name: iperf
  version: "1.0"
  description: iPerf server for IOx deployment
app:
  cpuarch: x86_64
  type: docker
  resources:
    profile: c1.small
    network:
      - interface-name: eth0
        ports:
          - port: 5201
            protocol: tcp
          - port: 5201
            protocol: udp
startup:
  rootfs: rootfs.tar
  target: iperf3 -s
```

README.md

````markdown
# IOx iPerf Network Testing App
