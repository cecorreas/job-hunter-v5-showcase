# Job Hunter v5 — Plataforma SaaS de Busqueda Inteligente de Empleo

<p align="center">
  <img src="assets/screenshot_vacantes.png" alt="Job Hunter v5 — Vacantes y Adaptador Curricular" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.12">
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/React-Vite-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React Vite">
  <img src="https://img.shields.io/badge/Firebase-Auth_%2B_Hosting-FFCA28?style=for-the-badge&logo=firebase&logoColor=black" alt="Firebase">
  <img src="https://img.shields.io/badge/Google_Cloud-Cloud_Run-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="GCP Cloud Run">
  <img src="https://img.shields.io/badge/Gemini_2.5-Flash-4285F4?style=for-the-badge&logo=google&logoColor=white" alt="Gemini 2.5">
  <img src="https://img.shields.io/badge/Mercado_Pago-Pagos-009EE3?style=for-the-badge&logo=mercadopago&logoColor=white" alt="Mercado Pago">
  <img src="https://img.shields.io/badge/Firestore-Database-FF6D00?style=for-the-badge&logo=firebase&logoColor=white" alt="Firestore">
  <img src="https://img.shields.io/badge/Vertex_AI-Gemini-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="Vertex AI">
  <img src="https://img.shields.io/badge/Closed--Source-Proprietary-1a1a2e?style=for-the-badge&logoColor=white" alt="Closed Source">
</p>

---

## Descripcion General

**Job Hunter v5** es una plataforma SaaS multi-usuario de busqueda automatizada de empleo con inteligencia artificial integrada. Desplegada completamente en Google Cloud Platform, permite a los usuarios cargar su curriculum, buscar vacantes en multiples portales de empleo simultaneamente, evaluar su nivel de encaje con cada oferta y adaptar su CV de forma personalizada para cada postulacion.

La plataforma opera bajo un modelo **freemium** con plan gratuito y plan Pro (pagos via Mercado Pago), evolucionando desde versiones anteriores (v3 desktop, v4 local) hacia un modelo cloud-native con autenticacion por usuario, configuracion persistente en Firestore y despliegue continuo en Cloud Run.

---

## Arquitectura del Sistema

```mermaid
graph TD
    A[React + Vite — Firebase Hosting] --> B[FastAPI — Cloud Run]
    A --> C[Firebase Auth — Google SSO]
    B --> D[Firestore — Config + Tracker por usuario]
    B --> E[Gemini 2.5 Flash — Vertex AI SDK]
    B --> F[Portales de Empleo — Backend]
    B --> P[Mercado Pago — Pagos]
    F --> G[LinkedIn]
    F --> H[Computrabajo]
    F --> I[GetOnBoard]
    F --> J[ADP Servicio Civil]
    F --> K[Empleos Publicos]
    F --> L[ChileTrabajos]
    F --> M[Laborum]
    F --> N[Trabajando.cl]
    F --> O[Vacantes Digitales]
```

---

## Modelo Freemium

| Feature | Gratis | Pro |
|---|---|---|
| Busquedas / dia | 1 | 3 |
| Portales | ADP, EmpleosPub., ChileTrabajos, Vacantes Digitales | Todos (+ LinkedIn, Computrabajo, GetOnBoard, Laborum, Trabajando) |
| Score de afinidad | No | Si |
| Filtro inteligente | No | Si |
| Tracker postulaciones | No | Si |
| PDF CV adaptado | No | Si |

Pagos procesados con **Mercado Pago** (planes 1, 3 y 6 meses).

---

## Modulos Core

### 1. Onboarding y Gestion de CV

El usuario completa un onboarding inicial ingresando nombre, area profesional y RUT (con validacion de digito verificador y bloqueo de RUT empresa). Sube su CV en formato PDF, DOCX o TXT. Gemini extrae automaticamente las palabras clave profesionales relevantes y genera un conjunto de filtros negativos (Escudo Anti-Basura) para excluir ofertas irrelevantes desde el primer momento.

El sistema incluye proteccion anti-account-sharing via `cv_guard`: al actualizar el CV, Gemini compara semanticamente el nuevo perfil con el perfil base registrado y bloquea si detecta que pertenece a una persona distinta.

<p align="center">
  <img src="assets/screenshot_cv.png" alt="Modulo Mi CV — Extraccion de Keywords con IA" width="100%">
</p>

### 2. Buscador Multi-Portal con Escudo Anti-Basura Semantico

El buscador ejecuta consultas en paralelo sobre 9 portales de empleo activos. Cada portal utiliza el metodo de extraccion mas eficiente disponible: scraping HTML, API JSON oficial o endpoint publico. Una vez recolectadas las ofertas, un filtro semantico basado en Gemini evalua la lista completa de titulos en una sola llamada, eliminando ofertas de areas completamente distintas al perfil del candidato antes de presentarlas al usuario.

<p align="center">
  <img src="assets/screenshot_buscador.png" alt="Buscador Multi-Portal — 297 vacantes encontradas" width="100%">
</p>

Caracteristicas del buscador:

- Busqueda simultanea en 9 portales: LinkedIn, Computrabajo, GetOnBoard, ADP Servicio Civil, Empleos Publicos, ChileTrabajos, Laborum, Trabajando.cl y Vacantes Digitales
- Portales con CORS bloqueado migrados a backend (Cloud Run actua como proxy sin bloqueo de IP)
- Filtro semantico batch con Gemini: una sola llamada evalua cientos de titulos contra el perfil del candidato
- Escudo Anti-Basura: combinacion de blacklist por keywords negativas + filtrado semantico por area profesional
- Deduplicacion automatica de ofertas repetidas entre portales

### 3. Vacantes y Adaptador Curricular con Smart Scorer por Portal

Las vacantes se presentan agrupadas por portal. Cada seccion tiene su propio boton de scoring independiente, permitiendo al usuario priorizar y evaluar portales de forma selectiva sin esperar la evaluacion completa de todas las ofertas.

<p align="center">
  <img src="assets/screenshot_vacantes.png" alt="Vacantes y Adaptador — Scorer por portal" width="100%">
</p>

Caracteristicas del modulo de vacantes:

- Scorer independiente por portal: evaluacion selectiva sin bloquear el resto de la interfaz
- Fit Score 0-100 por oferta con razones de encaje y brechas identificadas
- Filtro dinamico por umbral de fit score dentro de cada portal
- Adaptacion de CV con Gemini para cada oferta especifica aplicando formula X-Y-Z
- Generacion de PDF del CV adaptado con ReportLab
- Tracker de postulaciones guardado en Firestore por usuario

### 4. Pipeline de Inteligencia Artificial

Todos los modulos de IA utilizan Gemini 2.5 Flash via **Vertex AI SDK** (autenticacion por Service Account de Cloud Run, sin API key expuesta), con configuracion de `thinkingBudget: 0` para tareas estructuradas JSON, evitando latencia innecesaria de razonamiento interno.

Funciones IA implementadas:

- Extraccion de keywords profesionales desde CV
- Generacion de filtros negativos contextuales
- Filtrado semantico batch de titulos de ofertas
- Scoring de encaje CV vs oferta (score + razones + brechas)
- Adaptacion de CV personalizada por vacante con formula X-Y-Z
- Verificacion de identidad al actualizar CV (anti account-sharing)
- Generacion de CV estructurado en JSON para PDF de alta calidad

---

## Stack Tecnologico

| Capa | Tecnologia |
|---|---|
| Frontend | React 18 + Vite, dark mode personalizado |
| Backend | FastAPI (Python 3.12), Google Cloud Run |
| Autenticacion | Firebase Authentication (Google SSO) |
| Base de datos | Cloud Firestore (perfil + config + tracker por usuario) |
| Storage | Cloud Storage (CVs por usuario) |
| Hosting | Firebase Hosting (frontend), Cloud Run (backend) |
| IA | Gemini 2.5 Flash via Vertex AI SDK |
| Pagos | Mercado Pago SDK Python (planes 1/3/6 meses) |
| PDF | ReportLab (CV adaptados dinamicos) |
| DNS | Google Cloud DNS |
| Portales | LinkedIn, Computrabajo, GetOnBoard, ADP, EmpleosPub., ChileTrabajos, Laborum, Trabajando.cl, Vacantes Digitales |

---

## Estado de Licenciamiento

> [!IMPORTANT]
> **SOFTWARE PROPIETARIO — CODIGO CERRADO (Closed-Source Commercial Software)**
>
> Este repositorio contiene unicamente la presentacion de arquitectura, documentacion y capturas de pantalla del sistema Job Hunter v5 para propositos de portafolio profesional. El codigo fuente de produccion se encuentra en infraestructura privada y es distribuido bajo licencia comercial.
>
> Para consultas comerciales o demostraciones del software, puedes contactar al propietario de esta cuenta de GitHub.
