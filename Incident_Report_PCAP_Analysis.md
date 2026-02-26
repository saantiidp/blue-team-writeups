# Incident Report -- PCAP Analysis (LAN 10.1.17.0/24)

------------------------------------------------------------------------

## 1. Información inicial del entorno

**Rango LAN:** 10.1.17.0/24\
- 10.1.17.0 → Dirección de red\
- 10.1.17.255 → Broadcast\
- Rango útil: 10.1.17.1 -- 10.1.17.254

**Dominio Active Directory:** bluemoontuesday.com\
**Nombre NetBIOS:** BLUEMOONTUESDAY\
**Domain Controller:** 10.1.17.2\
**Autenticación:** Kerberos (TCP/88)

Formato de usuario en dominio: BLUEMOONTUESDAY`\username`{=tex}

------------------------------------------------------------------------

## 2. Análisis inicial del PCAP

Wireshark → Statistics → Protocol Hierarchy

Distribución:

-   93.2% TCP
-   5.9% UDP
-   Resto ICMP / ARP

El tráfico relevante está en TCP.

------------------------------------------------------------------------

## 3. Subprotocolos dentro de TCP

### TLS (61.6% de los bytes)

TLS protege HTTP (HTTPS).\
El alto porcentaje en bytes indica transferencia significativa de datos
(posibles descargas).

### HTTP (3.2%)

Permite ver: - GET - POST - Descargas en claro - Scripts PowerShell

### Kerberos (0.1%)

Autenticación en dominio Windows: - AS-REQ / AS-REP - TGS-REQ / TGS-REP

### LDAP / NetBIOS

Comunicación interna con Active Directory.

------------------------------------------------------------------------

## 4. Hipótesis inicial

1.  Vector inicial vía HTTP/HTTPS
2.  Descarga de payload
3.  Ejecución
4.  Persistencia
5.  Uso de Kerberos tras compromiso

------------------------------------------------------------------------

## 5. Filtrado HTTP

Filtro aplicado: \_ws.col.protocol == "HTTP"

Observaciones:

-   GET connecttest.txt → 200 OK
-   GET /api/file/get-file/264872
-   GET /api/file/get-file/29842.ps1

Descarga de script PowerShell sospechoso.

------------------------------------------------------------------------

## 6. Exportación de objetos HTTP

File → Export Objects → HTTP

Archivos relevantes:

-   pas.ps1
-   TeamViewer
-   Teamviewer_Resource_fr
-   TV
-   Otros recursos .aspx

------------------------------------------------------------------------

## 7. Análisis del PowerShell (pas.ps1)

Uso de: iex
(\[System.Text.Encoding\]::UTF8.GetString(...FromBase64String(...)))

Indica: - Ofuscación - Ejecución en memoria - Stager inicial

------------------------------------------------------------------------

## 8. Beaconing y polling C2

GET repetidos a:

/1517096937

Múltiples 404 → finalmente 200 OK

Esto indica polling al servidor C2 hasta que el payload está disponible.

------------------------------------------------------------------------

## 9. Cadena de infección (Multi‑Stage Loader)

### Fase 1 -- Stager PowerShell

-   Descarga TeamViewer.exe
-   Descarga DLLs
-   Crea carpeta: C:`\ProgramData`{=tex}`\huo`{=tex}

### Fase 2 -- Loader intermedio (TeamViewer)

Software legítimo usado como intermediario. Evade detección basada en
reputación.

### Fase 3 -- Recurso intermedio

Descarga desde IP externa (ej. 92.12.53.4).

### Fase 4 -- Payload real (TV)

Descargado desde IP externa (ej. 5.7.14.53). Detectado como malware en
VirusTotal.

------------------------------------------------------------------------

## 10. Persistencia

Creación de acceso directo:

Startup`\TeamViewer`{=tex}.lnk

Permite ejecución automática tras reinicio.

------------------------------------------------------------------------

## 11. Comunicación C2

Función Send-Log envía resultado al atacante mediante HTTP.

------------------------------------------------------------------------

## 12. Identificación del cliente infectado

Filtro: tcp.dstport == 88

IP que realiza TGS-REQ: 10.1.17.215

------------------------------------------------------------------------

## 13. Dirección MAC

00:d0:b7:26:4a:74

------------------------------------------------------------------------

## 14. Hostname

DESKTOP-L8C5GSJ

------------------------------------------------------------------------

## 15. Usuario autenticado

Campo cname-string:

shutchenson

------------------------------------------------------------------------

## 16. Dominio falso Google Authenticator

Filtro DNS: dns.qry.name ==
"google-authenticator.burleson-appliance.net"

Dominio malicioso: google-authenticator.burleson-appliance.net

Técnica: Typosquatting.

------------------------------------------------------------------------

## 17. Respuestas finales

IP infectada: 10.1.17.215

MAC: 00:d0:b7:26:4a:74

Hostname: DESKTOP-L8C5GSJ

Usuario: shutchenson

Dominio phishing: google-authenticator.burleson-appliance.net

------------------------------------------------------------------------

## 18. Conclusión técnica

El incidente muestra un ataque multi‑stage:

1.  Initial Access vía HTTP
2.  PowerShell stager ofuscado
3.  Descarga de loader legítimo (TeamViewer)
4.  Descarga de payload real (TV)
5.  Persistencia en Startup
6.  Comunicación C2
7.  Activación tras reinicio

Se confirma compromiso del host 10.1.17.215.
