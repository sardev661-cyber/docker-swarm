
# 🐳 TechRetail — Docker Swarm Deployment

> Trabajo 1: Orquestación de contenedores con Docker Swarm
> Entorno: WSL2 + Docker Desktop 29.4.1

## 📋 Descripción

Implementación de un clúster **Docker Swarm** de 3 nodos (1 Manager + 2 Workers) para la empresa ficticia **TechRetail**, una plataforma de comercio electrónico que requiere alta disponibilidad y escalabilidad.

Se desplegaron 5 microservicios orquestados con Docker Swarm usando contenedores Docker-in-Docker (DinD) sobre WSL2.

## 🏗️ Arquitectura

    ┌─────────────────────────────────────┐
    │         Docker Swarm Cluster        │
    │  ┌────────────┐                     │
    │  │  MANAGER   │ Leader 172.18.0.2   │
    │  │  database  │                     │
    │  │  frontend  │                     │
    │  │  backend   │                     │
    │  └────────────┘                     │
    │  ┌────────────┐  ┌────────────┐     │
    │  │  WORKER 1  │  │  WORKER 2  │     │
    │  │ frontend x2│  │ frontend x2│     │
    │  │  backend   │  │   cache    │     │
    │  └────────────┘  └────────────┘     │
    └─────────────────────────────────────┘

## 🚀 Servicios

| Servicio | Imagen | Réplicas | Puerto |
|---|---|---|---|
| frontend | nginx:alpine | 5 | 80 |
| backend | node:18-alpine | 2 | - |
| database | mysql:8 | 1 | - |
| cache | redis:7-alpine | 1 | - |
| visualizer | dockersamples/visualizer | 1 | 8080 |

## ⚙️ Requisitos

- Docker Desktop 29.x o superior
- WSL2 (Ubuntu) o Linux
- 4GB RAM mínimo

## 🛠️ Instalación y Despliegue

**1. Crear la red y los nodos**

    docker network create --driver bridge swarm-net
    docker run -d --privileged --name manager --network swarm-net -p 8080:8080 -p 80:80 docker:dind
    docker run -d --privileged --name worker1 --network swarm-net docker:dind
    docker run -d --privileged --name worker2 --network swarm-net docker:dind

**2. Inicializar el Swarm**

    docker exec -it manager sh
    ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1
    docker swarm init --advertise-addr 172.18.0.2

**3. Unir los Workers**

    docker swarm join --token SWMTKN-1-xxxx 172.18.0.2:2377

**4. Crear el Docker Secret**

    echo "MiPasswordSegura123" | docker secret create db_password -

**5. Desplegar el stack**

    docker cp docker-compose.yml manager:/docker-compose.yml
    docker stack deploy -c docker-compose.yml techretail
    docker stack services techretail

## 📈 Escalado Dinámico

    docker service scale techretail_frontend=5
    docker stack services techretail

## 🌐 Acceso

| Servicio | URL |
|---|---|
| Frontend | http://localhost |
| Visualizer | http://localhost:8080 |

## 🔐 Docker Secrets

Las credenciales se gestionan como secrets cifrados y se montan en `/run/secrets/db_password` dentro del contenedor.

## ♻️ Gestión del clúster

    docker stop manager worker1 worker2
    docker start manager worker1 worker2
    docker stack rm techretail
    docker rm -f manager worker1 worker2
    docker network rm swarm-net

## 📊 Criterios Cumplidos

| Criterio | Puntaje | Estado |
|---|---|---|
| Configuración correcta del clúster Swarm | 20% | ✅ |
| Despliegue funcional de todos los servicios | 25% | ✅ |
| Implementación de réplicas y escalado | 20% | ✅ |
| Uso correcto de Secrets y Configs | 10% | ✅ |
| Informe técnico y documentación | 15% | ✅ |
| Video demostrativo | 10% | ✅ |

## 👤 Autor
Carlos Valeriano Colan
---

