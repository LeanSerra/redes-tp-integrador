# Requisitos minimos

- [x] 1 Network controller
- [x] 6 routers totales (usar 3650 y 4331)
- [x] Switches 2960
- [x] 1 Servidor DNS
- [x] 1 Servidor FTP
- [x] 1 Servidor HTTP
- [x] 1 Servidor NTP
- [x] 1 servidor DHCP en un 3650 y otro en un 4331 (no vale configurar desde la solapa services de un servidor)
- [x] Direccionamiento tipo classles para IPv4 (proponer direccionamiento basado en la RFC 1918 para la intranet, definir máscaras, etc.)
- [x] Direccionamiento público para el servidor http (inventar una IP pública y un dominio)
- [ ] Direccionamiento IPv6 (definir IP globales y link local estáticas para las líneas punto a punto, máscaras, ULA optativo, etc.)
- [ ] Rutas por default en el router de borde (en IPv4 e IPv6)
- [ ] Simular un router ISP para su conexión a internet dual stack
- [ ] Propagar rutas por default a todos los dispositivos del sistema autónomo
- [ ] Ruteo dinámico (a elección RIP, EIGRP, OSPF con sus variante para IPv6)
- [x] Vlans (definir las mismas a elección). Al menos un router 4331 ruteando en 802.1q
con switches 2960 (método router on a stick)
- [x] Hosts (laptops, PC desktop, impresoras, servers, etc. Todos conectados por medio de Fast/ Gigabit Ethernet).

# Pasos que seguí para configurar

## Configuraciones de seguridad

```
enable
configure terminal
hostname <HOSTNAME>
line console 0
password tpfinal
login
line vty 0 4
password tpfinal
login
logging synchronous
exit
no ip domain-lookup
enable secret grupo8
```

### R1

1. [Configuarciones de seguridad](#configuraciones-de-seguridad)
2. SSH
    ```
    configure terminal
    ip domain-name r1.grupo8.com
    crypto key generate rsa
    line vty 0 4
    transport input ssh
    login local
    exit
    username grupo8 privilege 15 password tpfinal
    ip ssh version 2
    ```
3. Interfaces R1
    ```
    configure terminal

    interface S0/1/0
    ip address 10.0.1.1 255.255.255.252
    clock rate 4000000
    no shutdown
    exit

    interface S0/1/1
    ip address 11.0.1.1 255.255.255.252
    no shutdown
    exit

    interface G0/0/0
    ip address 10.1.0.1 255.255.255.252
    no shutdown
    exit

    interface G0/0/1
    no ip address
    no shutdown
    exit

    interface G0/0/1.50
    encapsulation dot1Q 50
    ip address 10.0.50.1 255.255.255.0
    no shutdown
    exit

    interface G0/0/1.60
    encapsulation dot1Q 60
    ip address 10.0.60.1 255.255.255.0
    no shutdown
    exit
    exit
    write
    ```
4. Ruta estatica por defecto
    ```
    configure terminal
    ip route 0.0.0.0 0.0.0.0 S0/1/1
    ```
5. RIPv2
    ```
    configure terminal
    router rip
    version 2
    network 10.0.50.0
    network 10.0.60.0
    network 10.0.1.0
    no auto-summary
    ```
6. DHCP R1
    ```
    configure terminal

    ip dhcp excluded-address 10.0.50.1 
    ip dhcp excluded-address 10.0.60.1
    ip dhcp excluded-address 10.0.50.254

    ip dhcp pool VLAN50
    network 10.0.50.0 255.255.255.0
    default-router 10.0.50.1
    dns-server 10.0.50.254
    exit

    ip dhcp pool VLAN60
    network 10.0.60.0 255.255.255.0
    default-router 10.0.60.1
    dns-server 10.0.50.254
    exit
    ```

### R2

1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. SSH
    ```
    configure terminal
    ip domain-name r2.grupo8.com
    crypto key generate rsa
    line vty 0 4
    transport input ssh
    login local
    exit
    username grupo8 privilege 15 password tpfinal
    ip ssh version 2
    ```
3. Interfaces R2
    ```
    configure terminal

    interface S0/1/0
    ip address 10.0.1.2 255.255.255.252
    no shutdown
    exit

    interface S0/1/1
    ip address 10.0.2.1 255.255.255.252
    clockrate 4000000
    no shutdown
    exit

    interface G0/0/0
    ip address 10.2.0.1 255.255.255.252
    no shutdown
    exit
    ```
4. RIP R2
    ```
    configure terminal

    router rip
    version 2
    network 10.0.0.0
    no auto-summary
    ```

### R3

1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. SSH
    ```
    configure terminal

    ip domain-name r3.grupo8.com
    crypto key generate rsa
    line vty 0 4
    transport input ssh
    login local
    exit
    username grupo8 privilege 15 password tpfinal
    ip ssh version 2
    ``` 
3. Interfaces R3
    ```
    configure terminal

    interface S0/1/1
    ip addresss 10.0.2.2 255.255.255.252
    no shutdown
    exit

    interface G0/0/0
    ip address 10.2.0.1 255.255.255.252
    no shutdown
    exit
    ```
4. RIP R3
    ```
    configure terminal

    router rip
    version 2
    network 10.0.0.0
    no auto-summary
    ```

### SW0
1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. VLANS SW0
    ```
    configure terminal

    interface range F0/1-10
    switchport mode access
    switchport access vlan 50
    exit

    interface range F0/11-20
    switchport mode access
    switchport access vlan 60
    exit

    interface G0/2
    switchport mode access
    switchport access vlan 60
    exit

    interface G0/1
    switchport mode trunk
    switchport trunk allowed vlan 50,60
    exit

    exit
    write
    ```

### SW1
1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. SSH
    ```
    configure terminal

    ip domain-name sw1.grupo8.com
    crypto key generate rsa
    line vty 0 4
    transport input ssh
    login local
    exit
    username grupo8 privilege 15 password tpfinal
    ip ssh version 2
    ```
3. VLANS SW1
    ```
    configure terminal

    ip routing

    vlan 10
    name INGPISO1

    interface range G1/0/1-10
    switchport mode access
    switchport access vlan 10
    no shutdown
    exit

    interface vlan 10
    ip address 10.0.10.254 255.255.255.0
    no shutdown
    exit

    vlan 20
    name INGPISO2

    interface range G1/0/11-20
    switchport mode access
    switchport access vlan 20
    no shutdown
    exit

    interface vlan 20
    ip address 10.0.20.254 255.255.255.0
    no shutdown
    exit

    interface G1/0/24
    no switchport
    ip address 10.1.0.2 255.255.255.252
    no shutdown
    exit
    ```
4. RIP SW1
    ```
    configure terminal
    router rip
    version 2
    network 10.0.0.0
    no auto-summary
    ```
5. DHCP SW1
    ```
    configure terminal

    ip dhcp excluded-address 10.0.10.254 
    ip dhcp excluded-address 10.0.20.254

    ip dhcp pool VLAN10
    network 10.0.10.0 255.255.255.0
    default-router 10.0.10.254
    dns-server 10.0.50.254
    exit

    ip dhcp pool VLAN20
    network 10.0.20.0 255.255.255.0
    default-router 10.0.20.254
    dns-server 10.0.50.254
    exit
    ```

### SW2
1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. SSH
    ```
    configure terminal

    ip domain-name sw2.grupo8.com
    crypto key generate rsa
    line vty 0 4
    transport input ssh
    login local
    exit
    username grupo8 privilege 15 password tpfinal
    ip ssh version 2
    ```
3. VLANS SW2
    ```
    configure terminal

    ip routing

    vlan 30
    name ADMPISO1

    interface range G1/0/1-10
    switchport mode access
    switchport access vlan 30
    no shutdown
    exit

    interface vlan 30
    ip address 10.0.30.254 255.255.255.0
    no shutdown
    exit

    vlan 40
    name ADMPISO2

    interface range G1/0/11-20
    switchport mode access
    switchport access vlan 40
    no shutdown
    exit

    interface vlan 40
    ip address 10.0.40.254 255.255.255.0
    no shutdown
    exit

    interface G1/0/24
    no switchport
    ip address 10.2.0.2 255.255.255.252
    exit
    ```
4. RIP SW2
    ```
    configure terminal

    router rip
    version 2
    network 10.0.0.0
    no auto-summary
    ```
5. DHCP SW1
    ```
    configure terminal

    ip dhcp excluded-address 10.0.30.254 
    ip dhcp excluded-address 10.0.40.254

    ip dhcp pool VLAN30
    network 10.0.30.0 255.255.255.0
    default-router 10.0.30.254
    dns-server 10.0.50.254
    exit

    ip dhcp pool VLAN40
    network 10.0.40.0 255.255.255.0
    default-router 10.0.40.254
    dns-server 10.0.50.254
    exit
    ```

### SW3
1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. VLANS SW3
    ```
    configure terminal

    ip routing

    vlan 70
    name IMPRESORAS

    interface range G1/0/1-22
    switchport mode access
    switchport access vlan 70

    interface vlan 70
    ip address 10.0.70.254 255.255.255.240
    no shutdown
    exit

    interface G1/0/24
    no switchport
    ip address 10.3.0.2 255.255.255.252
    no shutdown
    exit
    ```
3. RIP R3
    ```
    configure terminal

    router rip
    version 2
    network 10.0.0.0
    no auto-summary
    ```

### ISP
1. [Configuraciones de seguridad](#configuraciones-de-seguridad)
2. ssh? TODO preguntar
2. Interfaces ISP
    ```
    configure terminal

    interface S0/1/1
    ip address 11.0.1.2 255.255.255.252
    clock rate 4000000
    no shutdown
    exit

    interface G0/0/0
    ip address 100.100.100.1
    255.255.255.0
    no shutdown
    exit
    ```
2. Ruta estatica por defecto
    ```
    configure terminal
    ip route 11.0.1.0 255.255.255.252 S0/1/1
    ip route 10.0.0.0 255.0.0.0 11.0.1.0
    ```

# Configuraciones Network Controller
1. Desde el navegador de ADMPC ir a http://10.0.60.2
2. `username`: `grupo8` `password`: `tpfinal`
3. Ir a provisioning
    - Solapa credentials
    - add credential
        - username: grupo8
        - password: tpfinal
        - enable password: tpfinal
        - description: SSH
    - solapa discovery
        - name: SW1
        - ip address: 10.1.0.2
        - CLI credential list: grupo8 - SSH

# Postman

[Ejemplo postman](./postman/README.md)

# Python

[Ejemplo python](./python/README.md)