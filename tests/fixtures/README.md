# Fixtures de prueba

Logs para pruebas de fuerza bruta. Todos son inventados, tanto los hosts, usuarios y puertos como las IPs.

## Fixtures

| Fichero | Qué prueba | Alertas esperadas |
|---|---|---|
| `limite_10_fallos.log` | Umbral exacto de 10 fallos | 1 alerta de fuerza bruta: 192.0.2.10, 10 fallos, de 11:00:00 a 11:04:30 con usuario alice |
| `nueve_fallos.log` | Un fallo por debajo del umbral | Ninguna. 198.51.100.7 tiene 9 fallos |
| `diez_fallos_lentos.log` | Uso de la ventana: 10 fallos repartidos en 6 minutos | Ninguna. 203.0.113.20 llega como máximo a 8 en una ventana de 5 minutos |
| `dos_ips_cinco_cada_una.log` | Agrupar por IP | Ninguna. 192.0.2.10 y 203.0.113.5 hacen 5 fallos cada una, por lo que no salta alerta |
| `muchos_usuarios.log` | Una IP prueba muchos usuarios | 1 alerta: 198.51.100.50, 10 fallos, 10 usuarios distintos |
| `compromiso.log` | `Accepted` de la misma IP tras fuerza bruta | 2 alertas de 192.0.2.40: fuerza bruta y posible compromiso |
| `accepted_otra_ip.log` | `Accepted` de una IP distinta a la atacante | 1 alerta de fuerza bruta de 203.0.113.30. Sin alerta de compromiso |
| `accepted_sin_alerta.log` | Login normal y login tras pocos fallos | Ninguna. Son 3 fallos, por debajo del umbral, y puede ser un error al escribir la contraseña |
| `ruido.log` | Las filas 1, 2, 6, 7, 9 y 10 del catálogo no suman | Ninguna. Solo 7 fallos reales de la misma IP (192.0.2.10) |
| `cambio_de_anio.log` | Cruce del 31 de diciembre al 1 de enero | 1 alerta: 203.0.113.77, 10 fallos |

## Reglas

- Solo se usan IPs de 192.0.2.0/24, 198.51.100.0/24 y 203.0.113.0/24. Nunca se ponen los logs reales en el repo.
- Un fallo es una línea de las filas 3 o 4 del catálogo (`Failed password`). El resto de líneas no cuentan.
- El umbral es de 10 fallos de una misma IP en menos de 300 segundos. La ventana es deslizante y el borde es exclusivo.
- Cada alerta incluye IP, número de fallos, primer y último timestamp y usuarios probados.
- Los timestamps de syslog no traen año, por lo que los tests deberán fijarlo mediante `--year`.

## Notas por fixture

- `ruido.log` tiene 7 fallos reales y 10 líneas que no se cuentan. Si el parser suma las filas 9 y 10, superaría el umbral, y eso sería un error.
- `cambio_de_anio.log` necesita que `--year` sea el del 31 de diciembre. El detector suma un año cuando el timestamp retrocede.
- `accepted_otra_ip.log` genera la alerta de fuerza bruta de la IP atacante. No hay compromiso porque el `Accepted` viene de otra IP (198.51.100.15), que no tiene fallos previos.
