# WebStrike Lab – Network Forensics

## 1. Descripción del laboratorio
Este laboratorio consiste en el análisis de un archivo PCAP para investigar el posible compromiso de un servidor web.  
El objetivo es identificar cómo ocurrió la intrusión y qué tipo de actividad maliciosa se llevó a cabo.

## 2. Objetivos
- Analizar tráfico de red sospechoso.
- Identificar posible subida de web shell.
- Detectar comunicación de tipo reverse shell / C2.
- Identificar indicios de exfiltración de datos.
- Practicar análisis forense de red con Wireshark.

## 3. Herramientas utilizadas
- Wireshark

## 4. Metodología de análisis
1. Revisión general del tráfico de red en el PCAP.
2. Filtrado de tráfico relevante (HTTP, conexiones sospechosas, transferencias de datos).
3. Identificación de peticiones y respuestas anómalas.
4. Análisis de posibles cargas maliciosas y comunicaciones externas.
5. Correlación de eventos para reconstruir la línea temporal del ataque.

## 5. Análisis paso a paso

### 5.1 Vista general del PCAP

Se comenzó con una revisión general del archivo PCAP utilizando las estadísticas de Wireshark para identificar protocolos y flujos de tráfico más relevantes.  
Se observó una cantidad significativa de tráfico HTTP, lo cual es coherente con un posible ataque contra un servidor web.

![Vista general del tráfico](images/webstrike/overview.png)

### 5.2 Análisis de tráfico HTTP

Se aplicó el siguiente filtro de visualización para centrarse en tráfico web:



## 6. Conclusión
Este laboratorio permitió practicar el análisis de tráfico de red en un escenario de compromiso realista, reforzando habilidades en:
- Network Forensics
- Identificación de actividad maliciosa en PCAPs
- Uso de Wireshark para investigación de incidentes
- Reconstrucción de la cadena de ataque desde el tráfico de red

## 7. Qué aprendí / qué mejoré
- Mejora en el uso de filtros y análisis de flujos en Wireshark.
- Mayor facilidad para identificar patrones de tráfico malicioso.
- Mejor comprensión de cómo se ve un ataque real a nivel de red.
