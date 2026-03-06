# Landing Tennis ML – Confirmar correo y restablecer contraseña

Esta carpeta contiene una web mínima para que los enlaces de **confirmar correo** y **restablecer contraseña** (enviados por Supabase) abran una página en el navegador en lugar de la app. Así cualquier persona (p. ej. un profesor) puede completar el flujo solo haciendo clic en el email.

## 1. Configurar la landing antes de desplegar

1. Abre **`auth/callback/index.html`**.
2. Sustituye al inicio del `<script>`:
   - **`SUPABASE_URL`**: la URL de tu proyecto. En Supabase: **Project Settings** (engranaje) → **API** → **Project URL**. Ejemplo: `https://abcdefgh.supabase.co`
   - **`SUPABASE_ANON_KEY`**: usa **solo** una clave pública (nunca la secreta ni `service_role`):
     - **Nuevas claves (2025+)**: en **Project Settings** → **API** busca la clave **Publishable** (`sb_publishable_...`). Es la que sustituye a `anon` y está pensada para frontend.
     - **Claves heredadas**: si aún ves la pestaña **Legacy API Keys**, ahí está **anon** (public). Puedes usar esa también.
     - **Nunca** pongas la clave **Secret** (`sb_secret_...`) ni **service_role** en el HTML; esas son solo para backend.

**¿Es seguro poner la clave en el HTML?** Sí. La clave publicable / `anon` está **diseñada** para ir en páginas web y apps: tiene privilegios bajos y la seguridad sigue dependiendo de las políticas (RLS) y de Auth. Supabase la considera “segura de exponer”. La que no debe salir nunca en el frontend es la clave secreta / `service_role`.

Guarda el archivo.

## 2. Desplegar en Vercel (recomendado)

1. Entra en [vercel.com](https://vercel.com) e inicia sesión (con GitHub, por ejemplo).
2. **Add New** → **Project**.
3. Si la landing está en tu repo:
   - Importa el repositorio.
   - En **Root Directory** pon: `tennis-ml-landing` (o la carpeta donde estén `index.html` y `auth/`).
   - Deploy.
4. Si no está en el repo: instala Vercel CLI y despliega desde tu máquina:
   ```bash
   cd tennis-ml-landing
   npx vercel
   ```
   Sigue los pasos (login si hace falta). Al terminar te dará una URL como `https://tennis-ml-landing-xxx.vercel.app`.
5. Anota la URL base de tu despliegue, por ejemplo:  
   `https://tu-proyecto.vercel.app`  
   La URL del callback será: **`https://tu-proyecto.vercel.app/auth/callback`**.

## 3. Desplegar en Netlify

1. Entra en [netlify.com](https://www.netlify.com) e inicia sesión.
2. **Add new site** → **Deploy manually** (o conecta tu repo y elige la carpeta `tennis-ml-landing`).
3. Para deploy manual: arrastra la carpeta **`tennis-ml-landing`** (con `index.html` y la carpeta `auth/` dentro) a la zona de drag & drop de Netlify.
4. Anota la URL que te asignen (ej. `https://random-name-123.netlify.app`). El callback será: **`https://tu-sitio.netlify.app/auth/callback`**.

## 4. Configurar Supabase

1. En Supabase: **Authentication** → **URL Configuration**.
2. **Site URL**: pon la URL base de tu landing, por ejemplo:
   - `https://tu-proyecto.vercel.app`
   - o `https://tu-sitio.netlify.app`
3. **Redirect URLs**: añade exactamente la URL del callback (una por línea), por ejemplo:
   - `https://tu-proyecto.vercel.app/auth/callback`
   - o `https://tu-sitio.netlify.app/auth/callback`
4. **Save**.

## 5. Logo en los correos (Tenly)

Para que los emails de Supabase (confirmar correo y restablecer contraseña) muestren el logo de Tenly:

1. **Consigue un PNG del logo**  
   Usa el logo que tienes en la app (`tennis-ml-frontend/assets/images/tenly-logo.svg`). Exporta o convierte a PNG (por ejemplo desde Canva, o con cualquier herramienta que permita “exportar como PNG”). Tamaño recomendado: unos 200–300 px de ancho, fondo transparente si es posible.

2. **Pon el archivo en esta carpeta**  
   Guarda el PNG con el nombre **`tenly-logo.png`** en la **raíz** de `tennis-ml-landing` (mismo nivel que `index.html`), por ejemplo:
   ```
   tennis-ml-landing/
   ├── index.html
   ├── tenly-logo.png   ← aquí
   └── auth/
       └── callback/
           └── index.html
   ```

3. **Vuelve a desplegar**  
   - Si la landing está conectada a un repo en Vercel/Netlify: haz commit del nuevo `tenly-logo.png`, push, y el deploy se hará solo.  
   - Si despliegas con Vercel CLI: desde la carpeta `tennis-ml-landing` ejecuta `npx vercel --prod` (o `vercel` para preview).

4. **Comprueba la URL**  
   Abre en el navegador:  
   `https://TU-URL-DE-VERCEL.vercel.app/tenly-logo.png`  
   (sustituye por la URL real de tu proyecto). Si ves el logo, los correos lo cargarán bien.

5. **Plantillas de correo**  
   Las plantillas en `tennis-ml-frontend/supabase/email-templates/` ya usan la URL del logo. Si tu landing tiene otra URL (por ejemplo un dominio propio), edita en esas plantillas la línea del `<img src="...">` y pon tu URL base + `/tenly-logo.png`.

## 6. Configurar la app (Expo)

En el proyecto de la app (`tennis-ml-frontend`), en tu archivo `.env` (o variables de entorno), define:

```env
EXPO_PUBLIC_AUTH_REDIRECT_URL=https://tu-proyecto.vercel.app/auth/callback
```

Sustituye por la URL real de tu landing (Vercel o Netlify). Así, al registrarse o pedir “restablecer contraseña”, Supabase enviará en el email un enlace que llevará a esa página.

## Resumen

| Dónde           | Qué hacer |
|-----------------|-----------|
| `auth/callback/index.html` | Poner tu `SUPABASE_URL` y `SUPABASE_ANON_KEY` |
| Vercel o Netlify | Desplegar la carpeta `tennis-ml-landing` |
| Supabase → URL Configuration | Site URL = base de la landing; Redirect URLs = `.../auth/callback` |
| App `.env`      | `EXPO_PUBLIC_AUTH_REDIRECT_URL=https://.../auth/callback` |
| Logo en correos | Añadir `tenly-logo.png` en la raíz de `tennis-ml-landing` y redesplegar |

Después de esto, al hacer clic en “Confirmar correo” o “Restablecer contraseña” del email se abrirá la landing, se completará el flujo en el navegador y se mostrará el mensaje de éxito. Si añades `tenly-logo.png` y redespliegas, ese logo aparecerá también en los correos.
