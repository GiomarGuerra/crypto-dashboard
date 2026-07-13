# 💰 Crypto Dashboard — Data App de Análisis de Criptomonedas

> Data App interactiva en **Python** y **Streamlit** que consume datos en tiempo real de la API de CoinGecko para visualizar el comportamiento del mercado de criptomonedas. **Desplegada públicamente** en Streamlit Community Cloud.

![Status](https://img.shields.io/badge/status-deployed-success)
![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Streamlit](https://img.shields.io/badge/Streamlit-Data%20App-red)
![API](https://img.shields.io/badge/API-CoinGecko-orange)

---

## 🎯 Visión General

Crypto Dashboard resuelve un problema real: las personas interesadas en el mercado de criptomonedas no cuentan con una herramienta **gratuita, clara y accesible** para analizar el comportamiento de los activos digitales en tiempo real.

La solución es un **pipeline ETL completo** que extrae datos de la API pública de CoinGecko, transforma y limpia la información, y la presenta en un dashboard interactivo con visualizaciones accionables.

**Resultado:** Un análisis centralizado del Top 50 de criptomonedas, accesible desde cualquier dispositivo sin instalación local.

---

## 🏗️ Arquitectura del Sistema

```
┌──────────────┐     ┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│  API REST    │     │     ETL      │     │  Almacenamiento │     │ Visualización│
│ (CoinGecko)  │ --> │   (Python)   │ --> │    (CSV/Cache)  │ --> │  (Streamlit) │
└──────────────┘     └──────────────┘     └─────────────────┘     └──────────────┘
      ↓                      ↓                      ↓                      ↓
 50 criptosActualizadas    Pandas          Limpieza de nulos      Dashboard interactivo
 en tiempo real           requests         Renombrado de cols     KPIs + Gráficos + Filtros
```

| Módulo | Función | Tecnología |
|--------|---------|------------|
| **Ingestión** | Consume `/coins/markets` de CoinGecko | `requests` (HTTP) |
| **Transformación** | Selección de columnas, limpieza y wrangling | `Pandas` (Data Wrangling) |
| **Almacenamiento** | Persistencia local + caché de datos | CSV + `st.cache_data` |
| **Visualización** | Dashboard interactivo y KPIs | `Streamlit` |
| **Despliegue** | Aplicación pública y continua | Streamlit Community Cloud |

---

## 📊 Características Principales

- ✅ **Datos en tiempo real:** Consume API de CoinGecko cada sesión (Top 50 criptomonedas)
- ✅ **KPIs ejecutivos:** Capitalización total, cambio promedio 24h, crypto de mayor alza/caída
- ✅ **Visualizaciones:** Gráficos de barras interactivos (capitalización vs. variación 24h)
- ✅ **Filtro dinámico:** Análisis detallado por criptomoneda individual
- ✅ **Tabla detallada:** Datos completos con formato condicional (color por ganancia/pérdida)
- ✅ **Responsive:** Funciona en móvil, tablet y desktop
- ✅ **Despliegue automático:** GitHub → Streamlit Cloud (CI/CD)

---

## 🗂️ Estructura del Proyecto

```
crypto-dashboard/
├── app/
│   └── app.py              # Aplicación principal (Streamlit UI)
├── src/
│   ├── ingest.py           # Módulo de ingesta (extrae de CoinGecko API)
│   └── transform.py        # Módulo de transformación (Data Wrangling)
├── data/
│   └── crypto.csv          # Dataset limpio (generado en runtime)
├── docs/
│   ├── Dashboard_Criptomonedas.pdf                       
│   ├── Informe Final LENGUAJE DE CIENCIA DE DATOS II.md       
│   └── Informe Final LENGUAJE DE CIENCIA DE DATOS II.pdf      
├── .devcontainer/          # Configuración para desarrollo en contenedor
├── .streamlit/
│   └── secrets.toml        # Credenciales seguras (no versionado)
├── requirements.txt        # Dependencias del proyecto
├── .gitignore              # Archivos excluidos de Git
└── README.md               # Informacion general de todo el proyecto
```

---

## 🚀 Cómo Ejecutar Localmente

### Requisitos Previos
- Python 3.10+
- Conda (para manejo de entorno aislado)
- Git

### Pasos de Instalación

**1. Clonar el repositorio:**
```bash
git clone https://github.com/GiomarGuerra/crypto-dashboard.git
cd crypto-dashboard
```

**2. Crear y activar el entorno Conda:**
```bash
conda create -n crypto_env python=3.10 -y
conda activate crypto_env
```

**3. Instalar dependencias:**
```bash
pip install -r requirements.txt
```

**4. Crear archivo de secretos (opcional para local):**
```bash
mkdir -p .streamlit
notepad .streamlit/secrets.toml
```

**5. Ejecutar la aplicación:**
```bash
streamlit run app/app.py
```

La app se abre en `http://localhost:8501`

---

## 🌐 Acceso Online (Deployment)

La aplicación está **desplegada públicamente en Streamlit Community Cloud:**

🔗 **[Crypto Dashboard Live](https://crypto-dashboard-live.streamlit.app/)** *(URL del deployment real)*

- **Sin instalación requerida**
- Datos actualizados en tiempo real
- Accesible desde cualquier navegador

---

## 📋 Flujo de Datos (ETL)

### Fase 1: Extracción (ingest.py)
```python
# Consume el endpoint /coins/markets de CoinGecko
# Parámetros:
#   - vs_currency: USD
#   - order: market_cap_desc (orden descendente)
#   - per_page: 50 (Top 50)

import requests
response = requests.get("https://api.coingecko.com/api/v3/coins/markets", params={...})
data = response.json()
```

**Resultado:** JSON con ~15 columnas, 50 filas (criptos)

### Fase 2: Transformación (transform.py)
```python
# Data Wrangling con Pandas:
# - Selecciona: name, current_price, market_cap, price_change_percentage_24h
# - Renombra a español: crypto, precio, capitalización, cambio_24h
# - Elimina filas con nulos
# - Formatea tipos de datos

df = pd.read_json(data)
df = df[['name', 'current_price', 'market_cap', 'price_change_percentage_24h']]
df.columns = ['crypto', 'precio', 'capitalización', 'cambio_24h']
df = df.dropna()
df.to_csv('data/crypto.csv', index=False)
```

**Resultado:** CSV limpio, 50 filas, 4 columnas

### Fase 3: Carga & Visualización (app.py)
```python
# Streamlit UI
import streamlit as st
import pandas as pd

df = pd.read_csv('data/crypto.csv')

st.title("💰 Crypto Dashboard")
st.dataframe(df)  # Tabla
st.bar_chart(df.set_index('crypto')['precio'])  # Gráfico
st.metric("Market Cap Total", f"${df['capitalización'].sum():,.0f}")  # KPI
```

**Resultado:** Dashboard interactivo publicado en Streamlit Community Cloud

---

## 🛠️ Stack Tecnológico

| Componente | Herramienta | Propósito |
|-----------|-----------|----------|
| **API Consumption** | `requests` | Llamadas HTTP a CoinGecko |
| **Data Processing** | `Pandas` | Transformación y limpieza |
| **Web Framework** | `Streamlit` | Interfaz interactiva sin HTML/CSS |
| **Caching** | `st.cache_data` | Actualización eficiente de datos |
| **Environment** | `Conda` | Aislamiento de dependencias |
| **Version Control** | `Git` + GitHub | Repositorio y CI/CD |
| **Deployment** | Streamlit Community Cloud | Hosting gratuito |

---

## 📈 Resultados & Impacto

- **Usuarios:** Acceso público sin restricciones (miles de visitantes potenciales)
- **Tiempo de análisis:** Análisis de 50 criptos en <2 segundos
- **Disponibilidad:** 24/7 (actualización cada sesión de usuario)
- **Replicabilidad:** Cualquiera puede clonar, ejecutar y customizar en <10 minutos
- **Escalabilidad:** Arquitectura modular permite agregar más fuentes de datos

---

## 🎓 Aprendizajes Técnicos

Este proyecto integra en la práctica:

1. **Consumo de APIs REST** — Manejo de JSON, parámetros y errores HTTP
2. **Data Wrangling con Pandas** — Limpieza, transformación y estructuración
3. **Desarrollo web sin HTML/CSS** — Streamlit para ciencia de datos
4. **Gestión de entornos** — Conda para reproducibilidad
5. **CI/CD y Deployment** — GitHub Actions → Streamlit Cloud
6. **Visualización interactiva** — Gráficos, filtros y KPIs dinámicos

---

## 📝 Documentación Completa

La documentación académica completa (14 páginas) está disponible en `docs/Proyecto_Crypto_Dashboard.pdf`:
- Justificación detallada del problema
- Objetivos SMART
- Plan de trabajo con hitos
- Conclusiones técnicas
- Recomendaciones para próximas fases

---

## 🤝 Colaboradores

| Rol | Nombre |
|-----|--------|
| **Data Engineer / Backend** | Giomar Guerra (i202400989) |
| **Data Analyst** | Elizabeth Cordova |
| **Frontend/UX** | Miguel Castillo |
| **QA/Testing** | Thania Colan |
| **DevOps/Deploy** | Yenny Salas |

---

## 📞 Contacto & Links

- **GitHub:** [GiomarGuerra/crypto-dashboard](https://github.com/GiomarGuerra/crypto-dashboard)
- **LinkedIn:** [data-giomar](https://www.linkedin.com/in/data-giomar)
- **Email:** giomarguerra0@gmail.com

---

## 📄 Licencia

MIT License — Libre para usar, modificar y distribuir con atribución.

---

## 🔮 Roadmap — Mejoras Futuras

- [ ] Gráficos de velas (OHLCV) con datos históricos
- [ ] Análisis de correlación entre activos (heatmap)
- [ ] Migración a base de datos SQLite/PostgreSQL
- [ ] Alertas de variación de precio (Telegram Bot)
- [ ] Dark mode y customización de tema
- [ ] Multi-lenguaje (ES/EN/PT)
- [ ] Exportación de reportes (PDF)

---

**Construido con ❤️ como proyecto de carrera en Arquitectura de Datos Empresariales — Cibertec 2026**

