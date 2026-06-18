# IPSec Site-to-Site Route-Based con IKEv1

## Topología

### R2 (Sitio A)
- WAN: `200.1.15.2/24`
- LAN: `10.15.99.1/24`
- Tunnel10: `172.16.1.1/30`

### R3 (Sitio B)
- WAN: `200.1.99.2/24`
- LAN: `192.168.99.1/24`
- Tunnel10: `172.16.1.2/30`

---

# Configuración de R1 (ISP)

```bash
enable
configure terminal

interface e0/0
 ip address 200.1.15.1 255.255.255.0
 no shutdown

interface e0/1
 ip address 200.1.99.1 255.255.255.0
 no shutdown

end
write memory
```

---

# Configuración de R2 (Sitio A)

```bash
enable
configure terminal

hostname R2

interface e0/0
 ip address 200.1.15.2 255.255.255.0
 no shutdown

interface e0/1
 ip address 10.15.99.1 255.255.255.0
 no shutdown

ip route 0.0.0.0 0.0.0.0 200.1.15.1

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400

crypto isakmp key cisco address 200.1.99.2

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
 mode tunnel

crypto ipsec profile VPN-PROFILE
 set transform-set VPN-SET

interface Tunnel10
 ip address 172.16.1.1 255.255.255.252
 tunnel source 200.1.15.2
 tunnel destination 200.1.99.2
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile VPN-PROFILE

ip route 192.168.99.0 255.255.255.0 Tunnel10

end
write memory
```

---

# Configuración de R3 (Sitio B)

```bash
enable
configure terminal

hostname R3

interface e0/0
 ip address 200.1.99.2 255.255.255.0
 no shutdown

interface e0/1
 ip address 192.168.99.1 255.255.255.0
 no shutdown

ip route 0.0.0.0 0.0.0.0 200.1.99.1

crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400

crypto isakmp key cisco address 200.1.15.2

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
 mode tunnel

crypto ipsec profile VPN-PROFILE
 set transform-set VPN-SET

interface Tunnel10
 ip address 172.16.1.2 255.255.255.252
 tunnel source 200.1.99.2
 tunnel destination 200.1.15.2
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile VPN-PROFILE

ip route 10.15.99.0 255.255.255.0 Tunnel10

end
write memory
```

---

# Verificación

## Verificar IKEv1

```bash
show crypto isakmp sa
```
Estado esperado:

```text
QM_IDLE
```

## Verificar IPSec

```bash
show crypto ipsec sa
```

## Verificar túnel

```bash
show interface tunnel10
```

Resultado esperado:

```text
Tunnel10 is up
line protocol is up
```

## Pruebas de conectividad

```bash
ping 172.16.1.2
ping 192.168.99.1
ping 192.168.99.2
```
