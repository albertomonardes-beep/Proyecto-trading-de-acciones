# Proyecto Trading de Acciones

## Descripcion

Sistema de recopilacion, almacenamiento y visualizacion de datos de precios semanales de acciones y ETFs del mercado americano, como base para un modelo de trading con lista de seguimiento semanal.

---

## Objetivo

Filtrar ~9.000 instrumentos cada semana y generar una watchlist de 4-5 acciones accionables, basada en condiciones de mercado y criterios individuales por instrumento.

---

## Decisiones tomadas (contexto para continuar)

### Base de datos
- Se eligio **MongoDB Atlas (M0 free)** en lugar de Supabase/PostgreSQL
- Motivos: el usuario ya tenia cuenta, MongoDB no pausa proyectos por inactividad (Supabase pausa tras 7 dias sin actividad, lo que es un problema para un pipeline semanal)
- Cluster existente del usuario reutilizado (ya tenia M0 con 116 MB usados)
- Base de datos creada: `trading`

### Modelo de datos
- **No se acumula historial**. Solo se guarda la "foto" de la semana actual (upsert semanal)
- Esto mantiene el tamano de la base de datos estatico (~5-10 MB total)

### Brokers
- Se agregan dos campos booleanos en `tickers`: `capital_com` y `pepperstone`
- Indican si el instrumento se puede tradear en cada broker
- **Capital.com**: actualizacion mensual automatica via su API (el usuario tiene cuenta)
- **Pepperstone**: actualizacion mensual automatica via web scraping (sin API publica)
- Ambas actualizaciones son 100% automaticas, sin intervencion manual

---

## Stack tecnologico

| Componente | Tecnologia | Costo |
|---|---|---|
| Fuente de datos | yfinance + NASDAQ CSV | Gratuito |
| Base de datos | MongoDB Atlas M0 | Gratuito |
| Scheduler | GitHub Actions | Gratuito |
| Interfaz | Streamlit Community Cloud | Gratuito |

---

## Repositorio de codigo

El codigo del proyecto esta en: https://github.com/albertomonardes-beep/trading-pipeline

---

## Arquitectura MongoDB

### Coleccion `tickers`
```json
{
  "ticker": "AAPL",
  "name": "Apple Inc.",
  "exchange": "NASDAQ",
  "capital_com": true,
  "pepperstone": false
}
```

### Coleccion `weekly_prices`
Foto semanal OHLCV. Se sobreescribe cada semana.
```json
{
  "ticker": "AAPL",
  "open": 189.50,
  "high": 192.30,
  "low": 187.10,
  "close": 191.20,
  "volume": 58000000
}
```

### Coleccion `market_data`
Indicadores macro: indices, volatilidad, ETFs de sectores.

---

## Logica de filtrado (2 niveles)

1. **Nivel mercado**: evalua condiciones generales del mercado
2. **Nivel instrumento**: filtra activos individuales dentro de mercados favorables

Resultado: watchlist semanal de 4-5 acciones para revisar

---

## Estado del proyecto

- [x] Repositorio `trading-pipeline` creado en GitHub
- [x] Dependencias instaladas (yfinance, pymongo, pandas, requests, beautifulsoup4)
- [x] Conexion a MongoDB Atlas configurada
- [x] Colecciones creadas (tickers, weekly_prices, market_data)
- [x] Secreto `MONGODB_URI` guardado en GitHub Actions
- [ ] Script de carga de tickers desde NASDAQ CSV
- [ ] Script de descarga de precios semanales
- [ ] Script de datos de mercado
- [ ] Filtros de watchlist (criterios por definir)
- [ ] GitHub Actions scheduler semanal
- [ ] Interfaz Streamlit
- [ ] Integracion Capital.com API (mensual)
- [ ] Integracion Pepperstone scraping (mensual)

---

## Informacion pendiente de definir

- Campos especificos que se necesitan para cada activo (OHLCV + que mas?)
- Campos especificos para market_data
- Criterios de los 2 niveles de filtrado
