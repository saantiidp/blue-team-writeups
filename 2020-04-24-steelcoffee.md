# SteelCoffee - Traffic Analysis Exercise (2020-04-24)

## Objetivo
Documentar paso a paso el análisis del PCAP del ejercicio **2020-04-24 - Traffic Analysis Exercise - SteelCoffee**, usando Wireshark, para responder a:

- ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?
- ¿Cuál de esos clientes está infectado?
- ¿Qué tipo de malware infectó a ese cliente?

Este laboratorio se realiza en un entorno de análisis de tráfico y tiene fines educativos.

Fuente del ejercicio:
https://www.malware-traffic-analysis.net/2020/04/24/index.html

---

## Preparación

1. Descarga los dos archivos ZIP desde el sitio del ejercicio:
   - `2020-04-24-traffic-analysis-exercise.pcap.zip`
   - `2020-04-24-traffic-analysis-exercise-alerts.jpg.zip`

2. Descomprímelos con la contraseña:

```text
infected_20200424
```

3. En Kali, por ejemplo, los guardamos en:

```bash
┌──(kali㉿kali)-[~/Documents/STEELCOFFEE]
└─$ ls
2020-04-24-traffic-analysis-exercise-alerts.jpg  
2020-04-24-traffic-analysis-exercise.pcap
```

4. Abrimos primero el JPG de alertas y observamos:
- Un analizador de tráfico de red
- Consultas DNS
- Tráfico SSL / HTTPS
- Resoluciones DNS sospechosas (posible dominio .GA)
- Un ejecutable de malware disfrazado de imagen
- Más tráfico HTTPS
- Consultas a dominios .co
- Etc.

---

## Apertura del PCAP en Wireshark

- Abre Wireshark.
- Ve a **File -> Open** y selecciona el archivo PCAP.

![Abrir PCAP](images/step_1.png)

---

## Pregunta 1: ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?

### Paso 1: Vista general de protocolos

1. Ve a **Statistics -> Protocol Hierarchy**.

![Protocol Hierarchy - menú](images/step_2.png)

2. Observa el resumen de tráfico del PCAP.

![Protocol Hierarchy - resumen](images/step_3.png)

3. Datos relevantes:
- Aproximadamente **10.5%** del tráfico es **UDP**.
- Aproximadamente **89.4%** del tráfico es **TCP**.

4. Nos centramos en **TCP** porque concentra la mayor parte del tráfico y es donde es más probable encontrar actividad interesante.

---

### Paso 2: Subprotocolos dentro de TCP

Dentro de TCP se observan, entre otros:

- **SMTP (25)**: ~0% de tráfico  
- **POP3 (110)**: ~0% de tráfico  
- **NetBIOS (137)**: ~7% de tráfico → indica posibles autenticaciones en red  
- **LDAP (389)**: ~1% de tráfico → típico de Active Directory  
- **TLS**: ~24% de tráfico → cifrado (HTTPS)  
- **HTTP**: ~6% de tráfico  

---

### Paso 3: Enfocarnos en Kerberos

Kerberos es clave en entornos Windows con Active Directory.

Filtro:
```text
kerberos
```

---

## Respuestas finales

- **10.0.0.149** → usuario: `alyssa.fitzgerald`  
- **10.0.0.167** → usuario: `elmer.obrien`  

Cliente infectado:
- **10.0.0.167** (elmer.obrien)

Malware:
- **QBot / Qakbot** usando **System Binary Proxy Execution: CMSTP**

---

## Conclusión

Este ejercicio muestra un flujo realista de análisis de tráfico y respuesta a incidentes en laboratorio.
