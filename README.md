# 🛡️ CI/CD Gatekeeper AI con n8n & Claude 3.5 Sonnet

Sistema automatizado de **DevSecOps Gatekeeper** para auditoría y control de calidad en Pull Requests de GitHub. Integra la orquestación de **n8n Self-Hosted**, el modelo **Anthropic Claude 3.5 Sonnet** (con capacidad extendida de 8,192 tokens) y las **APIs REST de GitHub** para bloquear automátiamente despliegues con vulnerabilidades críticas o aprobar cambios seguros.

---

## 📌 Repositorios del Proyecto

* **Repositorio del Gatekeeper & Infraestructura:** [gastonarce777-bit/ci-cd-gatekeeper-ai](https://github.com/gastonarce777-bit/ci-cd-gatekeeper-ai)
* **Repositorio Target de Pruebas (Mini Ecommerce):** [gastonarce777-bit/mini-ecommerce-storefront](https://github.com/gastonarce777-bit/mini-ecommerce-storefront)

---

## 🚀 Requisitos Previos

* **Docker & Docker Compose** instalados en la máquina host.
* Una cuenta de **Anthropic API** (para obtener la API Key de Claude).
* Una cuenta de **GitHub** con un **Personal Access Token (PAT)** con permisos `repo` y `pull_requests:write`.
* Una **cuenta secundaria de GitHub** agregada como colaboradora en el repositorio de prueba (para evitar el bloqueo por auto-aprobación HTTP 422).

---

## 🛠️ Guía de Instalación y Despliegue Paso a Paso

### 1. Clonar el Repositorio
```bash
git clone [https://github.com/gastonarce777-bit/ci-cd-gatekeeper-ai](https://github.com/gastonarce777-bit/ci-cd-gatekeeper-ai)
cd ci-cd-gatekeeper-ai
```

### 2. Iniciar el Entorno con Docker Compose
Levanta los contenedores de **n8n** y **Cloudflare Tunnel** (configurado con DNS público `8.8.8.8` para evitar bloqueos de red interna):

```bash
cd docker
docker-compose up -d
```

### 3. Obtener la URL Pública de Cloudflare Tunnel
Revisa los logs del contenedor de Cloudflare para obtener la URL HTTPS generada automáticamente:

```bash
docker logs cloudflared_tunnel
```
*Busca una línea similar a:* `https://xxxx-xx-xx-xx.trycloudflare.com`

### 4. Importar el Workflow en n8n
1. Abre tu navegador y accede a `http://localhost:5678`.
2. Completa la configuración inicial de n8n.
3. Ve a **Workflows** -> **Import from File**.
4. Selecciona el archivo `workflow/workflow-gatekeeper-ai.json` incluido en este repositorio.

### 5. Configurar Credenciales en n8n
Dentro de n8n, configura las dos credenciales necesarias:
* **Anthropic Header Auth:** Agrega tu `x-api-key` de Anthropic.
* **GitHub Header Auth:** Agrega tu `Authorization: Bearer <TU_PAT_GITHUB>`.

### 6. Configurar el Webhook en GitHub
1. Entra a tu repositorio objetivo (`mini-ecommerce-storefront`) -> **Settings** -> **Webhooks** -> **Add webhook**.
2. **Payload URL:** `https://tu-url-cloudflare.trycloudflare.com/webhook/github-pr-webhook`
3. **Content type:** `application/json`
4. **Which events would you like to trigger this webhook?:** Selecciona **Let me select individual events** y marca únicamente **Pull requests**.
5. Haz clic en **Add webhook**.

---

## 🧪 Cómo Reproducir las Pruebas de Funcionamiento

### Escenario A: Probar Bloqueo Automático (`REQUEST_CHANGES`)
1. Desde una cuenta secundaria colaboradora, crea una rama en el repositorio objetivo:
   ```bash
   git checkout -b test-vulnerabilidad-critica
   ```
2. Agrega un archivo con una clave expuesta o código inseguro (ejemplo `test-security.js`):
   ```javascript
   const STRIPE_SECRET_KEY = "sk_live_51MzXXXXXXXXXXXXXXXXXXXXX";
   function execute(userInput) {
     return eval(userInput); // Vulnerabilidad RCE / Injection
   }
   ```
3. Abre un **Pull Request** hacia la rama `main`.
4. **Resultado esperado:** n8n procesará la solicitud en 4-8 segundos, Claude detectará la vulnerabilidad `CRITICAL` y el bot publicará un review bloqueando el PR con **`REQUEST_CHANGES`**.

### Escenario B: Probar Aprobación Automática (`APPROVE`)
1. Crea una nueva rama con cambios limpios y documentados.
2. Abre un **Pull Request** hacia `main`.
3. **Resultado esperado:** El evaluador de severidad (`If1`) tomará la rama `FALSE` y el bot aprobará el PR con **`APPROVE`**.

---

## 📄 Documentación Técnica
El diseño detallado del sistema, arquitectura por fases, matriz de manejo de excepciones y justificación estratégica se encuentran documentados en el PDF oficial ubicado en:  
`docs/DDT_CI_CD_Gatekeeper_AI_v1.0.pdf`
