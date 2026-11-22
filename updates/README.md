# Abstral — actualización de clientes

Este repositorio **público** distribuye instaladores. El código fuente y los secretos de firma deben permanecer privados. Windows y Android usan el mismo repositorio de distribución, pero versiones y canales independientes.

## Windows (canal estable)

- Etiquetas: `v0.48.1`, `v0.48.2`, etc., **no prerelease**.
- Artefactos: `Abstral-Setup-X.Y.Z.exe`, `abstral-update.json`, `abstral-update.json.sig` y firma binaria del instalador.
- El cliente nativo Rust ya consulta `/releases/latest/download/abstral-update.json`, verifica las firmas RSA y SHA-256 y prepara el instalador para ejecutarlo al cerrar Abstral.
- **Nunca** publicar una versión Android como «Latest» estable: un cliente Windows antiguo interpretaría esa publicación como su versión más reciente y fallaría si faltan los archivos Windows.
- Para comprobar la actualización de Windows de extremo a extremo se necesita una versión estable más reciente y un ensayo en instalación aislada. No se ha realizado esa prueba aquí.

## Android (canal beta)

- Etiquetas: `android-v0.9.1`, `android-v0.9.2`, etc., siempre con bandera **prerelease**.
- Archivo único y predecible por release: `Abstral-Android-X.Y.Z.apk`.
- Manifiesto público: [updates/android/beta.json](updates/android/beta.json) (el cliente consulta la versión raw de GitHub).
- Contrato: `schema=1`, `platform=android`, `channel=beta`, `package_name`, `version_code`, `version`, `apk_url`, `bytes`, `sha256`, `notes`, `mandatory`.
- **Primera instalación** del actualizador: instalar manualmente Android 0.9.1 (versionCode 17). Las versiones posteriores con mayor `versionCode` se descubren automáticamente.
- El cliente rechaza descargas que no correspondan al paquete `io.abstral.mobile.m0`, a la firma del APK instalado o a su SHA-256 publicado.
- Android solicita la aprobación del usuario para instalar actualizaciones por APK; no es instalación silenciosa. La conservación de sesión depende de la misma firma y paquete de Android.
- La versión 0.9.1 es beta **firmada con la clave de depuración usada por versiones anteriores**. No usar esa clave como mecanismo de firma de producción. Migrar de clave fuera de Play App Signing normalmente requerirá desinstalar, por lo que debe definirse antes del lanzamiento estable.

## Procedimiento seguro para publicar Android 0.9.2+

1. Compilar y ejecutar pruebas en una copia aislada. Incrementar `versionCode` (18, 19, ...). Conservar `applicationId` y la firma mientras continúe la beta.
2. Verificar firma del APK, paquete y SHA-256; probar Android con dos cuentas y sin pérdida de sesión.
3. Publicar el APK como **prerelease**, tag `android-vX.Y.Z`. Verificar el checksum del asset servido por GitHub.
4. **Después** de comprobar que el archivo remoto puede descargarse, cambiar `updates/android/beta.json` al nuevo tag, tamaño, SHA-256 y versión. Nunca actualizar el manifiesto antes del asset.
5. Verificar `/releases/latest` sigue apuntando a una versión Windows estable y que el APK Android es accesible.
6. Probar desde un Android con versión anterior y aceptar manualmente la instalación en el instalador del sistema.
7. Si una versión falla, retirar temporalmente el manifiesto o publicar otra con `versionCode` superior. No forzar downgrades.

## Producción y mejoras futuras

Unificar el código fuente en un monorepo privado es opcional y debe hacerse por etapas (p. ej. `apps/windows`, `apps/android`, `server`, `shared` y `release-tools`). No es requisito para compartir las descargas y no debe combinarse con una migración no probada de clientes o backend.

Para actualizaciones tipo Discord completamente automáticas en Windows, conservar los manifiestos **firmados con la clave ya configurada**; implementar CI que construya y publique instaladores después de aprobar tests. Android fuera de Google Play no puede actualizar silenciosamente sin permisos privilegiados.
