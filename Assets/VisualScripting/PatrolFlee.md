# Patrolling / Flee

`PatrolFlee.asset` es un **State Graph** con dos Script States y dos transiciones por proximidad. Todo el comportamiento usa nodos estandar de Unity Visual Scripting.

## Configuracion

1. Pon el agente sobre un NavMesh horneado y añade un componente **NavMeshAgent**.
2. Añade **State Machine**, selecciona Source = Graph y asigna `PatrolFlee.asset`. Unity añade el componente **Variables** automaticamente.
3. En **Variables > Object**, crea `Threat` de tipo **GameObject** y arrastra el objeto del que debe huir.
4. Crea un objeto vacio fijo llamado `PatrolRoute`, fuera de la jerarquia del agente. Añade objetos vacios hijos (`Point0`, `Point1`, `Point2`...) en posiciones accesibles sobre el NavMesh. El orden de los hijos determina el recorrido.
5. En las variables del agente, crea `PatrolRoute` de tipo **GameObject** y asigna ese objeto padre.
6. Si el agente tiene otro Script Machine de movimiento, por ejemplo `Wander`, desactivalo para que ambos grafos no controlen el mismo NavMeshAgent.
7. Inicia Play. Puedes observar el estado activo en el State Graph y en la variable de objeto `CurrentState`.

## Comportamiento

- **Patrolling** es el estado inicial. Recorre los hijos de `PatrolRoute` en orden y repite. Al volver de Flee solicita una nueva ruta de patrulla, conservando la anterior hasta que la nueva sea completa y aceptada. Mantiene su indice y anticipa el siguiente punto para no frenar deliberadamente en cada llegada. Si un punto no ofrece una ruta completa, prueba el siguiente; los reintentos de recuperacion se espacian 0,25 segundos.
- **Patrolling -> Flee** cuando la distancia horizontal hasta `Threat` es de 6 unidades o menos.
- **Flee** aumenta la velocidad y busca un destino sobre el NavMesh cuya distancia horizontal a `Threat` sea, como minimo, `max(SafeDistance, DetectDistance + 1)`. El candidato se genera alrededor de la amenaza, con radio suficiente para superar ese limite. Se comprueba de nuevo la distancia del punto real devuelto por `SamplePosition`, y solo se acepta si ademas avanza en la direccion de escape y tiene una ruta completa. Recalcula cada 0,35 segundos; tras un intento fallido prueba direcciones alternativas de hasta 60 grados a ambos lados.
- **Flee -> Patrolling** cuando la amenaza esta a 10 unidades o mas, o deja de existir. Las dos distancias distintas evitan oscilaciones entre estados cerca del limite.
- Si falta `Threat`, sigue patrullando. Si falta `PatrolRoute` o no tiene hijos, permanece quieto en Patrolling, pero sigue comprobando la amenaza.
- Los giros usan la rotacion del NavMeshAgent, limitada por `TurnSpeed`, y una aceleracion gradual.

## Parametros opcionales

Puedes crearlos como variables **Object**, de tipo **Float**, en el mismo agente. Si no existen, el grafo usa estos valores:

| Variable | Valor | Funcion |
| --- | ---: | --- |
| `DetectDistance` | 6 | Distancia para empezar a huir |
| `SafeDistance` | 10 | Distancia para volver a patrullar |
| `PatrolSpeed` | 2,5 | Velocidad de patrulla |
| `FleeSpeed` | 5 | Velocidad al huir |
| `WaypointDistance` | 1 | Distancia para pasar al siguiente punto |
| `FleeDistance` | 8 | Separacion adicional deseada respecto a la distancia actual a la amenaza; la distancia segura tiene prioridad |
| `RepathInterval` | 0,35 | Segundos entre recalculos de huida |
| `SampleDistance` | 2 | Radio para buscar NavMesh cerca del destino |
| `TurnSpeed` | 120 | Velocidad de giro, en grados por segundo |
| `Acceleration` | 4 | Aceleracion del agente |

`SafeDistance` se fuerza a un minimo de `DetectDistance + 1`. Usa velocidades, radios e intervalos positivos. Coloca los puntos de patrulla separados por mas de `WaypointDistance`.

Si `remainingDistance` devuelve infinito o NaN durante la patrulla, la llegada se comprueba usando la distancia al ultimo destino aceptado, para poder avanzar al siguiente punto.

La deteccion y la distancia segura se miden en el plano XZ; no comprueban vision ni paredes. En el Script State Flee, `SafeFleeDestination` muestra en Graph Variables el ultimo destino aceptado. La validacion se hace con la posicion de la amenaza en el momento de elegir el punto; si la amenaza se mueve, se recalcula en los siguientes intentos.

Si no encuentra un punto suficientemente alejado con ruta completa, conserva su ruta anterior y sigue buscando. No garantiza escapar de un recinto cerrado ni encontrar un punto seguro si no existe uno accesible en las direcciones exploradas.
