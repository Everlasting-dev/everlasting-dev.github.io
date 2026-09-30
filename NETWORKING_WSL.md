# Slip and WSL / Virtual Network Addresses

## Why 172.21.0.1 showed up

WSL 2, Hyper-V, Docker, VMware, VirtualBox, and some Wi-Fi Direct features create virtual network adapters on the Windows host. Those adapters often use private ranges such as `172.16.0.0/12`; for example, WSL may show the host-side adapter as `172.21.0.1`.

That address is real on the local machine, but it is not the normal Wi-Fi or Ethernet address that another PC on the room LAN can reach. If Slip advertises that virtual address, another PC tries to connect to a network that only exists inside this machine, so discovery or transfer appears broken.

Slip 1.3.4 filters these virtual adapters out when choosing the LAN address and when broadcasting presence. The Check connection screen still lists ignored virtual addresses so you can see what Windows reported.

## What to use instead

Use the address from the real adapter, usually Wi-Fi or Ethernet:

```powershell
Get-NetIPConfiguration | Where-Object { $_.IPv4DefaultGateway } | Select-Object InterfaceAlias,IPv4Address,IPv4DefaultGateway
```

In Slip, open:

```text
More -> Check connection
```

The main `LAN address` should be your reachable address, such as `192.168.x.x` or `10.x.x.x`. Virtual addresses should appear on the `Ignored` line.

## Temporarily stop WSL networking

If you are presenting and do not need WSL running, this is usually enough:

```powershell
wsl --shutdown
```

That stops WSL instances. Windows may remove or quiet the WSL virtual adapter until WSL starts again.

## Disable or enable the WSL adapter

Run PowerShell as Administrator.

List the virtual adapters:

```powershell
Get-NetAdapter | Where-Object { $_.Name -like 'vEthernet*' } | Select-Object Name,Status,InterfaceDescription
```

Disable the WSL adapter, using the exact name shown on your PC:

```powershell
Disable-NetAdapter -Name 'vEthernet (WSL (Hyper-V firewall))' -Confirm:$false
```

Enable it again:

```powershell
Enable-NetAdapter -Name 'vEthernet (WSL (Hyper-V firewall))'
```

The adapter name can vary between Windows builds. Do not guess if the name is different; copy it from `Get-NetAdapter`.

## Demo checklist

1. Connect both PCs to the same Wi-Fi or Ethernet network.
2. In Slip, open `More -> Check connection`.
3. Confirm the main LAN address is not `172.21.0.1` or another virtual adapter address.
4. Confirm `Network:` is Private, not Public.
5. If a receiver still does not appear, use Quick receive's Type IP option and enter the receiver's real Wi-Fi or Ethernet address.
