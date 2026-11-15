# Experimento 1: Caída del líder en Consul

## Objetivo
Comprobar qué pasa en un clúster de Consul de 3 servers cuando se cae
el líder: si el sistema sigue funcionando, cuánto tarda en elegir otro
líder y cómo se reincorpora el nodo caído.

## Entorno
- Windows, Docker Desktop, Consul 1.20.6
- 3 servers en Docker Compose (server1, server2, server3)

## Comandos usados
```
docker compose up -d
docker exec consul-server1 consul operator raft list-peers
docker stop consul-server2
docker exec consul-server1 consul operator raft list-peers
docker start consul-server2
docker exec consul-server1 consul operator raft list-peers
```

## Salida obtenida
### Antes de la caída
```
Node     ID                                    Address          State     Voter  RaftProtocol  Commit Index  Trails Leader By
server2  632c5c51-33de-cd9f-2f8d-2e16b98563f2  172.20.0.3:8300  leader    true   3             191           -
server1  9a12cc36-97a2-39be-1a4d-43aab65fda95  172.20.0.4:8300  follower  true   3             191           0 commits
server3  43580844-8783-dfce-ae36-eb7dfb549968  172.20.0.2:8300  follower  true   3             191           0 commits
```

### Después de apagar al líder (docker stop consul-server2)
```
Node     ID                                    Address          State     Voter  RaftProtocol  Commit Index  Trails Leader By
server1  9a12cc36-97a2-39be-1a4d-43aab65fda95  172.20.0.4:8300  follower  true   3             224           0 commits
server3  43580844-8783-dfce-ae36-eb7dfb549968  172.20.0.2:8300  leader    true   3             224           -
```

### Reincorporación de server2 (primera consulta)
```
Node     ID                                    Address          State     Voter  RaftProtocol  Commit Index  Trails Leader By
server1  9a12cc36-97a2-39be-1a4d-43aab65fda95  172.20.0.4:8300  follower  true   3             250           0 commits
server3  43580844-8783-dfce-ae36-eb7dfb549968  172.20.0.2:8300  leader    true   3             250           -
server2  632c5c51-33de-cd9f-2f8d-2e16b98563f2  172.20.0.3:8300  follower  false  3             250           0 commits
```

### Reincorporación de server2 (segunda consulta)
```
Node     ID                                    Address          State     Voter  RaftProtocol  Commit Index  Trails Leader By
server1  9a12cc36-97a2-39be-1a4d-43aab65fda95  172.20.0.4:8300  follower  true   3             260           0 commits
server3  43580844-8783-dfce-ae36-eb7dfb549968  172.20.0.2:8300  leader    true   3             260           -
server2  632c5c51-33de-cd9f-2f8d-2e16b98563f2  172.20.0.3:8300  follower  true   3             260           0 commits
```

## Interpretación
- Antes de la caída, server2 era el líder y los 3 nodos tenían el mismo
  Commit Index, o sea que estaban sincronizados.
- Al apagar a server2, al volver a consultar, los otros dos eligieron
  a server3 como nuevo líder usando Raft. El clúster siguió funcionando
  sin intervención: tolerancia a fallos y transparencia de fallos.
- server2 desapareció de la lista porque docker stop manda una señal de
  apagado ordenado (SIGTERM) y Consul lo tomó como salida voluntaria.
  Una caída real no avisa (eso se prueba con docker kill en el exp. 2).
- Al volver, server2 entró primero sin voto (Voter=false). Autopilot
  espera a que esté estable antes de darle voto. Luego pasó a Voter=true.
- server2 se puso al día solo (mismo Commit Index) y no hubo otra
  elección: server3 siguió de líder.
- Conclusión: Consul detecta el fallo, elige un nuevo líder, se recupera
  y reincorpora al nodo sin perder datos.