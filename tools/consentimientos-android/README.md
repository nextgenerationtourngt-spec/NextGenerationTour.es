# NGT · Consentimiento audiovisual Android (preparación técnica)

> DOCUMENTACIÓN PÚBLICA. **No subir a este repositorio vídeos, voces, rostros, consentimientos, identificadores reales, tokens ni claves.** Esta rama solo contiene las especificaciones para Codex; no modifica el sitio desplegado (rama main).

## Destino definitivo de respaldos

Codex deberá crear desde el PC, **mediante GitHub CLI autenticada**, un repositorio **PRIVADO** independiente:
`nextgenerationtourngt-spec/ngt-consentimientos-backup`. Si ya existe, verificar visibilidad PRIVADA antes de usarlo; si no es posible crear el repo de forma privada, bloquear la sincronización remota y mostrar el error, sin recurrir a uno público.

URL planificada (no creada aún): https://github.com/nextgenerationtourngt-spec/ngt-consentimientos-backup

La aplicación Android usa la cámara y micrófono del móvil del operador. No requiere que participantes instalen ninguna app ni entreguen nombre, teléfono, email o documento.

## Funciones mínimas

1. Panel de operador protegido: sesión/evento, nuevo participante (UUID aleatorio y código corto visible), aceptar o rechazar sin penalizar la experiencia.
2. Presentar antes de grabar texto informativo completo y versión identificada; registrar consentimiento afirmativo separado, explícito y libre. No asumir que hablar equivale a consentir si no se informó adecuadamente.
3. Cámara frontal + audio, vídeo breve revisable y confirmación expresa del participante. Mensaje vocal orientativo configurable: «Autorizo a NGT a usar mi imagen y voz en sus redes sociales para promocionar este evento»; permitir 8-12 s, sin recorte automático del enunciado.
4. Guardado local durable, cifrado, **antes de intentar subir**. Mostrar estados: guardado local / esperando Internet / subiendo / respaldo verificado / error / revocado.
5. Vincular código a dispositivo Quest 3 y sesión, opcionalmente archivos promocionales; no reconocimiento facial automatizado.
6. Pantalla consulta con estado, copia local y remoto, reintentos y exportación cifrada al PC como respaldo complementario.
7. Gestión de retiro del consentimiento con auditoría y procedimiento seguro para supresión del material, sujeto a revisión legal y retención de prueba necesaria. No borrar evidencia ciegamente por una pulsación.
8. Participantes menores: bloquear publicación por defecto; necesitar flujo de representante legal revisado jurídicamente.
9. Idiomas ES/EN. Política y finalidades editables por el responsable, con versión, fecha y hash de texto exacto. No inventar validez legal universal.
10. Administrador de la app (PIN o biometría), bloqueo por inactividad, sin analíticas de terceros, sin reconocimiento facial ni biométrico.

## Cifrado y secretos

- Generar clave de recuperación de 256 bits con CSPRNG, **guardarla fuera de GitHub y fuera del teléfono**; requerir confirmación de resguardo. Nunca meterla en código, logs ni copias sin cifrar.
- Cifrar cada paquete (MP4 + registro/manifest) con AES-256-GCM, IV aleatorio distinto por paquete, AAD versionada y claves derivadas/envoltorio documentado; considerar Android Keystore para claves locales, pero asegurar recuperación en otro dispositivo mediante la clave externa.
- SHA-256 del MP4 original y hash del paquete cifrado, tamaños y fechas en manifest interno cifrado; archivar prueba original sin edición.
- Token GitHub fine-grained limitado a un único repo privado, permisos Contents read/write; introducir durante configuración, protegido en Android Keystore, sin incluir en APK o repo. Habilitar revocación/cambio.

## Respaldo GitHub

No hacer commit de vídeos cifrados como blobs Git: el historial dificultaría retención y borrado. Preferir **GitHub Releases como depósito de assets cifrados**: una release privada por evento (o particionar si se alcanzan límites) y un asset único por consentimiento. Nombres opacos por UUID, metadata privada dentro del paquete cifrado. Subir por REST API como binario y verificar descargas/sha antes de confirmar; no confiar solo en HTTP 201. En caso de choque de nombre, consultar remoto y comprobar hash; jamás sobrescribir silenciosamente. Respetar límites y rate limiting de GitHub.

**GitHub NO es un sistema de conservación probatoria inmutable ni un archivo legal certificado.** Para casos de alto riesgo, añadir respaldo cifrado externo y sellado de tiempo independiente.

## Pruebas obligatorias

Modo avión al iniciar, perder Internet a mitad de upload, cierre forzado, reinicio Android, batería baja, almacenamiento casi lleno, cuota/API GitHub, revocación de token, archivo corrupto, actualización APK, teléfono extraviado (simular restauración con clave), doble toque para evitar duplicados; confirmar que no se pierde grabación confirmada ni se declara respaldo sin verificación.

Documentación técnica adicional: `BACKUP_PROTOCOL.md`.
