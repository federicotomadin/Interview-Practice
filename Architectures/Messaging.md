# Messaging: Kafka vs RabbitMQ

Las dos son herramientas de mensajería, pero resuelven problemas distintos. La confusión es común porque "ambas pasan mensajes", pero su arquitectura y casos de uso son muy diferentes.

---

## RabbitMQ — Message Broker tradicional

Es un broker que enruta mensajes de productores a consumidores. Sigue el modelo AMQP.

**Modelo mental:** una oficina de correos. Llega un mensaje, lo entrega al destinatario, y una vez entregado, lo descarta. El mensaje deja de existir.

```
Producer → Exchange → Queue → Consumer (consume → ACK → mensaje borrado)
```

### Características

- **Smart broker, dumb consumer:** el broker hace todo el ruteo (exchanges, bindings, routing keys).
- **Push-based:** el broker empuja mensajes a los consumidores.
- **Mensaje efímero:** una vez consumido y confirmado (ACK), se borra.
- **Patrones flexibles:** pub/sub, routing por topic, fanout, RPC.
- **Throughput:** decenas de miles de mensajes por segundo.
- **Reintentos y DLX** (Dead Letter Exchange): nativos.

---

## Kafka — Distributed Event Log

Es un log distribuido y persistente. Los mensajes se escriben append-only a un archivo, y los consumidores leen desde una posición (offset).

**Modelo mental:** el libro de actas de una empresa. Los eventos se escriben en orden, quedan ahí para siempre (o por el tiempo de retención), y cualquiera puede venir a leer desde donde quiera. Leer no borra nada.

```
Producer → Topic (particionado) → [evento][evento][evento][evento]...
                                       ↑                ↑
                                  Consumer A       Consumer B
                                  (offset 5)       (offset 12)
```

### Características

- **Dumb broker, smart consumer:** el broker solo guarda mensajes; el consumidor maneja su propio offset.
- **Pull-based:** el consumidor pide mensajes cuando quiere.
- **Mensaje persistente:** queda en disco según política de retención (días, semanas, ilimitado).
- **Particiones:** el topic se divide para escalar horizontalmente. Dentro de una partición hay orden estricto.
- **Throughput:** millones de mensajes por segundo.
- **Replay:** podés "rebobinar" y reprocesar eventos viejos.

---

## Comparación

| Aspecto | RabbitMQ | Kafka |
|---|---|---|
| Modelo | Queue / broker | Log / stream |
| Persistencia | Hasta ACK | Por tiempo configurado (días/años) |
| Replay de mensajes | No | Sí (volvés a un offset) |
| Routing | Sofisticado (exchanges, routing keys) | Simple (topics + particiones) |
| Orden | Por queue | Por partición |
| Throughput típico | 10K–50K msg/s | 100K–1M+ msg/s |
| Latencia | Muy baja (sub-ms) | Baja, pero más alta que Rabbit |
| Reintentos | Nativos (DLX) | El consumidor los maneja |
| Curva de aprendizaje | Más suave | Más empinada |

---

## Cuándo usar cada uno

### RabbitMQ

- Necesitás procesamiento de tareas (workers, jobs).
- Querés ruteo complejo basado en headers/topics.
- Cada mensaje representa un **comando** ("procesá este pago", "enviá este mail").
- Te importa que cada mensaje se procese una vez y se descarte.
- Necesitás patrones request/reply o RPC.

> Ejemplo: en una API de Policies, mandás un mensaje "renovar póliza 12345" a una cola, un worker lo toma, lo procesa, ACK, listo.

### Kafka

- Tenés eventos que múltiples sistemas necesitan consumir independientemente.
- Te importa el orden estricto dentro de un agregado/partición.
- Querés event sourcing o auditoría histórica.
- Necesitás stream processing (Kafka Streams, Flink, Spark).
- Hacés analytics o pipelines de datos en tiempo real.

> Ejemplo: cada vez que se crea/modifica una Policy, publicás un evento `PolicyChanged`. El servicio de notificaciones lee para mandar emails, el de analytics para dashboards, el de auditoría para el histórico — todos consumen lo mismo a su ritmo.

### Regla rápida

- ¿Tarea para procesar y olvidar? → **RabbitMQ**
- ¿Evento que cuenta algo que pasó y varios sistemas necesitan saber? → **Kafka**

En arquitecturas grandes **coexisten**: Rabbit para commands, Kafka para events.
