# Server

## WireGuard

```
sudo dnf install wireguard-dkms wireguard-tools
sudo dnf copr enable jdoss/wireguaSrd
mkdir wg
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

## TunSafe

```
sudo tunsafe genkey | tee privatekey | sudo tunsafe pubkey > publickey
sudo tunsafe stop tun0
sudo tunsafe start -d tunsafe.conf
```

## Misc

```
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.0/24" masquerade' --permanent
```

# Client

## Add ipset

### Ipset version

```amm
val url = "https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt"
val londonNetworks = requests.get(url).text.split('\n')
%('sudo, 'ipset, 'create, 'london_networks, "hash:net")
londonNetworks foreach (x => %('sudo, 'ipset, 'add, 'london_networks, x))
```

```bash
sudo bash -c "sudo ipset save > /etc/ipset/ipset"
```
### Firewalld version

```amm
val url = "https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt"
val londonNetworks = requests.get(url).text.split('\n')
val ipsetFileContent =
  s"""<?xml version="1.0" encoding="utf-8"?>
<ipset type="hash:net">
${londonNetworks.mkString("  <entry>", "</entry>\n  <entry>", "</entry>")}
</ipset>
"""
write(wd/"london_networks.xml", ipsetFileContent)
```

```bash
sudo firewall-cmd --permanent --ipset=ipset --new-ipset-from-file=london_networks.xml
```

## Misc

```
sudo groupadd cavorite
getent group cavorite

sudo usermod -a -G cavorite $USER
sudo cp t /usr/local/bin/
sudo chmod a+xr /usr/local/bin/t

sudo mkdir /etc/tunsafe/
sudo cp tunsafe.conf /etc/tunsafe/
sudo cp tunsafe.start-stop /usr/local/bin/
sudo cp tunsafe.service /etc/systemd/system/
sudo chmod a+x /etc/tunsafe/
sudo chmod a+r -R /etc/tunsafe/tunsafe.conf
sudo chmod a+r /usr/local/bin/tunsafe.start-stop
sudo chmod a+r /etc/systemd/system/tunsafe.service
```
