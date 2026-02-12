
# SteelCoffee - Traffic Analysis Exercise (2020-04-24)

## Fuente del ejercicio

Este laboratorio está basado en el ejercicio publicado en:

https://www.malware-traffic-analysis.net/2020/04/24/index.html

Nombre del ejercicio:
**2020-04-24 - Traffic Analysis Exercise - SteelCoffee**

---

## Objetivo

Documentar paso a paso el análisis del PCAP del ejercicio usando Wireshark para responder a:

- ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?
- ¿Cuál de esos clientes está infectado?
- ¿Qué tipo de malware infectó a ese cliente?

Este laboratorio se realiza en un entorno de análisis de tráfico y tiene fines educativos (Blue Team / Network Forensics).

---

## Preparación del entorno

### Descarga de los archivos

1. Descargamos los dos ZIP desde la web del ejercicio dentro de Kali Linux.
2. Descomprimimos usando la contraseña:

```
infected_20200424
```

3. Guardamos los archivos en el directorio de trabajo, por ejemplo:

```bash
┌──(kali㉿kali)-[~/Documents/STEELCOFFEE]
└─$ ls
2020-04-24-traffic-analysis-exercise-alerts.jpg
2020-04-24-traffic-analysis-exercise.pcap
```

---

## Revisión inicial del archivo de alertas

Abrimos el archivo:

```
2020-04-24-traffic-analysis-exercise-alerts.jpg
```

En esta imagen vemos un **analizador de eventos de red** que ya nos da pistas de lo ocurrido en la red:

- Consultas DNS
- Tráfico SSL / HTTPS
- Resoluciones DNS hacia dominios sospechosos (posiblemente dominios tipo `.ga`)
- Transferencia de un ejecutable Windows (probablemente un `.exe`) disfrazado
- Más tráfico SSL
- Consultas a dominios `.co`
- Alertas relacionadas con SMB, NetBIOS y posible actividad maliciosa

Esto nos indica que **hay alta probabilidad de infección en uno de los hosts Windows**.

---

## Apertura del PCAP en Wireshark

- Abrimos Wireshark.
- Vamos a **File -> Open**.
- Seleccionamos el archivo:

```
2020-04-24-traffic-analysis-exercise.pcap
```

---

## Pregunta 1: ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?

### Paso 1: Vista general de protocolos

1. Vamos a **Statistics -> Protocol Hierarchy**.
2. Observamos el resumen de tráfico del PCAP.

Datos relevantes:
- Aproximadamente **10.5%** del tráfico es **UDP**.
- Aproximadamente **89.4%** del tráfico es **TCP**.

Nos centramos en **TCP** porque concentra la mayor parte del tráfico.

---

### Paso 2: Subprotocolos dentro de TCP

Dentro de TCP se observan, entre otros:

- **SMTP (25)**: ~0% de tráfico  
- **POP3 (110)**: ~0% de tráfico  
- **NetBIOS (137)**: ~7% de tráfico → posibles autenticaciones en red  
- **LDAP (389)**: ~1% de tráfico → típico de Active Directory  
- **TLS**: ~24% de tráfico → tráfico cifrado (HTTPS)  
- **HTTP**: ~6% de tráfico  

---

### Paso 3: Enfocarnos en Kerberos

Aplicamos el filtro:

```
kerberos
```

El **source 10.0.0.10** corresponde al **Domain Controller (SteelCoffee-DC)**.

---

### Paso 4: Identificar usuarios autenticándose

- El paquete **3289** muestra un **AS-REQ** con:
```
cNameString: alyssa.fitzgerald
```

- Más adelante aparece:
```
cNameString: elmer.obrien
```

En el paquete **4185** la autenticación de `elmer.obrien` es correcta y la IP asociada es **10.0.0.167**.

---

### Paso 5: Identificar los clientes Windows

Se observan los equipos:

- **10.0.0.149** → usuario: `alyssa.fitzgerald`  
- **10.0.0.167** → usuario: `elmer.obrien`  

---

## Pregunta 2: ¿Cuál de estos dos clientes está infectado?

Buscamos descargas sospechosas mediante:

```
File -> Export Objects -> HTTP
```

Guardamos todos los objetos y analizamos los nombres de archivo.

---

## Identificación de un fichero sospechoso

Se detecta un archivo con nombre extraño:

```
8888.png%3fuid=VwBpAG4AZABvAHcAcwAgAEQAZQBmAGUAbgBkAGUAcgAgAC0AIAA2ACwAMgAxACwAMAB8AE0AaQBjAHIAbwBzAG8AZgB0ACAAVwBpAG4AZABvAHcAcwAgADEAMAAgAFAAcgBvAA==
```

Decodificando la parte Base64 en CyberChef se confirma que es un intento de camuflar un ejecutable.

---

## Análisis en VirusTotal

Al subir el archivo sospechoso a VirusTotal, múltiples motores lo detectan como:

- QBot / Qakbot
- Trojan Banker
- Backdoor
- Dropper

En los detalles aparece:

```
Original Name: CMSTP.EXE
```

Esto corresponde a la técnica:

**System Binary Proxy Execution: CMSTP**

---

## Identificar qué host lo descargó

El origen del archivo indica que fue descargado por:

- **10.0.0.167**

Por tanto, el cliente infectado es:

- **10.0.0.167** (usuario: `elmer.obrien`)

---

## Respuestas finales

### ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?

- **10.0.0.149** → `alyssa.fitzgerald`  
- **10.0.0.167** → `elmer.obrien`  

### ¿Cuál de esos clientes está infectado?

- **10.0.0.167** (usuario: `elmer.obrien`)

### ¿Qué tipo de malware infectó a ese cliente?

- **QBot / Qakbot**
- Técnica: **System Binary Proxy Execution: CMSTP**

---

## Conclusión

Este laboratorio muestra un flujo realista de análisis Blue Team:

- Uso de estadísticas de protocolos
- Análisis de Kerberos y Active Directory
- Reconstrucción de ficheros desde PCAP
- Identificación de malware real
- Atribución del incidente a un host concreto

Ejercicio muy completo de **Network Forensics y Threat Hunting básico**.
