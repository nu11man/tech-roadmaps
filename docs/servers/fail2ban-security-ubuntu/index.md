---
layout: default
title: Configuración y gestión de *fail2ban* en servidores Ubuntu
---

# Configuración y Gestión de Fail2Ban en Ubuntu

En nuestra entrada anterior estuvimos aprendiendo como instalar, configurar y usar el firewall _UFW_ en nuestro servidor Ubuntu con el fin de prevenir accesos indeseados a servicios o puertosa los que no nos interesa que se realicen conexiones desde internet. Sin embargo, prevenir las conexiones es solo una parte de las prácticas de seguridad, la otra parte es prevenir ataques de fuerza bruta en los puntos donde si esperamos conexiones.

Pensemos en un escenario clásico, tenemos un puerto en donde esperamos realizar conexiones SSH con el fin de que el administrador del sistema realice sus gestiones cotidianas, por lo tanto, ese puerto debe estar abierto a internet, pero también hay múltiples atacantes, bots, enviando miles de intentos de conexión probando combinaciones de usuario/contraseña.

Para contrarrestar el impacto de un escenario como el anterior hoy vamos a instalar, configurar y aprender a usar una herramienta clásica llamada _fail2ban_ que se encarga de monitorear constantemente los logs en busca de patrones de intentos fallidos de conexión, entre otros y aplicar reglas al firewall con el fin de "encarcelar" las direcciones IP desde donde provienen esos ataques o directamente "bloquear" conexiones entrantes según las reglas que hayamos establecido.

#### Contenido

- [Instalación de fail2ban](#instalacion-fail2ban)
- [Configuración de reglas](#configuracion-de-reglas)

---

### Instalación de fail2ban {#instalacion-fail2ban}

### Configuración de reglas {#configuracion-de-reglas}

Esto fue todo por ahora, en el próximo artículo estaremos dando un vistazo a todo lo que necesitamos saber sobre uno de los servidores web más versátiles y más utilizados en la industria, estoy hablando de **Nginx**. Nos vemos en el próximo artículo.

---

Autor: Julio César Echeverri Marulanda
