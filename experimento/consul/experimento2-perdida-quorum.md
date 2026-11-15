# Experimento 2: Pérdida de quórum en Consul

## Objetivo
Comprobar qué prioriza Consul cuando pierde la mayoría de servers
(partición): consistencia o disponibilidad. Así se verifica si es CP.

## Fundamento
Con 3 servers el quórum es 2. Si solo queda 1, no hay mayoría y no se
puede elegir líder.

## Comandos usados
```
docker exec consul-server1 consul kv put tarea/estado "antes-de-la-caida"
docker kill consul-server3
docker kill consul-server2
docker exec consul-server1 consul kv put tarea/estado "durante-la-caida"
docker exec consul-server1 consul kv get tarea/estado
docker exec consul-server1 consul kv get -stale tarea/estado
docker start consul-server2 consul-server3
docker exec consul-server1 consul kv get tarea/estado
```

## Salida obtenida
### Con quórum perdido (solo server1 vivo)
```
> consul kv put tarea/estado "durante-la-caida"
Error! Failed writing data: Unexpected response code: 500 (No cluster leader)

> consul kv get tarea/estado
Error querying Consul agent: Unexpected response code: 500 (No cluster leader)

> consul kv get -stale tarea/estado
antes-de-la-caida
```

### Después de recuperar el quórum
```
> consul kv get tarea/estado
antes-de-la-caida
```

## Interpretación
- Usé docker kill para simular una caída real, sin aviso.
- Con un solo server vivo, la escritura falló con "No cluster leader".
  Consul prefiere no responder antes que guardar datos que podrían quedar
  inconsistentes. Ante una partición elige consistencia: es CP.
- La lectura normal también falló, porque pasa por el líder.
- La lectura con -stale sí respondió con el valor guardado antes. El nodo
  usa su copia local aunque podría estar desactualizada.
- Esto muestra el matiz de PACELC: por defecto Consul elige consistencia,
  pero el cliente puede pedir disponibilidad a cambio de datos viejos.
- Al levantar los servers volvió el quórum y la lectura normal funcionó.
- Comparación: Eureka siempre responde, como si todo fuera -stale (AP).