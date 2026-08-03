

# Habilidades de Pasarela de Pago de Malasia

![Made in Malaysia](https://img.shields.io/badge/Made%20in-Malaysia-red)
![Agent Skills](https://img.shields.io/badge/Agent-Skills-blue)
![Payment Gateway](https://img.shields.io/badge/Payment-Gateway-green)

Habilidades para agentes que implementan integraciones de pasarelas de pago en Malasia con checkout del lado del servidor, webhooks verificados y liquidación idempotente.

Desarrollado en Malasia para desarrolladores que construyen flujos de pago FPX, DuitNow QR, tarjeta, billetera electrónica y de pago local.

## Habilidades

| Habilidad | Uso para |
| --- | --- |
| `malaysia-payment-gateway` | Planificación, revisión y lista de verificación de seguridad agnóstica a la pasarela |
| `setup-chip` | Checkout CHIP Collect, compras, callbacks, reembolsos, métodos de pago |
| `setup-curlec` | API de Órdenes Curlec/Razorpay, Checkout.js, callbacks, webhooks |
| `setup-xendit` | Sesiones de Pago Xendit, Solicitudes de Pago, webhooks, xenPlatform |
| `setup-bayarcash` | API Bayarcash v2/v3, intenciones de pago, callbacks, checksums |
| `setup-bcl` | BCL Pay, Enlace de Pago, Terminal QR, Formularios, configuración vinculada a Bayarcash |
| `setup-toyyibpay` | Categorías toyyibPay, facturas, callbacks, DuitNow QR, verificación de transacciones |
| `setup-billplz` | Cobros Billplz, facturas, callbacks X Signature, pagos de Orden de Pago |
| `setup-fiuu` | Pago alojado Fiuu, Integración Sin Interrupciones, Inpage Checkout, XDK, callbacks |
| `setup-senangpay` | Checkout alojado/manual senangPay, API Directa, callbacks, consulta de estado, reembolsos, plugins, recurrente, payout, tokenización, liquidación dividida |
| `setup-hitpay` | API de Solicitud de Pago HitPay, checkout alojado, Drop-In UI, webhooks, reembolsos, plugins, facturación recurrente, payouts |
| `setup-stripe-malaysia` | Checkout Stripe Malasia, Payment Element, webhooks, Stripe CLI, MYR, tarjetas, FPX |

## Instalación

Antes de instalar:

1. Verifica la versión instalada actual.
2. Verifica que tu agente admita Skills.
3. Actualiza npm si es necesario.

Instala desde este repositorio:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway
```

La CLI puede mostrar las habilidades disponibles y permitirte seleccionarlas de forma interactiva.

Lista las habilidades disponibles:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --list
```

Instala todas las habilidades:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --all
```

Instala una habilidad:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill setup-curlec
```

Instala globalmente:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway -g
```

Verifica la versión de la CLI:

```bash
npx skills@latest --version
```

## Configuración Recomendada

1. Instala primero la habilidad de visión general:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill malaysia-payment-gateway
```

2. Instala el proveedor que necesites:

```bash
npx skills@latest add afu-it/malaysia-payment-gateway --skill setup-curlec
```

3. Pídele a tu agente que inspeccione tu proyecto antes de escribir código.

4. Pídele a tu agente que implemente checkout, verificación de webhooks, liquidación, pruebas y documentación juntos.

## Cómo Pedirle a Tu Agente

Usa el nombre de la habilidad en tu prompt:

```text
Use $malaysia-payment-gateway to review my app payment flow and tell me what is missing before production.
```

```text
Use $setup-curlec to implement Curlec checkout, callback verification, webhook settlement, and tests in this Next.js project.
```

```text
Use $setup-chip to add CHIP Collect checkout and webhook handling. Do not unlock access from redirect alone.
```

```text
Use $setup-xendit to implement hosted checkout with Xendit Payment Sessions and idempotent webhook settlement.
```

```text
Use $setup-bayarcash to integrate Bayarcash payments with checksum/callback verification.
```

```text
Use $setup-bcl to set up BCL Pay and explain which parts are BCL-owned versus Bayarcash-linked.
```

```text
Use $setup-toyyibpay to integrate toyyibPay bills, callback hash verification, and idempotent settlement.
```

```text
Use $setup-billplz to integrate Billplz bills, X Signature verification, and idempotent settlement.
```

```text
Use $setup-fiuu to integrate Fiuu hosted checkout, Notify URL verification, Callback URL handling, and idempotent settlement.
```

```text
Use $setup-senangpay to integrate senangPay hosted checkout, Callback URL verification, query status reconciliation, and idempotent settlement.
```

```text
Use $setup-hitpay to integrate HitPay Payment Request checkout, webhook signature verification, status fallback, and idempotent settlement.
```

```text
Use $setup-stripe-malaysia to integrate Stripe Malaysia checkout, webhook verification, and idempotent settlement.
```

## Reglas del Agente

Pídele al agente que:

- Lea el código existente antes de agregar nuevos archivos.
- Lea `references/account-setup.md` del proveedor antes de explicar el registro, credenciales, configuración del panel o claves API requeridas.
- Use `references/account-setup.md` del proveedor para notas de configuración de framework, variables de entorno requeridas y secretos de despliegue donde esté documentado.
- Cuando exista guía de referidos o invitaciones, preséntala como opcional y transparente; nunca implica que es obligatoria a menos que el registro del proveedor requiera realmente una invitación.
- Mantenga los secretos de la pasarela en el lado del servidor.
- Cree un pago pendiente local antes del checkout del proveedor.
- Verifique las firmas de webhooks/callbacks.
- Vuelva a obtener el estado del pago desde el proveedor cuando sea posible.
- Coincidir con el proveedor, referencia de la pasarela, monto, moneda y estado del pago local antes de marcar como pagado.
- Haga que la liquidación sea idempotente.
- Agregue pruebas para firmas inválidas, eventos duplicados, monto/moneda incorrectos y liquidación exitosa.

## Lista de Verificación de Producción

- Las variables de entorno requeridas están documentadas.
- Las migraciones de base de datos están incluidas.
- Las URLs de webhook están configuradas en los paneles sandbox y live.
- El pago sandbox tiene éxito.
- La reproducción duplicada de webhooks es inofensiva.
- Una redirección falsa de éxito no puede desbloquear acceso de pago.
- Las claves live y sandbox están separadas.

## Contribuir con una Nueva Pasarela

Las contribuciones son bienvenidas. Usa una carpeta por habilidad.

### Paso 1: Elige un nombre de habilidad

Usa nombres de estilo acción:

```text
setup-toyyibpay
setup-senangpay
setup-stripe-malaysia
```

Reglas:

- Solo minúsculas.
- Usa guiones.
- Manténlo corto.
- El nombre de la carpeta debe coincidir con `name:` en `SKILL.md`.

### Paso 2: Crea la carpeta

```bash
mkdir -p setup-yourgateway/references setup-yourgateway/agents
```

### Paso 3: Agrega `SKILL.md`

Forma mínima:

```markdown
---
name: setup-yourgateway
description: Set up, build, debug, review, and explain YourGateway payment integrations. Use when working with checkout, payment links, callbacks, webhooks, signatures, refunds, sandbox/live setup, and payment settlement for YourGateway.
---

# Setup YourGateway

Use this skill for YourGateway integration work.

## Source

Read `references/yourgateway.md` before giving factual API guidance or writing integration code.

## Core Workflow

1. Identify environment: sandbox or production.
2. Create local pending payment first.
3. Create checkout/session server-side.
4. Verify callback/webhook signature.
5. Re-fetch payment state from provider when possible.
6. Settle idempotently after amount, currency, provider, and reference match.
```

### Paso 4: Agrega `agents/openai.yaml`

```yaml
interface:
  display_name: "Setup YourGateway"
  short_description: "Set up YourGateway payments"
  default_prompt: "Use $setup-yourgateway to integrate YourGateway payments."
```

### Paso 5: Agrega referencias

Crea:

```text
setup-yourgateway/references/yourgateway.md
```

Incluye:

- URLs oficiales de la documentación.
- URLs base del entorno.
- Método de autenticación.
- Endpoint de checkout/sesión.
- Reglas de firma de webhook/callback.
- Reglas de unidades de monto.
- Notas de prueba sandbox.
- Correcciones comunes de errores.

No incluyas claves API reales, tokens, secretos de comerciante o datos privados de clientes.

### Paso 6: Valida

Ejecuta el validador:

```bash
python3 /home/user/.codex/skills/.system/skill-creator/scripts/quick_validate.py setup-yourgateway
```

Si no tienes ese validador, instala la última CLI de Skills y verifica al menos:

```bash
npx skills@latest --version
```

### Paso 7: Prueba la instalación localmente

Desde un directorio de prueba separado:

```bash
npx skills@latest add /path/to/malaysia-payment-gateway --skill setup-yourgateway
```

### Paso 8: Abre una pull request

```bash
git checkout -b add-setup-yourgateway
git add setup-yourgateway
git commit -m "Add setup-yourgateway skill"
git push origin add-setup-yourgateway
```

Luego abre una pull request con:

- Nombre de la pasarela.
- Qué productos/características están cubiertos.
- Enlaces de origen de la documentación oficial.
- Resultado de la validación.
- Cualquier brecha conocida.
