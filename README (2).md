# Cybersecurity Home Lab: VirtualBox + Kali Linux

My isolated practice lab for learning ethical hacking and network security. This repo documents how I built it, the problems I hit, and how I fixed them.

> Built as part of the Networkwalks Academy cybersecurity course (Week 1, Project Module 1).

## Lab at a glance

| Item | Value |
|---|---|
| Host OS | Fedora Linux 43 (workstation edition) |
| Hypervisor | Oracle VirtualBox 7.2.22_RPMFUSION |
| Guest OS | Kali Linux 2026.3 (Rolling) |
| Network type | NAT Network (`NatNetwork`, eth0 at 10.0.0.2/24) |
| Subnet | 10.0.0.0/24 |
| Kali IP | 10.0.0.2/24 |
| Gateway / DNS | 10.0.0.1 |

## Why a NAT Network?

A plain NAT adapter isolates each VM from the others. A NAT Network puts all my VMs on one private subnet, so they can talk to each other and still reach the internet through the host. That lets me later add vulnerable target machines next to Kali without exposing anything to my real home network.

## Setup steps

### 1. Install VirtualBox
Downloaded the latest installer from virtualbox.org and installed it on the host.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6b4ee1be-c368-49a8-8ae7-3249efa75b60" />

### 2. Create the NAT Network
`File > Tools > Network > NAT Networks > Create`, then set:
- Name: `NatNetwork`
- IPv4 prefix: `10.0.0.0/24`
- DHCP: [enabled / disabled]


<img width="1913" height="1047" alt="image" src="https://github.com/user-attachments/assets/8a1524af-0a12-45a9-b3ed-18dace1116df" />


### 3. Import Kali Linux
Downloaded the prebuilt VirtualBox image from kali.org, extracted it with 7-Zip, and imported the `.vbox` file.


### 4. Configure the VM
- **Network:** Adapter 1 enabled, attached to NAT Network `NatNetwork`
- **Clipboard / drag and drop:** `General > Advanced`, set to Bidirectional
- **Shared folder:** host `Downloads` folder, auto-mount on, full access

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/63115626-f6a4-4bfa-a02d-726f7892731b" />


### 5. Set a static IP inside Kali
Right-click the network icon in the panel, choose **Edit Connections**, open **Wired connection 1 > IPv4 Settings**:

- Method: Manual
- Address: `10.0.0.2`, Netmask: `24`, Gateway: `10.0.0.1`
- DNS: `10.0.0.1` (8.8.8.8 also works)


<img width="1403" height="762" alt="image" src="https://github.com/user-attachments/assets/2f5d378c-469e-4ee1-814d-1a7f895435af" />


Command-line equivalent:

```bash
sudo nmcli con mod "Wired connection 1" ipv4.method manual \
  ipv4.addresses 10.0.0.2/24 ipv4.gateway 10.0.0.1 ipv4.dns 10.0.0.1
sudo nmcli con down "Wired connection 1"
sudo nmcli con up "Wired connection 1"
```

### 6. Verify
```bash
ip a                  # confirm 10.0.0.2/24 on eth0
ping -c 3 10.0.0.1    # gateway
ping -c 3 8.8.8.8     # internet
ping -c 3 google.com  # DNS
```

<img width="1920" height="1080" alt="Screenshot_21-Nov_11-49-31_370" src="https://github.com/user-attachments/assets/ea31430a-39e1-4166-89db-4e33d34e4859" />



### 7. Snapshot
With the VM powered off, `Snapshot > Take`, named "Fresh Kali after setup". This is my clean restore point.

<img width="514" height="123" alt="image" src="https://github.com/user-attachments/assets/abf27472-54a1-481f-b428-6507464799b9" />


## Problems I ran into

**Wired connection showed as disabled and nothing appeared in the menu**
- Left-clicking the network icon only shows connect/disconnect. The editor is under right-click > Edit Connections (or run `nm-connection-editor`).
- Also checked `Devices > Network > Connect Network Adapter` and that Adapter 1 was enabled in the VM settings.
- Kali really takes time to download if your wifi is average like mine, needs patience.

**No internet after setting the static IP**
- Fix for newer Kali/VirtualBox 7 combos:
```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```
- Then rebooted the VM.

## What I learned
- some changes dont just come the vm has to be restarted for effectiveness

## Next steps
- [ ] Add a vulnerable target VM on the same NAT Network
- [ ] Practice nmap scans against it
- [ ] work on week 2

## Credits
Lab requirements from Networkwalks Academy's cybersecurity course. Screenshots and write-up are my own.
