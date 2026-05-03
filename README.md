---

## 🚀 Servicios

| Servicio | Imagen | Réplicas | Puerto |
|---|---|---|---|
| frontend | nginx:alpine | 5 | 80 |
| backend | node:18-alpine | 2 | - |
| database | mysql:8 | 1 | - |
| cache | redis:7-alpine | 1 | - |
| visualizer | dockersamples/visualizer | 1 | 8080 |

---

## ⚙️ Requisitos

- Docker Desktop 29.x o superior
- WSL2 (Ubuntu) o Linux
- 4GB RAM mínimo

---

## 🛠️ Instalación y Despliegue

### 1. Crear la red y los nodos simulados

```bash
docker network create --driver bridge swarm-net

docker run -d --privileged \
  --name manager \
  --network swarm-net \
  --hostname manager \
  -p 8080:8080 -p 80:80 \
  docker:dind

docker run -d --privileged \
  --name worker1 \
  --network swarm-net \
  --hostname worker1 \
  docker:dind

docker run -d --privileged \
  --name worker2 \
  --network swarm-net \
  --hostname worker2 \
  docker:dind
```

### 2. Inicializar el Swarm

```bash
docker exec -it manager sh
ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1
docker swarm init --advertise-addr 172.18.0.2
```

### 3. Unir los Workers

```bash
docker exec -it worker1 sh
docker swarm join --token SWMTKN-1-xxxx 172.18.0.2:2377

docker exec -it worker2 sh
docker swarm join --token SWMTKN-1-xxxx 172.18.0.2:2377
```

### 4. Verificar el clúster

```bash
docker node ls
```

### 5. Crear el Docker Secret

```bash
echo "MiPasswordSegura123" | docker secret create db_password -
docker secret ls
```

### 6. Desplegar el stack

```bash
docker cp docker-compose.yml manager:/docker-compose.yml
docker exec -it manager sh
docker stack deploy -c docker-compose.yml techretail
docker stack services techretail
```

---

## 📈 Escalado Dinámico

```bash
docker service scale techretail_frontend=5
docker stack services techretail
```

---

## 🌐 Acceso

| Servicio | URL |
|---|---|
| Frontend | http://localhost |
| Visualizer | http://localhost:8080 |

---

## 🔐 Docker Secrets

```bash
echo "MiPasswordSegura123" | docker secret create db_password -
```

Las credenciales se montan en `/run/secrets/db_password` dentro del contenedor.

---

## ♻️ Gestión del clúster

```bash
# Detener nodos
docker stop manager worker1 worker2

# Reiniciar nodos
docker start manager worker1 worker2

# Eliminar el stack
docker stack rm techretail

# Eliminar todo
docker rm -f manager worker1 worker2
docker network rm swarm-net
```

---

## 📊 Criterios Cumplidos

| Criterio | Puntaje | Estado |
|---|---|---|
| Configuración correcta del clúster Swarm | 20% | ✅ |
| Despliegue funcional de todos los servicios | 25% | ✅ |
| Implementación de réplicas y escalado | 20% | ✅ |
| Uso correcto de Secrets y Configs | 10% | ✅ |
| Informe técnico y documentación | 15% | ✅ |
| Video demostrativo | 10% | ✅ |

---

## 👤 Autor
Carlos Valeriano Colan
