# HOIST para iPhone (IPA)

La app es la misma `index.html` de la raíz. El workflow `.github/workflows/ios-build.yml`
la envuelve con Capacitor y compila en un runner macOS de GitHub. Aquí no hay carpeta `ios/`:
se genera en cada compilación, así que **editar `index.html` actualiza Pages y el IPA a la vez**.

## Una sola vez (desde tu PC Intel, sin Mac)

1. **Registrar el iPhone.** Portal de Apple → Certificates, Identifiers & Profiles → Devices → añade el UDID
   de cada iPhone (17 Pro y Duo).
2. **Identificador.** Identifiers → App IDs → nuevo con el Bundle ID que elijas (por defecto `com.cohenzero.hoist`).
3. **Certificado de desarrollo.** En el PC (Git Bash/WSL con openssl):
   ```
   openssl genrsa -out hoist.key 2048
   openssl req -new -key hoist.key -out hoist.csr -subj "/CN=Alberto/emailAddress=TU_CORREO"
   ```
   Subes `hoist.csr` en Certificates → "Apple Development" y descargas `development.cer`. Luego:
   ```
   openssl x509 -inform der -in development.cer -out development.pem
   openssl pkcs12 -export -inkey hoist.key -in development.pem -out hoist.p12 -passout pass:LA_CLAVE_QUE_ELIJAS
   ```
4. **Perfil de aprovisionamiento.** Profiles → "iOS App Development" → tu App ID, tu certificado y tus iPhones.
   Descarga el `.mobileprovision` y anota su nombre.
5. **Secretos y variables del repo** (Settings → Secrets and variables → Actions):
   - Secrets: `CERT_P12_BASE64` (`base64 -w0 hoist.p12`), `CERT_PASSWORD`, `PROFILE_BASE64` (`base64 -w0 perfil.mobileprovision`).
   - Variables: `BUNDLE_ID`, `TEAM_ID` (Membership en el portal) y `PROFILE_NAME`.

   Guarda `hoist.key` y `hoist.p12` en un sitio seguro: sin ellos no puedes volver a firmar.

## Cada compilación

Actions → "Compilar IPA (iOS)" → Run workflow → descarga `HOIST-ipa` → instala `HOIST.ipa` con tu instalador.

## Ojo

- El IPA tiene **su propio almacenamiento**: no hereda los datos de la PWA. Al abrirlo, introduce la llave de GitHub
  en Ajustes y recupera los datos desde `hoist-datos`.
- La vibración (`navigator.vibrate`) no existe en iOS, ni en PWA ni aquí. Si la quieres, hay que añadir el plugin Haptics.
- Los perfiles de desarrollo caducan al año: repite el paso 4 y actualiza `PROFILE_BASE64`.
