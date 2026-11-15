# Bitácora — TA-IND-01

## 2026-11-13 (viernes)
- **Qué hice:** Primero revisé en el grupo qué sistemas ya estaban tomados para no repetir. Elegí HashiCorp Consul y compararlo
  con Netflix Eureka, porque los dos resuelven el descubrimiento de servicios, solo que uno es CP y el otro AP. Luego revisé las herramientas instaladas en mi PC y
  creé el repositorio en GitHub.
- **Problema encontrado:** Al revisar las herramientas, Maven, pdflatex y
  biber no estaban instalados.
- **Cómo lo resolví:** Para Eureka voy a usar el Maven wrapper (mvnw) que
  trae el proyecto de Spring Boot, así no tengo que instalar Maven. LaTeX lo
  instalare mañana.
- **Tiempo dedicado:** 30 min
- Ademas se levanto el clúster y vi que server3 es el lider.

## 2026-11-14 (sabado)
- **Experimento 1:** caída del líder en Consul y reelección con Raft.
- No tengo luz xd.