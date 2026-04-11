# server-monitoring
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white)
![Oracle Cloud](https://img.shields.io/badge/Oracle_Cloud-F80000?logo=oracle&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_20.04-E95420?logo=ubuntu&logoColor=white)

🌐 [English](README.md)
🌐 [Español](README.es.md)

Un stack de monitorización que he montado para aprender cómo funcionan Prometheus y Grafana en la práctica, usando un servidor real en Oracle Cloud que ya tenía en marcha.

## Contexto

Soy un estudiante de ingeniería de telecomunicaciones metiéndome en el mundo de las infraestructuras. Quería trabajar con herramientas que resuelven problemas reales. Además, hice algo de investigación por mi cuenta para entender cómo funciona Prometheus y Grafana, más allá de simplemente leer documentación — así que usé una VM en Oracle Cloud que ya tenía (que corre un servidor de Minecraft) como objetivo a monitorizar. Al final el servidor de Minecraft es la excusa para aprender Docker Compose en condiciones.

## Qué hace

![Minecraft dashboard](https://github.com/user-attachments/assets/4e5a746c-a2e7-4da9-92ea-09e703209f57)
*Minecraft server dashboard — TPS, MSPT, players and entities in real time*


![Node Exporter dashboard](https://github.com/user-attachments/assets/a5462a0a-d338-4d81-badc-351bc6fd78aa)
*System dashboard — CPU, RAM, network and disk*

- Recoge métricas del sistema (CPU, RAM, disco, red) desde el host usando Node Exporter
- Recoge métricas específicas del servidor de Minecraft (TPS, MSPT, jugadores online, entidades cargadas) usando FabricExporter + Spark
- Almacena todo en Prometheus
- Lo visualiza en Grafana con dos dashboards

## Stack

- **Prometheus** — recoge y almacena las métricas
- **Grafana** — dashboards y visualización
- **Node Exporter** — métricas a nivel de sistema
- **FabricExporter** — métricas del servidor de Minecraft (requiere el mod Spark)
- **Docker Compose** — lo une todo
- **Oracle Cloud** — donde corre (Ubuntu 20.04, ARM)

## Arquitectura

    Browser
      │
      └── :3000  →  Grafana
                      │
                  Prometheus  ←──  Node Exporter  (métricas del sistema)
                      │
                      └──────────  FabricExporter  (métricas de Minecraft, via Crafty Controller)

Prometheus y Node Exporter se comunican internamente a través de la red de Docker — ningún puerto extra expuesto al exterior.

## Instalación

### Requisitos previos

- Docker y Docker Compose instalados
- Un servidor de Minecraft con Fabric (si quieres el dashboard de Minecraft)
- Puerto 3000 abierto en el firewall (interfaz de Grafana)

### Arrancarlo

    git clone https://github.com/peristack/server-monitoring
    cd server-monitoring
    docker compose up -d

Grafana estará disponible en `http://ip-de-tu-servidor:3000`.
Credenciales por defecto: `admin / admin` — **recomiendo cambiarlas inmediatamente y si es posible, activar 2FA o algo similar**.

### Métricas de Minecraft (opcional)

Instala estos mods en tu servidor Fabric:

- [Spark](https://modrinth.com/mod/spark) — profiler de rendimiento, proporciona TPS y MSPT
- [FabricExporter](https://modrinth.com/mod/fabricexporter) — expone métricas en formato Prometheus

FabricExporter escucha en el puerto 25585 por defecto. Actualiza `prometheus/prometheus.yml` con la dirección de tu servidor.

### Dashboards

Impórtalos desde la librería de dashboards de Grafana:

- **Node Exporter Full** — ID `1860` (métricas del sistema)
- **Minecraft Fabric** — ID `14492` (métricas del juego)

## Qué he aprendido

- Cómo funciona el scraping de Prometheus y cómo configurar los targets
- Cómo clasifica Prometheus los diferentes tipos de métricas y por qué importa
- Cómo conecta Grafana con fuentes de datos y cómo funcionan las variables de los dashboards
- Redes Docker entre contenedores (por qué `container_name:port` funciona internamente)
- Seguridad básica en cloud — mantener los servicios internos fuera de los puertos públicos

## Notas

Este es un proyecto de aprendizaje, no está preparado para un entorno real. Algunas cosas que mejoraría con más tiempo:

- Añadir AlertManager para recibir notificaciones cuando el TPS baje o el disco se llene
- Configurar autenticación más robusta más allá del login básico de Grafana
- Usar variables de entorno para los secretos en lugar de ponerlos directamente en los ficheros de configuración
