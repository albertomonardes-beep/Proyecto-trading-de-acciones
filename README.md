# Proyecto Trading de Acciones

## Descripción

Sistema de recopilación, almacenamiento y visualización de datos de precios semanales de acciones y ETFs del mercado americano, como base para un modelo de trading con lista de seguimiento semanal.

---

## Objetivo

Construir un pipeline de datos que:
1. Descargue semanalmente precios de acciones y ETFs del mercado americano
2. Los almacene en una base de datos cloud accesible desde cualquier lugar
3. Permita aplicar dos niveles de filtros para generar una lista de seguimiento semanal

---

## Arquitectura del Sistema

```
GitHub Actions (cron semanal)
        ↓
Python + yfinance (descarga de datos)
        ↓
Supabase PostgreSQL (base de datos, gratis)
        ↓
Streamlit Community Cloud (interfaz web, gratis)
```

---

## Stack Tecnológico

| Componente         | Tecnología                 | Costo  |
|--------------------|----------------------------|--------|
| Fuente de datos    | yfinance                   | Gratis |
| Lista de tickers   | CSV NASDAQ                 | Gratis |
| Base de datos      | Supabase (PostgreSQL)      | Gratis |
| Scheduler          | GitHub Actions (cron)      | Gratis |
| Interfaz web       | Streamlit Community Cloud  | Gratis |
| **Total**          |                            | **$0** |

---

## Universo de Instrumentos

- Acciones: ~5.000–6.000 tickers (NYSE, NASDAQ, AMEX)
- ETFs: ~3.000 tickers
- Total estimado: ~8.000–9.000 instrumentos
- Fuente de la lista: CSV público de NASDAQ (incluye campo ETF: Y/N)
- Actualización: semanal (para capturar listings y delistings)

---

## Estructura de la Base de Datos

### Tabla 1: `tickers`
Universo de instrumentos disponibles.

```sql
CREATE TABLE tickers (
    symbol        VARCHAR(10) PRIMARY KEY,
    name          VARCHAR(255),
    exchange      VARCHAR(20),
    sector        VARCHAR(100),
    type          VARCHAR(10),   -- 'stock' o 'etf'
    is_active     BOOLEAN,
    listed_date   DATE
);
```

### Tabla 2: `weekly_prices`
Precios OHLCV semanales ajustados.

```sql
CREATE TABLE weekly_prices (
    symbol    VARCHAR(10),
    date      DATE,
    open      NUMERIC,
    high      NUMERIC,
    low       NUMERIC,
    close     NUMERIC,
    volume    BIGINT,
    adj_close NUMERIC,
    PRIMARY KEY (symbol, date)
);
```

### Tabla 3: `market_data` (por definir)
Indicadores de mercado para el primer nivel de filtro.
- Índices principales: SPY, QQQ, IWM
- Volatilidad: VIX
- ETFs sectoriales: XLK, XLF, XLE, etc.
- Métricas de amplitud del mercado

---

## Lógica de Filtrado (dos niveles)

### Primer filtro — Datos de mercado
Evalúa el contexto macro y sectorial del mercado.
Define si las condiciones generales son favorables para operar.

### Segundo filtro — Datos de acciones
Sobre el universo que pasa el primer filtro, aplica criterios por instrumento individual.
Output: **lista de seguimiento semanal**

---

## Interfaz Web (Streamlit)

Dos vistas principales:

| Vista              | Contenido                                      |
|--------------------|------------------------------------------------|
| Pestaña 1: Mercado | Indicadores macro, filtros por sector y fecha  |
| Pestaña 2: Acciones| OHLCV + métricas, filtrado desde vista 1       |

Sin gráficos. Solo tablas con filtros. Accesible online sin instalar nada.

---

## Orden de Implementación

1. Script que descarga y carga la lista de tickers (acciones + ETFs) en Supabase
2. Script que descarga precios históricos semanales con yfinance
3. GitHub Action que ejecuta el pipeline cada semana
4. App Streamlit básica para visualizar y filtrar los datos

---

## Volumen Estimado

- 3.000 tickers × 5 años de historia semanal ≈ 780.000 filas
- Tamaño estimado en PostgreSQL: ~80–120 MB
- Entra cómodo en el free tier de Supabase (500 MB)

---

## Estado del Proyecto

- [ ] Definir schema final de base de datos
- [ ] Script de descarga de lista de tickers
- [ ] Script de descarga de precios históricos
- [ ] Configurar Supabase
- [ ] GitHub Action (scheduler semanal)
- [ ] App Streamlit (interfaz web)
- [ ] Definir criterios de filtros de mercado
- [ ] Definir criterios de filtros de acciones
