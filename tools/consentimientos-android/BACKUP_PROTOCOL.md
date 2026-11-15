# Protocolo de respaldo fiable de autorizaciones — Android → GitHub privado

## Principio central
**Nunca depender de Internet para guardar el consentimiento. Nunca marcar como respaldado sin verificar.** La captura audiovisual debe permanecer en almacenamiento privado de Android hasta que la copia remota y una copia adicional opcional estén verificadas. La app debe conservar trazabilidad de errores.

## Estados persistentes (Room/SQLite + archivos privados)
- `DRAFT`: texto visto, antes de grabar. Sin autorización final.
- `RECORDED`: vídeo local terminado pero falta confirmación; no autoriza publicación.
- `LOCAL_SECURE`: participante confirma y paquete cifrado escrito de forma durable (archivo temporal, fsync, atomic rename, transacción DB).
- `PENDING_NETWORK`: pendiente de subida, admite offline.
- `UPLOADING`: tarea actual controlada por WorkManager, con lease/identificador de intento.
- `VERIFYING`: asset recibido, falta comprobar contenido/sha/tamaño desde GitHub.
- `BACKED_UP`: descarga y sha coinciden; guardar asset_id, release_id y fecha confirmada.
- `RETRYABLE_ERROR`: error de red/5xx/429, backoff exponencial + jitter, reintento manual.
- `ACTION_REQUIRED`: permisos 401/403, repo no privado, espacio insuficiente, clave de recuperación ausente.
- `WITHDRAWN`: retirada registrada, impedir nuevas publicaciones; gestionar eliminación de assets conforme a política y obligaciones.
- `CORRUPTED`: el archivo local o remoto no verifica; avisar y conservar lo recuperable para auditoría.

Los estados de consentimiento y los de sincronización deben modelarse **por separado**: `NO`, `YES`, `WITHDRAWN` y `LOCAL_SECURE`, etc., evitando confundir autorización con subida.

## Identificador y paquete
- Identificador interno: UUID v4 aleatorio; visible: código corto 6–8 chars sin actuar como clave secreta.
- Cada asset opaco `<uuid>.ngtc` contiene sobre cifrado versionado: magic/version, salt/IV si corresponde, wrapped-DEK, ciphertext y GCM-tag.
- JSON **dentro del cifrado**: UUID, evento, autor del tratamiento o responsable, texto completo y versión/hash, idioma, fecha ISO-8601 con offset, consentimiento YES/NO, hash SHA256 del original, modelo de dispositivo local, ID de sesión Quest si se registró, auditoría, publicación y revocación.
- Guardar log local append-only lógico con hashes encadenados, sin afirmar que sea evidencia inmutable/certificada.
- Nunca incluir un token GitHub o clave de cifrado en la copia.

## GitHub privado, Releases
1. Comprobar GitHub /repos/{owner}/{repo} y `private=true` (ante error bloquear uploads).
2. Garantizar release del evento; evitar nombres con datos personales. Documentar uso y limitaciones.
3. Crear o verificar asset por UUID: si ya existe, recuperar y comprobar hash antes de decidir.
4. Subir paquete cifrado por endpoint de release assets utilizando Content-Type `application/octet-stream`.
5. Verificar respuesta, tamaño y, **preferentemente**, descargar/leer y comparar hash SHA-256 del asset con local; 201 no basta.
6. Persistir confirmación y IDs. Nunca borrar automáticamente el único fichero local por subida aparente; retención configurable con periodo de seguridad.
7. Si el servidor respondió pero se perdió Internet antes del ACK, listar assets y comparar hash para deduplicar.
8. Si 429, 403 rate limit o 5xx: consultar cabeceras de reintento y aplicar backoff, NO repetir indefinidamente en bucle.
9. Descarga de restauración: listar assets, descargar con autenticación, validar integridad, descifrar con clave de recuperación externa, reconstruir DB y eventos, sin confiar en nombres.
10. Borrado/retirada: política documentada; la eliminación en GitHub no garantiza supresión instantánea de todas sus copias internas. Nunca prometerlo.

## Resiliencia
- Android: Room + WorkManager con constraints CONNECTED, scheduling persistente tras reinicio, reintentos y trabajo único por UUID, sin depender de la pantalla abierta; avisos de acción humana cuando Android restrinja tareas.
- Cámara: detener con error explícito cuando falla permiso, cámara/micrófono, espacio o escritura. Sin clip válido y aceptación no mostrar «autorizado».
- Local: almacenamiento app-specific protegido; 1 copia cifrada con verificación + exportación cifrada adicional (PC/USB/SD/SAF) para escenarios de pérdida o destrucción del dispositivo antes de sincronizar. Un único móvil sin Internet nunca puede garantizar recuperación frente a pérdida física.
- Restauración comprobada y ensayada, con recovery-key almacenada aparte y prohibición de registrar esa clave.

## Acceso y ciclo de vida
- Solo operador autorizado; cierre por inactividad y PIN/biometría.
- No rastreo de usuario ni SDK de analíticas. No pedir nombre/teléfono/email de participantes.
- Retención diferenciada por clases de registro y plazos revisados con asesor legal; eliminación verificable de copia local y de release assets cuando proceda.
- Avisar que rostros y voz son datos personales aunque se utilicen códigos, y que GitHub es proveedor de tratamiento a evaluar contractualmente.
- Tests automatizados de estados, cifrado, reintentos idempotentes y fallo/restauración; tests instrumentados en dispositivo físico.

Fuentes de referencia:
- https://docs.github.com/en/rest/releases/releases
- https://docs.github.com/en/rest/releases/assets
- https://developer.android.com/develop/background-work/background-tasks/persistent
