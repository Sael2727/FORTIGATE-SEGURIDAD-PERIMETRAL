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
| Dominio bloqueado | Motivo |
|:-----------------:|--------|
| facebook.com | Red social |
| instagram.com | Red social |
| tiktok.com | Red social |
| x.com / twitter.com | Red social |
| whatsapp.com / whatsapp.net / wa.me | Mensajería |
| itla.edu.do y subdominios | Dominio institucional |

### DNS Filter — `DNS_BLOQUEOS`
Redirige al portal de bloqueo: `itla.edu.do`, `facebook.com`, `instagram.com`, `tiktok.com`, `whatsapp.com`, `whatsapp.net`, `wa.me` y sus wildcards.

### Application Control — `APP_BLOCK_WHATSAPP`
Bloquea: `WhatsApp`, `WhatsApp_Web`, `WhatsApp_VoIPCall`, `WhatsApp_File_Transfer`.

### Intrusion Prevention — `IPS_BLOCK_SCANNERS`
Bloquea firmas de severidad **media, alta y crítica** para escaneos de red y comportamientos maliciosos.

### Web Application Firewall — `WAF_WEB_SERVER`
Protege `10.7.25.130` contra: `Cross Site Scripting (XSS)`, `SQL Injection`, `Generic Attacks`, `Trojans`, `Known Exploits`. Incluye constraints de host, versión HTTP, métodos y tamaño de parámetros.

---

## ✅ Resultados de las Pruebas

| Prueba | Resultado |
|:------:|:---------:|
| Cliente-Linux recibe IP `10.7.25.10/25` por DHCP | ✅ |
| Servidor-Web IP estática `10.7.25.130/28` con Python HTTP activo | ✅ |
| `wget http://10.7.25.130` → HTTP 200 OK | ✅ Permitido |
| Acceso en navegador a `http://10.7.25.130` | ✅ Visible |
| Ping al servidor `10.7.25.130` desde LAN | ✅ Bloqueado |
| Ping a `8.8.8.8` desde LAN | ✅ Exitoso (NAT funciona) |
| Bloqueo de `itla.edu.do` | ✅ Página de bloqueo FortiGate |
| Bloqueo de `facebook.com` | ✅ Página de bloqueo FortiGate |
| Bloqueo de `instagram.com` | ✅ Página de bloqueo FortiGate |
| Log HTTP permitido hacia servidor (USERS_TO_WEB_HTTP_ONLY) | ✅ Visible en Forward Traffic |
| Log UTM Blocked para instagram.com | ✅ Visible en Forward Traffic |
| Escaneo Nmap: solo puerto 80 abierto, resto filtrado | ✅ |

---

## 📁 Archivos del Repositorio

| Archivo | Descripción |
|:-------:|-------------|
| [`SaelGerman_2025-0725_FortiGate_P2.pdf`](SaelGerman_2025-0725_FortiGate_P2.pdf) | Documentación técnica completa |

---

## 🖼️ Capturas de Pantalla

### Topología y Feature Visibility
- 📸 [Captura 1 — Topología general de seguridad perimetral](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/01_Topologia_general.png)
- 📸 [Captura 2 — Feature Visibility: funciones principales](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/02_Feature_Visibility_funciones_principales.png)
- 📸 [Captura 3 — Feature Visibility: opciones adicionales](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/03_Feature_Visibility_opciones_adicionales.png)

### Interfaces, DHCP, Ruta y Políticas
- 📸 [Captura 4 — Interfaces configuradas en FortiGate](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/04_Interfaces_configuradas_FortiGate.png)
- 📸 [Captura 5 — DHCP Server en LAN_USUARIOS](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/05_DHCP_Server_LAN_USUARIOS.png)
- 📸 [Captura 6 — Ruta por defecto](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/06_Ruta_por_defecto.png)
- 📸 [Captura 7 — Políticas firewall principales](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/07_Politicas_firewall_principales.png)

### Web Filter y DNS Filter
- 📸 [Captura 8 — Web Filter: redes sociales](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/08_Web_Filter_redes_sociales.png)
- 📸 [Captura 9 — Web Filter: WhatsApp e ITLA](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/09_Web_Filter_WhatsApp_ITLA.png)
- 📸 [Captura 10 — Web Filter: confirmación de entradas](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/10_Web_Filter_confirmacion_entradas.png)
- 📸 [Captura 11 — DNS Filter: dominios bloqueados](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/11_DNS_Filter_dominios_bloqueados.png)
- 📸 [Captura 12 — DNS Filter: WhatsApp y wa.me](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/12_DNS_Filter_WhatsApp_wa_me.png)

### Application Control, IPS y WAF
- 📸 [Captura 13 — Application Control: perfil APP_BLOCK_WHATSAPP](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/13_Application_Control_perfil_WhatsApp.png)
- 📸 [Captura 14 — Application Control: aplicaciones bloqueadas](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/14_Application_Control_aplicaciones_bloqueadas.png)
- 📸 [Captura 15 — Intrusion Prevention: perfil IPS_BLOCK_SCANNERS](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/15_Intrusion_Prevention_perfil_IPS.png)
- 📸 [Captura 16 — IPS_BLOCK_SCANNERS: severidades bloqueadas](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/16_IPS_BLOCK_SCANNERS_severidades_bloqueadas.png)
- 📸 [Captura 17 — WAF: perfil WAF_WEB_SERVER creado](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/17_WAF_WEB_SERVER_perfil_creado.png)
- 📸 [Captura 18 — WAF: firmas de protección](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/18_WAF_firmas_de_proteccion.png)
- 📸 [Captura 19 — WAF: restricciones y métodos HTTP](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/19_WAF_restricciones_metodos_HTTP.png)

### Políticas con Perfiles Aplicados
- 📸 [Captura 20 — Política HTTP: origen, destino y acción](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/20_Politica_HTTP_origen_destino_accion.png)
- 📸 [Captura 21 — Política HTTP: IPS y WAF aplicados](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/21_Politica_HTTP_IPS_WAF_aplicados.png)
- 📸 [Captura 22 — Política de Internet: origen y NAT](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/22_Politica_Internet_origen_NAT.png)
- 📸 [Captura 23 — Política de Internet: perfiles UTM](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/23_Politica_Internet_perfiles_UTM.png)

### Pruebas en Cliente y Servidor
- 📸 [Captura 24 — Cliente-Linux: IP por DHCP](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/24_Cliente_Linux_IP_por_DHCP.png)
- 📸 [Captura 25 — Servidor-Web: configuración de red y servicio HTTP](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/25_Servidor_Web_configuracion_red_servicio_HTTP.png)
- 📸 [Captura 26 — Prueba HTTP con wget (HTTP 200 OK)](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/26_Prueba_HTTP_wget_servidor_web.png)
- 📸 [Captura 27 — Prueba HTTP en navegador](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/27_Prueba_HTTP_navegador.png)
- 📸 [Captura 28 — Ping al servidor bloqueado e Internet/NAT funcionando](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/28_Prueba_ping_bloqueado_Internet.png)

### Bloqueos y Logs
- 📸 [Captura 29 — Bloqueo de itla.edu.do](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/29_Bloqueo_itla_edu_do.png)
- 📸 [Captura 30 — Bloqueo de facebook.com](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/30_Bloqueo_facebook_com.png)
- 📸 [Captura 31 — Bloqueo de instagram.com](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/31_Bloqueo_instagram_com.png)
- 📸 [Captura 32 — Log de HTTP permitido hacia servidor web](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/32_Log_HTTP_permitido_servidor_web.png)
- 📸 [Captura 33 — Logs de bloqueo de instagram.com por UTM](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/33_Logs_bloqueo_instagram_UTM.png)

### Prueba de Escaneo de Puertos
- 📸 [Captura 34 — Escaneo Nmap: solo puerto 80 abierto, resto filtrado](SaelGerman_2025-0725_Fotos_Documento_FortiGate_P2_GitHub/34_Prueba_escaneo_puertos_Nmap.png)

---

## 📎 Recursos

📄 **Documentación Técnica:** [Ver Informe PDF](SaelGerman_2025-0725_FortiGate_P2.pdf)  
▶️ **Video Demostración:** [Ver en YouTube](https://youtu.be/iqo2pbR7nd0?si=jOpn6T1zWkGmImRy)

---

## 📚 Referencias

1. Fortinet. *FortiGate Administration Guide — Security Profiles*. Documentación oficial FortiOS.
2. Reconocimiento especial: Troubleshooting y documentación apoyado en Inteligencia Artificial.

---

<div align="center">

*Este laboratorio fue desarrollado exclusivamente con fines académicos y educativos.*

</div>
