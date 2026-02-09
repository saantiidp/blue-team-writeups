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

## 5. Hallazgos principales
- Se identificó tráfico HTTP sospechoso relacionado con la subida de un archivo malicioso.
- Se observó comunicación posterior entre el servidor comprometido y una IP externa, consistente con un reverse shell o canal C2.
- Se detectaron transferencias de datos que indican posible exfiltración de información.
- El patrón de tráfico sugiere una secuencia de: acceso inicial → ejecución → comunicación remota → exfiltración.

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
