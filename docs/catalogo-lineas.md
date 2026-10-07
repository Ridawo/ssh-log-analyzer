| # | Patrón de la línea | Campos extraíbles | ¿Intento fallido? | Por qué |
|---|---|---|---|---|
| 1 | `Invalid user <USER> from <IP> port <PORT>` | timestamp, PID, USER, IP, PORT | depende | Se intentó un usuario inexistente. Sale una vez por conexión, no por contraseña probada. |
| 2 | `Connection closed by invalid user <USER> <IP> port <PORT> [preauth]` | timestamp, PID, USER, IP, PORT | no | El cliente cerró antes de autenticarse. Consecuencia de la fila 1, no un intento nuevo. |
| 3 | `Failed password for invalid user <USER> from <IP> port <PORT> ssh2` | timestamp, PID, USER, IP, PORT | sí | Contraseña rechazada para un usuario inexistente. Sale una vez por contraseña probada. |
| 4 | `Failed password for <USER> from <IP> port <PORT> ssh2` | timestamp, PID, USER, IP, PORT | sí | Contraseña rechazada para un usuario existente. Sale una vez por contraseña probada. |
| 5 | `Accepted password for <USER> from <IP> port <PORT> ssh2` | timestamp, PID, USER, IP, PORT | no | Login correcto. No es fallo, pero sirve para detectar un acierto tras muchos fallos. |
| 6 | `Connection closed by authenticating user <USER> <IP> port <PORT> [preauth]` | timestamp, PID, USER, IP, PORT | no | El cliente cerró antes de autenticarse, con usuario existente. Puede seguir a fallos ya contados (fila 4) o a una conexión cancelada sin intentos. |
| 7 | `Connection closed by <IP> port <PORT> [preauth]` | timestamp, PID, IP, PORT | no | El cliente cerró antes de dar usuario ni contraseña. Es la única huella de una conexión sin intento de login. |
| 8 | `pam_unix(sshd:session): session opened for user <USER>(uid=<UID>) by <USER>(uid=<UID>)` | timestamp, PID, USER | no | Sesión abierta tras un acierto. Acompaña a la fila 5. |
| 9 | `pam_unix(sshd:auth): authentication failure; logname= uid=<UID> euid=<UID> tty=ssh ruser= rhost=<IP>  user=<USER>` | timestamp, PID, USER, IP | no | Duplica el primer `Failed password` (fila 4). Contarla sumaría el mismo fallo dos veces. Tiene doble espacio antes de `user=`. |
| 10 | `PAM <N> more authentication failures; logname= uid=<UID> euid=<UID> tty=ssh ruser= rhost=<IP>  user=<USER>` | timestamp, PID, USER, IP | no | Resume los fallos que siguen al primero. Ya están contados en la fila 4. Sumar N los duplicaría. |



## Notas

- Formatos de IP vistos: IPv6 (`::1`), IPv4 (`127.0.0.1`). En las líneas PAM la IP va en `rhost=`, y en `Connection closed by` no lleva `from`.
- Variantes de la misma línea: `Failed password` con y sin `invalid user`; `Connection closed by` con `invalid user`, con `authenticating user` y sin usuario.
- Líneas que duplican a otra: 2 → 1; 6 → 4 (no siempre); 9 y 10 → 4.
- Unidad de fallo: una contraseña rechazada (filas 3 y 4). La fila 1 se usa solo como contexto.
- Casos que mi detector NO cubre: <decide si cuentas la fila 7 y apúntalo aquí>.