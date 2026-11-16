# TA-IND-01 — Análisis crítico de HashiCorp Consul vs Netflix Eureka

**Universidad Técnica Estatal de Quevedo** — Facultad de Ciencias de la Computación
**Carrera:** Ingeniería de Software
**Asignatura:** Sistemas Distribuidos
**Actividad:** TA-IND-01 — Informe Académico Individual
**Estudiante:** Leonel Leonidas Zambrano Triviño
**Paralelo:** B
**Fecha de entrega:** 15 de noviembre de 2026

---

## Video de sustentación

**Enlace:** https://youtu.be/gbmRAb0Qs1k

---

## Descripción

Análisis crítico de **HashiCorp Consul** (sistema de descubrimiento de servicios) comparado con **Netflix Eureka**, aplicando los conceptos de la Unidad 1: transparencias de distribución, teorema CAP y PACELC, modelos de comunicación y tolerancia a fallos.

El análisis se respalda con un clúster local de 3 servers de Consul levantado con Docker y dos experimentos propios:

1. **Caída del líder:** se detiene el líder y se observa la reelección con Raft y la reincorporación del nodo.
2. **Pérdida de quórum:** se apagan 2 de 3 servers y se prueban escrituras, lecturas normales y lecturas `stale`.

---

## Estructura del repositorio

```
TA-IND-01-Consul-vs-Eureka/
├── README.md                      # Este archivo
├── BITACORA.md                    # Bitácora de trabajo por día
├── arquitectura-consul.drawio     # Fuente editable del diagrama
├── capturas/                      # Capturas de la ejecución, Anexo A y Anexo B
├── experimento/
│   └── consul/
│       ├── docker-compose.yml             # Clúster de 3 servers
│       ├── experimento1-caida-lider.md    # Comandos, salida e interpretación
│       └── experimento2-perdida-quorum.md # Comandos, salida e interpretación
└── latex/
    ├── main.tex                   # Archivo principal del informe
    ├── referencias.bib            # Referencias (biblatex, estilo IEEE)
    ├── figuras/                   # Diagrama exportado
    └── main.pdf                   # Informe compilado
```

---

## Compilación del informe

### Dependencias

- Distribución LaTeX: **MiKTeX** (Windows) o **TeX Live** (Linux/macOS)
- Compilador: **pdflatex**
- Procesador de bibliografía: **biber**
- Paquetes principales: `babel` (spanish), `biblatex` (style=ieee), `graphicx`, `booktabs`, `tabularx`, `float`, `listings`, `xcolor`, `csquotes`, `hyperref`, `lmodern`

> Con MiKTeX, los paquetes que falten se instalan automáticamente al compilar si está activada la opción *"Install missing packages on-the-fly"*.

### Archivo principal

`latex/main.tex`

### Orden de comandos

Desde la raíz del repositorio:

```bash
cd latex
pdflatex main
biber main
pdflatex main
pdflatex main
```

El resultado es `latex/main.pdf`.

> Las capturas se leen desde `../capturas/` y el diagrama desde `figuras/`, por eso hay que compilar dentro de la carpeta `latex`.

---

## Reproducir el experimento

### Requisitos

- Docker Desktop (o Docker Engine) con Docker Compose

### Levantar el clúster

```bash
cd experimento/consul
docker compose up -d
docker exec consul-server1 consul members
docker exec consul-server1 consul operator raft list-peers
```

Interfaz web: http://localhost:8500

### Experimento 1: caída del líder

```bash
docker stop consul-serverX        # X = número del líder actual
docker exec consul-server1 consul operator raft list-peers
docker start consul-serverX
docker exec consul-server1 consul operator raft list-peers
```

### Experimento 2: pérdida de quórum

```bash
docker exec consul-server1 consul kv put tarea/estado "antes-de-la-caida"
docker kill consul-server3
docker kill consul-server2
docker exec consul-server1 consul kv put tarea/estado "durante-la-caida"
docker exec consul-server1 consul kv get tarea/estado
docker exec consul-server1 consul kv get -stale tarea/estado
docker start consul-server2 consul-server3
docker exec consul-server1 consul kv get tarea/estado
```

### Detener el clúster

```bash
docker compose stop
```

Los resultados completos y su interpretación están en `experimento/consul/`.

---

## Uso de inteligencia artificial

El uso de IA está declarado en la sección *"Declaración de uso de inteligencia artificial"* del informe (`latex/main.pdf`).
