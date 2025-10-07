# **LABORATORIO OSPFv2** 

================================================

## **OBJETIVO**

Configurar OSPFv2 en área 0 entre dos routers Cisco con una LAN en cada extremo y verificar conectividad extremo a extremo entre las PCs.

---

## **EQUIPOS**

* 2× Routers Cisco ISR 1941 (R1 y R2)  
* 2× Switches Cisco 2960-24TT (S1 y S2)  
* 2× PCs (PC0 y PC1)

---

## **CABLEADO EXACTO (PUERTO A PUERTO)**

### **Tipo de cable a usar:**

* **Copper Straight-Through (recto)** → PC ↔ Switch y Router ↔ Switch  
* **Copper Cross-Over (cruzado)** → Router ↔ Router

### **Conexiones físicas:**

| # | Dispositivo A / Puerto | Dispositivo B / Puerto | Tipo de cable |
|---|--------------------------|--------------------------|---------------|
| 1 | PC0 FastEthernet0        | S1 FastEthernet0/1       | Recto        |
| 2 | R1 GigabitEthernet0/0    | S1 FastEthernet0/2       | Recto        |
| 3 | PC1 FastEthernet0        | S2 FastEthernet0/1       | Recto        |
| 4 | R2 GigabitEthernet0/0    | S2 FastEthernet0/2       | Recto        |
| 5 | R1 GigabitEthernet0/1    | R2 GigabitEthernet0/1    | Cruzado      |

---

## **DIRECCIONAMIENTO IP**

| Dispositivo | Interfaz | IP Address       | Máscara              | Descripción         |
|-------------|----------|------------------|-----------------------|---------------------|
| R1         | g0/0     | 192.168.10.1     | 255.255.255.0        | LAN R1             |
| R1         | g0/1     | 10.10.10.1       | 255.255.255.252      | Enlace a R2       |
| PC0        | -        | 192.168.10.10    | 255.255.255.0        | GW: 192.168.10.1  |
| R2         | g0/0     | 192.168.20.1     | 255.255.255.0        | LAN R2             |
| R2         | g0/1     | 10.10.10.2       | 255.255.255.252      | Enlace a R1       |
| PC1        | -        | 192.168.20.10    | 255.255.255.0        | GW: 192.168.20.1  |

> **Recordatorio OSPF/Wildcard**:  
> /24 → 0.0.0.255  
> /30 → 0.0.0.3

---

## **CONFIGURACIÓN CLI**

### **> R1**

```bash
enable
conf t
hostname R1

interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 10.10.10.1 255.255.255.252
 no shutdown

router ospf 10
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.10.10.0 0.0.0.3 area 0
 passive-interface g0/0

end
wr
```

---

### **> R2**

```bash
enable
conf t
hostname R2

interface g0/0
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 10.10.10.2 255.255.255.252
 no shutdown

router ospf 10
 router-id 2.2.2.2
 network 192.168.20.0 0.0.0.255 area 0
 network 10.10.10.0 0.0.0.3 area 0
 passive-interface g0/0

end
wr
```

---

## **CONFIGURACIÓN DE LAS PCs**

### **PC0** → Desktop > IP Configuration

| Campo         | Valor             |
|---------------|--------------------|
| IP Address    | 192.168.10.10     |
| Subnet Mask   | 255.255.255.0     |
| Default GW    | 192.168.10.1     |

### **PC1** → Desktop > IP Configuration

| Campo         | Valor             |
|---------------|--------------------|
| IP Address    | 192.168.20.10     |
| Subnet Mask   | 255.255.255.0     |
| Default GW    | 192.168.20.1     |

---

## **VERIFICACIÓN**

### 1) Estado de interfaces (deben estar *up/up* con IP correcta)

```bash
R1# show ip interface brief
R2# show ip interface brief
```

---

### 2) Confirmar cableado (vecinos CDP)

```bash
R1# show cdp neighbors
R2# show cdp neighbors
```

**Esperado:**

- R1: Switch por Gi0/0 y R2 por Gi0/1  
- R2: Switch por Gi0/0 y R1 por Gi0/1

---

### 3) Vecindad OSPF (debe haber 1 vecino en FULL sobre Gi0/1)

```bash
R1# show ip ospf neighbor
R2# show ip ospf neighbor
```

---

### 4) Tabla de rutas (rutas OSPF marcadas con “O”)

```bash
R1# show ip route
(Esperado: O 192.168.20.0/24 via 10.10.10.2, GigabitEthernet0/1)

R2# show ip route
(Esperado: O 192.168.10.0/24 via 10.10.10.1, GigabitEthernet0/1)
```

---

### 5) Pruebas de ping

```bash
R1# ping 10.10.10.2
PC0> ping 192.168.20.10
PC1> ping 192.168.10.10
```

---

## **SALIDAS ESPERADAS**

### **Vecindad OSPF (ejemplo en R1)**

```bash
R1# show ip ospf neighbor

Neighbor ID     Pri   State   Dead Time   Address      Interface
2.2.2.2           1   FULL/DR 00:00:3x    10.10.10.2   GigabitEthernet0/1
```

---

### **Tabla de rutas (líneas clave en R1)**

```bash
R1# show ip route

O     192.168.20.0/24 [110/2] via 10.10.10.2, GigabitEthernet0/1
C     192.168.10.0/24 is directly connected, GigabitEthernet0/0
```

---

## **CHECKLIST RÁPIDO**

- [ ] R1 g0/1 ↔ R2 g0/1 (cable cruzado) y PCs/routers a switches con cable recto  
- [ ] IPs correctas: enlace /30 en g0/1, LAN /24 en g0/0  
- [ ] OSPF proceso 10, área 0 en ambos  
- [ ] `passive-interface` **solo** en g0/0 (LAN)  
- [ ] 1 vecino OSPF en FULL  
- [ ] Rutas “O” presentes en ambos routers  
- [ ] Ping PC0 ↔ PC1 exitoso

---

## **DIAGNÓSTICO RÁPIDO SI FALLA**

### 1) ¿IP en la interfaz correcta?

```bash
R# show ip interface brief
(Si 10.10.10.x aparece en g0/0 en vez de g0/1, corrige).
```

---

### 2) ¿Cable mal conectado?

```bash
R# show cdp neighbors
(Debe verse R1 Gi0/1 ↔ R2 Gi0/1).
```


