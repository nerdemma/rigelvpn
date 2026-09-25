# 🌌 RigelVPN - Layer 3 UDP VPN in C with AES-256-GCM

![C Language](https://img.shields.io/badge/Language-C11-00599C?style=for-the-badge&logo=c&logoColor=white)
![Linux](https://img.shields.io/badge/Platform-Linux%20Debian%2013-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![OpenSSL](https://img.shields.io/badge/Crypto-OpenSSL%20EVP-721412?style=for-the-badge&logo=openssl&logoColor=white)
![Network](https://img.shields.io/badge/Protocol-UDP%20%2F%20TUN-002244?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**RigelVPN** es un prototipo completamente funcional de una **VPN de Capa 3 (L3)** autocontenida y de alto rendimiento, desarrollada en **C puro** para entornos **Linux**. 

El proyecto aprovecha el subsistema virtual `TUN` del Kernel para capturar paquetes IP nativos, sockets `UDP` de baja latencia para el transporte de datos, y la biblioteca **OpenSSL (EVP API)** para establecer un túnel cifrado y autenticado de extremo a extremo.

Ideal para proteger el tráfico de navegación al conectarse desde redes Wi-Fi públicas no seguras (como las de un café), encapsulando de forma transparente todas las peticiones hacia un servidor de confianza.

---

## 🚀 Características y Tecnologías

- **Redes de Capa 3 (IP):** Uso de interfaces virtuales `TUN` (`tun0` / `tun1`) administradas por software para capturar e inyectar paquetes IP crudos directamente en la capa de red del SO.
- **Transporte UDP de Alta Eficiencia:** Implementación sobre sockets `SOCK_DGRAM` para mitigar el problema del *TCP Meltdown* (colapso de rendimiento causado por la doble encapsulación TCP), garantizando una latencia mínima.
- **Cifrado Autenticado (AEAD):** Cifrado de grado militar utilizando **AES-256-GCM** mediante la API `EVP` de OpenSSL.
  - Cada paquete transmite un **Vector de Inicialización (IV) de 12 bytes** único generado aleatoriamente (`RAND_bytes`).
  - Verificación de integridad mediante un **Tag de autenticación GCM de 16 bytes**, descartando automáticamente cualquier paquete manipulado o corrupto.
- **Manejo E/S No Bloqueante:** Uso de la llamada de sistema `poll()` para monitorear en tiempo real y en un solo hilo tanto el descriptor de archivo de la interfaz TUN como el socket UDP.
- **Aislamiento Avanzado para Pruebas:** Entorno de pruebas automatizado en Bash respaldado por **Linux Network Namespaces** (`netns`), simulando una topología de red real sin interferencias de ruteo local.

---

## 📊 Arquitectura del Sistema

El flujo de información atraviesa el siguiente ciclo cifrado desde la aplicación del cliente hasta la red remota:

```text
[ Cliente (10.0.0.2) ]
    │
    ├── 1. Paquete IP generado por el sistema
    ▼
[ Interfaz Virtual TUN (tun1) ]
    │
    ├── 2. C / read(tun_fd)
    ▼
[ RigelVPN Client Process ]
    │
    ├── 3. vpn_encrypt() ──> [ IV (12B) | Ciphertext | Tag (16B) ]
    ▼
[ Red Pública / Socket UDP (Port 4433) ]  <=== (Tráfico Cifrado AES-256-GCM)
    │
    ├── 4. socket sendto/recvfrom
    ▼
[ RigelVPN Server Process ]
    │
    ├── 5. vpn_decrypt() ──> Verifica Tag de Autenticidad
    ▼
[ Interfaz Virtual TUN (tun0) ]
    │
    ├── 6. C / write(tun_fd) inyecta paquete IP al Kernel
    ▼
[ Servidor (10.0.0.1) / Enrutamiento a Internet ]

🛠️ Requisitos del Sistema

    Sistema Operativo: Linux (Probado y optimizado para Debian 13 / Ubuntu 22.04+).

    Compilador: gcc con soporte C11.

    Librerías: libssl-dev (OpenSSL 3.x).

    Herramientas de Red: iproute2, iptables, iputils-ping.

    Permisos: Privilegios de superusuario (root o sudo) para la creación y manipulación de interfaces TUN.

🔧 Instalación y Compilación

    Clonar el repositorio:
    Bash

''git clone [https://github.com/tu-usuario/rigelvpn.git](https://github.com/tu-usuario/rigelvpn.git)
''cd rigelvpn

Instalar dependencias necesarias (Debian/Ubuntu):
Bash

''sudo apt update
''sudo apt install -y build-essential libssl-dev iproute2

Compilar manualmente:
Bash

    ''# Compilar el Cliente
    ''gcc -Wall -Wextra -o src/client src/client.c -lcrypto

    ''# Compilar el Servidor
    ''gcc -Wall -Wextra -o src/server src/server.c -lcrypto

🧪 Pruebas Automatizadas (Test Suite)

El repositorio incluye un script de prueba automatizado (test_vpn.sh) que no solo compila los binarios, sino que crea un entorno de red totalmente aislado utilizando Network Namespaces. Esto permite probar la VPN en la misma máquina física evitando que el Kernel evite el túnel mediante la tabla de ruteo local.

Para ejecutar las pruebas funcionales:
Bash

''sudo ./test_vpn.sh

El script realizará las siguientes acciones de forma limpia:

    Compilará el cliente y el servidor con sintaxis estricta y -lcrypto.

    Creará dos namespaces aislados (ns_vpn_server y ns_vpn_client).

    Interconectará los namespaces con un par de interfaces virtuales veth (simulando Internet).

    Inicializará el servidor y el cliente estableciendo el handshake UDP inicial.

    Ejecutará pruebas de conectividad bidireccional mediante ping (ICMP) cifradas al 100%.

    Generará un informe detallado en vpn_test_report.txt y limpiará los procesos al finalizar.

📁 Estructura del Proyecto
Plaintext

rigelvpn/
├── lib/
│   └── crypto.h          # Módulo de cifrado/descifrado AES-256-GCM (OpenSSL EVP)
├── src/
│   ├── client.c          # Cliente VPN (Manejo de tun1 y Socket UDP)
│   └── server.c          # Servidor VPN (Manejo de tun0 y múltiples clientes UDP)
├── test_vpn.sh           # Orchestrador de pruebas automatizadas con Namespaces
├── vpn_test_report.txt   # Reporte generado tras la ejecución de pruebas
└── README.md             # Documentación del proyecto

📜 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo LICENSE para más detalles.

Desarrollado como proyecto personal de ingeniería de software para demostrar programación de bajo nivel en C, manipulación de paquetes IP y criptografía aplicada.