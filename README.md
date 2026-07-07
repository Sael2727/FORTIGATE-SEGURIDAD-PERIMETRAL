# 🛡️ FortiGate — Seguridad Perimetral

<div align="center">

![FortiGate](https://img.shields.io/badge/FortiGate-FortiOS-red?style=for-the-badge&logo=fortinet)
![WebFilter](https://img.shields.io/badge/UTM-Web%20Filter-blue?style=for-the-badge)
![IPS](https://img.shields.io/badge/UTM-IPS-orange?style=for-the-badge)
![WAF](https://img.shields.io/badge/UTM-WAF-green?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-PNETLab-purple?style=for-the-badge)
![License](https://img.shields.io/badge/Uso-Educativo-gray?style=for-the-badge)

**Sael Germán García** | Matrícula: `2025-0725`  
Asignatura: Seguridad de Redes | Profesor: Jonathan Rondón  
Instituto Tecnológico de las Américas — ITLA | 2026

</div>

---

## 📋 Descripción

Implementación de una **topología de seguridad perimetral en FortiGate** que separa una LAN de usuarios y una LAN de servidores, habilita acceso a Internet mediante NAT, asigna direcciones IP por DHCP y aplica controles de seguridad avanzados mediante perfiles UTM para filtrar tráfico no autorizado, bloquear aplicaciones y proteger el servidor web.

---

## 🗺️ Topología de Red

FortiGate actúa como firewall perimetral con tres zonas: WAN hacia Internet, LAN de usuarios y LAN de servidores.

### 📊 Direccionamiento IP

| Dispositivo | Interfaz | IP / Máscara | Función |
|:-----------:|:--------:|:-------------:|---------|
| FortiGate | port1 | 192.168.226.211/24 | WAN_INTERNET |
| FortiGate | port2 | 10.7.25.1/25 | LAN_USUARIOS (DHCP) |
| FortiGate | port3 | 10.7.25.129/28 | LAN_SERVIDORES |
| Cliente-Linux | ens3 | 10.7.25.10/25 | Asignado por DHCP |
| Servidor-Web | ens3 | 10.7.25.130/28 | Servidor HTTP |

---

## 📋 Requerimientos Implementados

| Requerimiento | Estado |
|:-------------:|:------:|
| LAN de usuarios con /25 y DHCP habilitado | ✅ |
| LAN de servidores con /28 | ✅ |
| Ruta por defecto hacia Internet | ✅ |
| NAT para salida de usuarios a Internet | ✅ |
| Solo HTTP desde LAN_USUARIOS hacia servidor web | ✅ |
| Bloquear todo lo demás hacia LAN de servidores | ✅ |
| Bloquear redes sociales (Facebook, Instagram, TikTok, X) | ✅ |
| Bloquear WhatsApp (web, VoIP, transferencia de archivos) | ✅ |
| Bloquear dominios y subdominios de itla.edu.do | ✅ |
| Detectar y bloquear escaneos de red (IPS) | ✅ |
| WAF aplicado al servidor web | ✅ |

---

## 🔥 Políticas Firewall

| ID | Política | Origen | Destino | Servicio | Acción |
|:--:|:--------:|:------:|:-------:|:--------:|--------|
| 1 | `LAN_USERS_TO_INTERNET` | LAN_USUARIOS | WAN_INTERNET | ALL | Accept + NAT + UTM |
| 2 | `USERS_TO_WEB_HTTP_ONLY` | LAN_USUARIOS | SERVIDOR_WEB | HTTP | Accept + IPS + WAF |
| 3 | `BLOCK_USERS_TO_SERVERS` | LAN_USUARIOS | LAN_SERVIDORES | ALL | Deny |

---

## 🛡️ Perfiles de Seguridad UTM

### Web Filter — `WF_BLOQUEOS`
Bloquea acceso a dominios mediante categorías y entradas estáticas:

| Dominio bloqueado | Motivo |
|:-----------------:|--------|
| facebook.com | Red social |
| instagram.com | Red social |
| tiktok.com | Red social |
| x.com / twitter.com | Red social |
| whatsapp.com / whatsapp.net / wa.me | Mensajería |
| itla.edu.do y subdominios | Dominio institucional |

### DNS Filter — `DNS_BLOQUEOS`
Redirige al portal de bloqueo los dominios y subdominios:
`itla.edu.do`, `facebook.com`, `instagram.com`, `tiktok.com`, `whatsapp.com`, `whatsapp.net`, `wa.me` y sus wildcards.

### Application Control — `APP_BLOCK_WHATSAPP`
Bloquea a nivel de aplicación:
`WhatsApp`, `WhatsApp_Web`, `WhatsApp_VoIPCall`, `WhatsApp_File_Transfer`.

### Intrusion Prevention — `IPS_BLOCK_SCANNERS`
Detecta y bloquea firmas de severidad **media, alta y crítica** para escaneos de red y comportamientos maliciosos.

### Web Application Firewall — `WAF_WEB_SERVER`
Protege el servidor web `10.7.25.130` contra:
`Cross Site Scripting (XSS)`, `SQL Injection`, `Generic Attacks`, `Trojans`, `Known Exploits`.
Incluye constraints de host, versión HTTP, métodos, tamaño de cabeceras y parámetros.

---

## ✅ Resultados de las Pruebas

| Prueba | Resultado |
|:------:|:---------:|
| Cliente-Linux recibe IP `10.7.25.10/25` por DHCP | ✅ |
| Servidor-Web tiene IP estática `10.7.25.130/28` | ✅ |
| Acceso HTTP `http://10.7.25.130` → HTTP 200 OK | ✅ Permitido |
| `wget http://10.7.25.130` desde Cliente-Linux | ✅ HTTP 200 OK |
| Acceso en navegador a `http://10.7.25.130` | ✅ Directory listing visible |
| Ping al servidor `10.7.25.130` desde LAN | ✅ Bloqueado (solo HTTP permitido) |
| Ping a `8.8.8.8` desde LAN | ✅ Exitoso (NAT funciona) |
| Acceso a `itla.edu.do` | ✅ Bloqueado por Web/DNS Filter |
| Acceso a `facebook.com` | ✅ Bloqueado por Web Filter |
| Acceso a `instagram.com` | ✅ Bloqueado por Web Filter |
| Escaneo Nmap desde LAN hacia servidor web | ✅ Solo puerto 80/HTTP abierto; FTP, SSH, Telnet, HTTPS, MySQL, RDP y 8080 filtrados |
| Logs Forward Traffic | ✅ HTTP Accept + UTM Blocked evidenciados |

---

## 📁 Archivos del Repositorio

| Archivo | Descripción |
|:-------:|-------------|
| [`SaelGerman_2025-0725_FortiGate_P2.pdf`](SaelGerman_2025-0725_FortiGate_P2.pdf) | Documentación técnica completa |

---

## 🖼️ Capturas de Pantalla

### Topología y Feature Visibility
- 📸 [Captura 1 — Topología general de seguridad perimetral](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/01_Topologia_Seguridad_Perimetral.png)
- 📸 [Captura 2 — Feature Visibility: funciones principales](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/02_Feature_Visibility_Parte_1.png)
- 📸 [Captura 3 — Feature Visibility: opciones adicionales](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/03_Feature_Visibility_Parte_2.png)

### Interfaces, DHCP, Ruta y Políticas
- 📸 [Captura 4 — Interfaces configuradas en FortiGate](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/04_Interfaces_FortiGate.png)
- 📸 [Captura 5 — DHCP Server en LAN_USUARIOS](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/05_DHCP_LAN_USUARIOS.png)
- 📸 [Captura 6 — Ruta por defecto](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/06_Ruta_Default.png)
- 📸 [Captura 7 — Políticas firewall principales](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/07_Firewall_Policies.png)

### Web Filter y DNS Filter
- 📸 [Captura 8 — Web Filter: redes sociales (Parte 1)](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/08_Web_Filter_Redes_Sociales_Parte_1.png)
- 📸 [Captura 9 — Web Filter: WhatsApp e ITLA](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/09_Web_Filter_WhatsApp_ITLA.png)
- 📸 [Captura 10 — Web Filter: confirmación de entradas](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/10_Web_Filter_Tiktok_WhatsApp.png)
- 📸 [Captura 11 — DNS Filter: dominios bloqueados (Parte 1)](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/11_DNS_Filter_Dominios_Parte_1.png)
- 📸 [Captura 12 — DNS Filter: WhatsApp y wa.me (Parte 2)](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/12_DNS_Filter_WhatsApp_Parte_2.png)

### Application Control, IPS y WAF
- 📸 [Captura 13 — Application Control: perfil APP_BLOCK_WHATSAPP](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/13_Application_Control_Perfil.png)
- 📸 [Captura 14 — Application Control: aplicaciones bloqueadas](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/14_Application_Control_WhatsApp_Blocked_Apps.png)
- 📸 [Captura 15 — Intrusion Prevention: perfil IPS_BLOCK_SCANNERS](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/15_IPS_Profile_List.png)
- 📸 [Captura 16 — IPS_BLOCK_SCANNERS: severidades bloqueadas](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/16_IPS_BLOCK_SCANNERS.png)
- 📸 [Captura 17 — WAF: perfil WAF_WEB_SERVER creado](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/17_WAF_Profile_List.png)
- 📸 [Captura 18 — WAF: firmas de protección](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/18_WAF_WEB_SERVER_Signatures.png)
- 📸 [Captura 19 — WAF: restricciones y métodos HTTP](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/19_WAF_WEB_SERVER_Constraints.png)

### Políticas con Perfiles Aplicados
- 📸 [Captura 20 — Política HTTP: origen, destino y acción](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/20_Policy_USERS_TO_WEB_HTTP_ONLY_General.png)
- 📸 [Captura 21 — Política HTTP: IPS y WAF aplicados](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/21_Policy_USERS_TO_WEB_HTTP_ONLY_WAF_IPS.png)
- 📸 [Captura 22 — Política de Internet: origen y NAT](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/22_Policy_LAN_USERS_TO_INTERNET_General.png)
- 📸 [Captura 23 — Política de Internet: perfiles UTM](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/23_Policy_LAN_USERS_TO_INTERNET_Security_Profiles.png)

### Pruebas en Cliente y Servidor
- 📸 [Captura 24 — Cliente-Linux: IP por DHCP y ruta](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/24_Cliente_Linux_DHCP_IP_Route.png)
- 📸 [Captura 25 — Servidor-Web: configuración de red y servicio HTTP](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/25_Servidor_Web_IP_Route.png)
- 📸 [Captura 26 — Servidor-Web: IP estática y Python HTTP server](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/26_Servidor_Web_HTTP_Server.png)
- 📸 [Captura 27 — Prueba HTTP con wget (HTTP 200 OK)](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/27_Prueba_HTTP_wget.png)
- 📸 [Captura 28 — Prueba HTTP en navegador (Directory listing)](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/28_Prueba_HTTP_Navegador.png)
- 📸 [Captura 29 — Ping bloqueado al servidor e Internet/NAT OK](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/29_Prueba_Ping_Bloqueado_Internet_OK.png)

### Prueba de Escaneo de Puertos
- 📸 [Captura 30 — Escaneo Nmap: solo puerto 80 abierto, resto filtrado](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/30_escaneodepuertos%20.png)

### Bloqueos y Logs
- 📸 [Captura 31 — Bloqueo de facebook.com](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/31_Bloqueo_Facebook.png)
- 📸 [Captura 32 — Bloqueo de instagram.com](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/32_Bloqueo_Instagram.png)
- 📸 [Captura 33 — Bloqueo de itla.edu.do](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/33_Bloqueo_ITLA.png)
- 📸 [Captura 34 — Logs de tráfico (HTTP Accept + UTM Blocked)](SaelGerman_2025-0725_Capturas/SaelGerman_2025-0725_Imagenes_P1/34_Logs_HTTP_Accept_Deny.png)

---

## 📎 Recursos

📄 **Documentación Técnica:** [Ver Informe PDF](SaelGerman_2025-0725_FortiGate_P2.pdf)  
▶️ **Video Demostración:** [Ver en YouTube](https://youtu.be/iqo2pbR7nd0)

---

## 📚 Referencias

1. Fortinet. *FortiGate Administration Guide — Security Profiles*. Documentación oficial FortiOS.
2. Reconocimiento especial: Troubleshooting y documentación apoyado en Inteligencia Artificial.

---

<div align="center">

*Este laboratorio fue desarrollado exclusivamente con fines académicos y educativos.*

</div>
