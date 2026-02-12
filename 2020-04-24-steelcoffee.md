
# SteelCoffee - Traffic Analysis Exercise (2020-04-24)

## Objetivo
Documentar paso a paso el análisis del PCAP del ejercicio **2020-04-24 - Traffic Analysis Exercise - SteelCoffee**, usando Wireshark, para responder a:

- ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?
- ¿Cuál de esos clientes está infectado?
- ¿Qué tipo de malware infectó a ese cliente?

Fuente del ejercicio:
https://www.malware-traffic-analysis.net/2020/04/24/index.html

Este laboratorio se realiza en un entorno de análisis de tráfico y tiene fines educativos.

---

## Preparación

1. Descarga los dos archivos ZIP desde el sitio del ejercicio:
   - `2020-04-24-traffic-analysis-exercise.pcap.zip`
   - `2020-04-24-traffic-analysis-exercise-alerts.jpg.zip`

2. Descomprímelos con la contraseña:
   ```
   infected_20200424
   ```

3. En Kali Linux, por ejemplo:
   ```
   ┌──(kali㉿kali)-[~/Documents/STEELCOFFEE]
   └─$ ls
   2020-04-24-traffic-analysis-exercise-alerts.jpg  2020-04-24-traffic-analysis-exercise.pcap
   ```

4. Abre el JPG de alertas. En él se observa:
   - Peticiones DNS
   - Tráfico SSL/HTTPS (certificados)
   - Resoluciones DNS a dominios sospechosos (por ejemplo dominios tipo .GA)
   - Un malware ejecutable disfrazado de imagen
   - Más tráfico SSL
   - Consultas a dominios .co
   - Etc.

Esto nos da una pista clara de que hay descargas sospechosas en el tráfico.

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

Interpretación rápida:

- **HTTP** suele asociarse a portales web, descargas o posibles servidores de malware.
- **HTTPS (TLS)** indica tráfico cifrado, normalmente navegación web o descargas seguras.

Con solo la jerarquía de protocolos ya tenemos una buena idea de lo que ocurre en la red.
![Protocol Hierarchy - TCP](images/step_4.png)

---

### Paso 3: Enfocarnos en Kerberos

Kerberos es clave en entornos Windows con Active Directory: actúa como el “portero” que autentica usuarios y emite tickets de sesión.

1. Aplica un filtro por Kerberos:
   ```
   kerberos
   ```

![Filtro Kerberos](images/step_5.png)

2. Observa los primeros paquetes Kerberos.
   - El **source 10.0.0.10** corresponde al **Domain Controller (SteelCoffee-DC)**, la pieza más crítica de la red.

![Paquetes Kerberos](images/step_6.png)

---

### Paso 4: Identificar usuarios autenticándose

1. El paquete **3289** muestra un **AS-REQ** (Authentication Service Request).
2. El siguiente paquete indica error de autenticación.
3. En los detalles aparece:
   ```
   cNameString: alyssa.fitzgerald
   ```
   Esto indica que **alyssa.fitzgerald** intenta autenticarse contra Kerberos y falla varias veces.

![Alyssa Fitzgerald_AS-REQ](images/step_7.png)
![Alyssa Fitzgerald_error](images/step_8.png)
![Alyssa Fitzgerald](images/step_9.png)
4. Más adelante, en el paquete **3998**, aparece otra solicitud:
   ```
   cNameString: elmer.obrien
   ```
   - Primero falla.
   - En el paquete **4185** la autenticación es correcta y se emite el ticket.
   - La IP asociada es **10.0.0.167**.

![Elmer Obrien](images/step_10.png)

---

### Paso 5: Identificar los clientes Windows

1. Bajando en la captura, se observa un **HostAddress** con nombre de equipo:
   ```
   DESKTOP-GRIONXA<20>
   ```

2. Los equipos que destacan en este proceso de autenticación son:
- **10.0.0.167** (usuario: elmer.obrien)
- **10.0.0.149** (usuario: alyssa.fitzgerald)

Estos dos clientes Windows son los que tendremos en cuenta para las siguientes preguntas.

---

## Pregunta 2: ¿Cuál de estos dos clientes está infectado?

Sabemos que los dos clientes sospechosos son:
- 10.0.0.167 (elmer.obrien)
- 10.0.0.149 (alyssa.fitzgerald)

Ahora buscamos evidencias de **descarga de ficheros maliciosos**.

---

### Paso 6: Revisar tráfico TLS

Seleccionamos un stream TLS (por ejemplo el 3609) y hacemos:
- **Follow -> TCP Stream**

![Follow TCP Stream](images/step_11.png)

En este stream se observan certificados y dominios legítimos como `www.bing.com`, por lo que seguimos buscando.

---

### Paso 7: Exportar objetos HTTP

1. Ve a **File -> Export Objects -> HTTP**.

2. Aparece la lista de objetos transferidos.

![Lista de objetos HTTP](images/step_12.png)

3. Pulsa **Save All** y guarda los archivos en una carpeta.

![Archivos exportados](images/step_13.png)

En esa carpeta aparecen muchos ficheros reconstruidos a partir del tráfico de red: HTML, CSS, JS, imágenes, etc.

---

### Paso 8: Buscar archivos sospechosos

Sabemos por las alertas que hay un **Windows executable enviado como si fuera una imagen**.
![alerta](images/step_14.png)
Ordenamos por **Content-Type** o revisamos los nombres y encontramos uno especialmente raro:

```
8888.png%3fuid=VwBpAG4AZABvAHcAcwAgAEQAZQBmAGUAbgBkAGUAcgAgAC0AIAA2ACwAMgAxACwAMAB8AE0AaQBjAHIAbwBzAG8AZgB0ACAAVwBpAG4AZABvAHcAcwAgADEAMAAgAFAAcgBvAA==
```

![PNG sospechoso](images/step_15.png)

Este nombre es claramente sospechoso.

---

### Paso 9: Decodificar el nombre con CyberChef

Tomamos solo la parte Base64:

```
VwBpAG4AZABvAHcAcwAgAEQAZQBmAGUAbgBkAGUAcgAgAC0AIAA2ACwAMgAxACwAMAB8AE0AaQBjAHIAbwBzAG8AZgB0ACAAVwBpAG4AZABvAHcAcwAgADEAMAAgAFAAcgBvAA==
```

La decodificamos en **CyberChef** usando **From Base64** y vemos que intenta hacerse pasar por algo legítimo de Windows.

El resultado decodificado muestra una cadena que hace referencia a **Windows Defender / Microsoft Windows**, lo que indica que el archivo intenta hacerse pasar por algo legítimo del sistema para engañar al usuario.

Este es un patrón típico en malware:
- Usar nombres o descripciones que parecen componentes de Windows
- Disfrazar un ejecutable malicioso como si fuera un archivo inofensivo (por ejemplo, una imagen .png)
- Aumentar la probabilidad de que el usuario lo ejecute sin sospechar

![descifrado](images/step_16.png)
---

### Paso 10: Analizar el archivo en VirusTotal

Subimos el fichero exportado a **VirusTotal**.

Resultados:
- Múltiples detecciones como **QBot / Qakbot**
- Troyano bancario
- Backdoor
- Dropper

En la pestaña **Details** aparece:

```
Original Name: CMSTP.EXE
```

Esto corresponde a la técnica conocida como:

> **System Binary Proxy Execution: CMSTP**

---

### Paso 11: Identificar qué host lo descargó

Volviendo a **File -> Export Objects -> HTTP** y revisando el objeto sospechoso:

- El archivo proviene de: **119.31.234.40**
- El host que lo descarga es: **10.0.0.167**

Por tanto, el cliente infectado es:
- **10.0.0.167** (usuario: elmer.obrien)

---

## Respuestas finales

### ¿Cuáles son los dos clientes Windows y qué usuarios tienen asociados?

- **10.0.0.149** → usuario: `alyssa.fitzgerald`
- **10.0.0.167** → usuario: `elmer.obrien`

### ¿Cuál de esos dos clientes está infectado?

- **10.0.0.167** (usuario: `elmer.obrien`)

### ¿Qué tipo de malware infectó a ese cliente?

- **QBot / Qakbot**, usando la técnica:
  > **System Binary Proxy Execution: CMSTP**

---

## Conclusión

Este ejercicio muestra un flujo realista de análisis de tráfico:

- Identificación de usuarios y hosts Windows mediante Kerberos.
- Priorización de protocolos usando estadísticas.
- Reconstrucción de ficheros a partir de un PCAP.
- Identificación de un ejecutable malicioso disfrazado de imagen.
- Análisis en VirusTotal y atribución a un host concreto.
- Enfoque práctico de **Network Traffic Analysis** e **Incident Response** en laboratorio.
