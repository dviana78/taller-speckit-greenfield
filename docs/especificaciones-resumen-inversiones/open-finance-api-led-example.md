# Open Finance --- APIs de resumen de inversiones

> **Documento de ejemplo** para ilustrar una arquitectura API-Led con
> contratos OpenAPI y modelos canónicos. Los métodos, campos y reglas
> son ficticios y deben validarse con el catálogo de capacidades, las
> reglas de negocio y los estándares de la organización antes de
> utilizarlos como contratos reales.

## 1. Objetivo y alcance

El ejemplo modela cómo un canal digital consulta un resumen de
inversiones mediante tres capas API-Led:

  ---------------------------------------------------------------------------------------
  Capa                    API                                     Responsabilidad
  ----------------------- --------------------------------------- -----------------------
  Experience              `of-exp-port-investment-dashboard-v1`   Presentar el resumen de
                                                                  inversiones adaptado a
                                                                  un canal digital.

  Process                 `of-prc-port-investment-summary-v1`     Coordinar y componer la
                                                                  información necesaria
                                                                  para el resumen.

  System                  `of-sys-port-portfolio-v1`              Exponer capacidades del
                                                                  sistema de carteras.

  System                  `of-sys-acct-core-banking-v1`           Exponer capacidades del
                                                                  sistema de cuentas
                                                                  bancarias.
  ---------------------------------------------------------------------------------------

### Flujo de consulta

1.  El canal invoca la Experience API.
2.  La Experience API llama a la Process API.
3.  La Process API consulta la System API de carteras y, cuando sea
    necesario, la System API de cuentas.
4.  La Process API compone un modelo de resumen canónico.
5.  La Experience API devuelve la representación adecuada al canal.

La Experience API no debería duplicar la lógica de orquestación de la
Process API. Las System APIs aíslan las particularidades de los sistemas
de backend.

## 2. Convenciones del ejemplo

-   Prefijo `of`: Open Finance.
-   `exp`, `prc`, `sys`: Experience, Process y System.
-   `port`: dominio de carteras/inversiones.
-   `acct`: dominio de cuentas.
-   `v1` en el identificador lógico representa la versión mayor del
    contrato en este ejemplo. La estrategia exacta de versionado debe
    fijarse como estándar de plataforma.
-   Se utilizan recursos REST y JSON como propuesta de diseño, no como
    requisito de Open Finance.
-   Los identificadores de cliente, cartera y cuenta son identificadores
    de ejemplo. No deben incluir datos personales ni secretos.

## 3. Experience API: `of-exp-port-investment-dashboard-v1`

**Propósito:** ofrecer al canal digital una vista orientada a la
experiencia de usuario. Puede resumir o presentar datos, pero no debe
ser la propietaria de la orquestación de negocio.

Base path ilustrativo: `/open-finance/experience/v1`

  ------------------------------------------------------------------------------------------------------------------
  Método            Recurso                                                      Descripción       Respuestas
                                                                                                   principales
  ----------------- ------------------------------------------------------------ ----------------- -----------------
  `GET`             `/customers/{customerId}/investment-dashboard`               Obtiene el panel  `200`, `400`,
                                                                                 de inversiones de `401`, `403`,
                                                                                 un cliente.       `404`, `429`,
                                                                                                   `500`, `503`

  `GET`             `/customers/{customerId}/investment-dashboard/performance`   Obtiene una vista `200`, `400`,
                                                                                 resumida de       `401`, `403`,
                                                                                 rendimiento para  `404`, `429`,
                                                                                 un periodo.       `500`, `503`

  `GET`             `/customers/{customerId}/investment-dashboard/positions`     Obtiene las       `200`, `400`,
                                                                                 posiciones        `401`, `403`,
                                                                                 agregadas que se  `404`, `429`,
                                                                                 muestran en el    `500`, `503`
                                                                                 panel.            
  ------------------------------------------------------------------------------------------------------------------

### Ejemplo de respuesta: `GET /customers/{customerId}/investment-dashboard`

``` json
{
  "customerId": "cus-100045",
  "asOf": "2026-10-31T10:30:00Z",
  "currency": "EUR",
  "totalMarketValue": 125430.75,
  "totalCostBasis": 117000.00,
  "unrealizedGainLoss": 8430.75,
  "unrealizedGainLossPercentage": 7.21,
  "portfolioCount": 2,
  "portfolios": [
    {
      "portfolioId": "prt-20001",
      "name": "Cartera equilibrada",
      "marketValue": 80430.75,
      "currency": "EUR",
      "unrealizedGainLoss": 5430.75,
      "unrealizedGainLossPercentage": 7.24
    },
    {
      "portfolioId": "prt-20002",
      "name": "Cartera conservadora",
      "marketValue": 45000.00,
      "currency": "EUR",
      "unrealizedGainLoss": 3000.00,
      "unrealizedGainLossPercentage": 7.14
    }
  ]
}
```

Este payload es un ejemplo de representación para el canal. La
estructura podría diferir del modelo de dominio interno si las
necesidades de experiencia lo justifican.

## 4. Process API: `of-prc-port-investment-summary-v1`

**Propósito:** coordinar las fuentes necesarias y aplicar las reglas de
composición del resumen de inversiones.

Base path ilustrativo: `/open-finance/process/v1`

  ----------------------------------------------------------------------------------------------------------------
  Método            Recurso                                                    Descripción       Respuestas
                                                                                                 principales
  ----------------- ---------------------------------------------------------- ----------------- -----------------
  `GET`             `/customers/{customerId}/investment-summary`               Compone el        `200`, `400`,
                                                                               resumen de        `401`, `403`,
                                                                               inversión del     `404`, `429`,
                                                                               cliente.          `500`, `503`

  `GET`             `/customers/{customerId}/investment-summary/performance`   Compone las       `200`, `400`,
                                                                               métricas de       `401`, `403`,
                                                                               rendimiento para  `404`, `429`,
                                                                               un periodo.       `500`, `503`

  `GET`             `/customers/{customerId}/investment-summary/positions`     Obtiene y agrega  `200`, `400`,
                                                                               las posiciones de `401`, `403`,
                                                                               las carteras del  `404`, `429`,
                                                                               cliente.          `500`, `503`
  ----------------------------------------------------------------------------------------------------------------

### Parámetros sugeridos

  ----------------------------------------------------------------------------
  Parámetro      Ubicación      Tipo           Obligatorio    Descripción
  -------------- -------------- -------------- -------------- ----------------
  `customerId`   Path           string         Sí             Identificador
                                                              del cliente en
                                                              el dominio Open
                                                              Finance.

  `from`         Query          date           No             Inicio del
                                                              periodo de
                                                              consulta,
                                                              formato
                                                              `YYYY-MM-DD`.

  `to`           Query          date           No             Fin del periodo
                                                              de consulta,
                                                              formato
                                                              `YYYY-MM-DD`.

  `currency`     Query          string         No             Código de moneda
                                                              ISO 4217 para la
                                                              representación
                                                              solicitada.

  `page`         Query          integer        No             Página para
                                                              colecciones
                                                              paginadas, si
                                                              procede.

  `pageSize`     Query          integer        No             Número máximo de
                                                              elementos por
                                                              página, sujeto a
                                                              límites de
                                                              plataforma.
  ----------------------------------------------------------------------------

### Composición propuesta

Para `GET /customers/{customerId}/investment-summary`, la Process API
podría:

1.  Resolver las carteras asociadas al cliente mediante
    `of-sys-port-portfolio-v1`.
2.  Obtener valoraciones y posiciones de esas carteras.
3.  Consultar datos de cuentas en `of-sys-acct-core-banking-v1`
    únicamente si son necesarios para el resumen definido.
4.  Normalizar las monedas y aplicar las reglas de agregación acordadas.
5.  Devolver el modelo canónico `InvestmentSummary`.

Las conversiones de divisa, la fecha de valoración, el tratamiento de
posiciones sin precio y el redondeo deben quedar definidos
explícitamente como reglas de negocio. No se deben sumar importes de
monedas distintas sin conversión y sin indicar la fuente/tipo de cambio
utilizado.

## 5. System API: `of-sys-port-portfolio-v1`

**Propósito:** exponer las capacidades del sistema de carteras sin
trasladar sus detalles técnicos al consumidor.

Base path ilustrativo: `/open-finance/system/portfolios/v1`

  ---------------------------------------------------------------------------------------------
  Método            Recurso                                 Descripción       Respuestas
                                                                              principales
  ----------------- --------------------------------------- ----------------- -----------------
  `GET`             `/customers/{customerId}/portfolios`    Lista las         `200`, `400`,
                                                            carteras de un    `401`, `403`,
                                                            cliente.          `404`, `429`,
                                                                              `500`, `503`

  `GET`             `/portfolios/{portfolioId}`             Obtiene los datos `200`, `401`,
                                                            principales de    `403`, `404`,
                                                            una cartera.      `429`, `500`,
                                                                              `503`

  `GET`             `/portfolios/{portfolioId}/positions`   Obtiene las       `200`, `400`,
                                                            posiciones de una `401`, `403`,
                                                            cartera.          `404`, `429`,
                                                                              `500`, `503`

  `GET`             `/portfolios/{portfolioId}/valuation`   Obtiene la        `200`, `400`,
                                                            valoración de una `401`, `403`,
                                                            cartera en una    `404`, `429`,
                                                            fecha.            `500`, `503`
  ---------------------------------------------------------------------------------------------

Parámetros opcionales para posiciones y valoraciones podrían incluir
`asOf`, `page`, `pageSize` y `sort`. Deben definirse de forma coherente
en el contrato y en los estándares comunes.

## 6. System API: `of-sys-acct-core-banking-v1`

**Propósito:** exponer capacidades de consulta de cuentas del sistema
bancario. El ejemplo se limita a operaciones de lectura.

Base path ilustrativo: `/open-finance/system/accounts/v1`

  ------------------------------------------------------------------------------------------
  Método            Recurso                              Descripción       Respuestas
                                                                           principales
  ----------------- ------------------------------------ ----------------- -----------------
  `GET`             `/customers/{customerId}/accounts`   Lista las cuentas `200`, `400`,
                                                         del cliente que   `401`, `403`,
                                                         el consumidor     `404`, `429`,
                                                         está autorizado a `500`, `503`
                                                         consultar.        

  `GET`             `/accounts/{accountId}`              Obtiene los datos `200`, `401`,
                                                         principales de    `403`, `404`,
                                                         una cuenta.       `429`, `500`,
                                                                           `503`

  `GET`             `/accounts/{accountId}/balances`     Obtiene los       `200`, `401`,
                                                         saldos            `403`, `404`,
                                                         disponibles y     `429`, `500`,
                                                         contables.        `503`
  ------------------------------------------------------------------------------------------

El acceso debe comprobar autorización sobre el recurso y el cliente
correspondiente; conocer un `accountId` no debe conceder acceso por sí
mismo.

## 7. Modelos canónicos de ejemplo

Se proponen estos modelos reutilizables. Son ejemplos de diseño y no
sustituyen un modelo corporativo ya aprobado.

### 7.1 `Money`

Representa un importe monetario con moneda explícita.

``` json
{
  "amount": 125430.75,
  "currency": "EUR"
}
```

Campos: - `amount`: número decimal; definir precisión y escala en el
estándar de datos. - `currency`: código ISO 4217, por ejemplo `EUR`.

No usar `float` binario como tipo de cálculo financiero en la
implementación; establecer una representación decimal apropiada.

### 7.2 `CustomerReference`

Referencia mínima a un cliente, sin replicar todos sus datos personales.

``` json
{
  "customerId": "cus-100045"
}
```

Campos: - `customerId`: identificador de cliente dentro del dominio
definido.

### 7.3 `Portfolio`

``` json
{
  "portfolioId": "prt-20001",
  "customerId": "cus-100045",
  "name": "Cartera equilibrada",
  "status": "ACTIVE",
  "baseCurrency": "EUR",
  "openedOn": "2022-04-12"
}
```

Campos sugeridos: - `portfolioId`: identificador único de cartera. -
`customerId`: referencia al titular o cliente relacionado. - `name`:
nombre de presentación, si está disponible. - `status`: estado
controlado, por ejemplo `ACTIVE`, `INACTIVE` o `CLOSED`, sujeto al
catálogo de negocio. - `baseCurrency`: moneda base de la cartera. -
`openedOn`: fecha de apertura, si el sistema fuente la proporciona.

### 7.4 `InvestmentPosition`

``` json
{
  "positionId": "pos-30001",
  "portfolioId": "prt-20001",
  "instrumentId": "ins-40001",
  "instrumentName": "Fondo de inversión de ejemplo",
  "quantity": 125.50,
  "marketPrice": {
    "amount": 642.00,
    "currency": "EUR"
  },
  "marketValue": {
    "amount": 80571.00,
    "currency": "EUR"
  },
  "valuationDate": "2026-10-30"
}
```

Campos sugeridos: - `positionId`: identificador de la posición. -
`portfolioId`: cartera a la que pertenece. - `instrumentId`: referencia
al instrumento financiero. - `instrumentName`: nombre de presentación
opcional. - `quantity`: cantidad mantenida; definir la precisión según
el instrumento. - `marketPrice`: precio unitario y moneda. -
`marketValue`: valor de mercado y moneda. - `valuationDate`: fecha de
valoración.

El ejemplo no modela todos los atributos regulatorios o de instrumentos
financieros. Estos deberán añadirse según el alcance y las normas
aplicables.

### 7.5 `PortfolioValuation`

``` json
{
  "portfolioId": "prt-20001",
  "asOf": "2026-10-30T18:00:00Z",
  "currency": "EUR",
  "marketValue": 80430.75,
  "costBasis": 75000.00,
  "unrealizedGainLoss": 5430.75,
  "unrealizedGainLossPercentage": 7.24
}
```

Campos sugeridos: - `portfolioId`: cartera valorada. - `asOf`: instante
de cálculo o corte de la valoración. - `currency`: moneda en la que se
expresan los importes. - `marketValue`: valor de mercado total. -
`costBasis`: coste base utilizado en el cálculo. - `unrealizedGainLoss`:
diferencia entre valor de mercado y coste base. -
`unrealizedGainLossPercentage`: porcentaje calculado según una fórmula
acordada.

Definir si `costBasis` incluye comisiones, impuestos y otros ajustes, y
qué sucede cuando no está disponible.

### 7.6 `AccountReference` y `AccountBalance`

``` json
{
  "accountId": "acc-50001",
  "accountType": "CURRENT",
  "currency": "EUR",
  "status": "ACTIVE"
}
```

``` json
{
  "accountId": "acc-50001",
  "asOf": "2026-10-31T10:30:00Z",
  "currency": "EUR",
  "availableBalance": 5400.25,
  "bookBalance": 5650.25
}
```

Campos sugeridos: - `accountId`: identificador de cuenta. -
`accountType`: tipo de cuenta de un catálogo controlado. - `currency`:
moneda de la cuenta. - `status`: estado de la cuenta. - `asOf`:
fecha/hora de corte de los saldos. - `availableBalance`: saldo
disponible. - `bookBalance`: saldo contable.

Los conceptos de saldo deben definirse con precisión de negocio, porque
su significado puede variar según el sistema bancario.

### 7.7 `InvestmentSummary`

Modelo compuesto que podría utilizar la Process API y que la Experience
API adaptaría a las necesidades del canal.

``` json
{
  "customerId": "cus-100045",
  "asOf": "2026-10-31T10:30:00Z",
  "currency": "EUR",
  "totalMarketValue": 125430.75,
  "totalCostBasis": 117000.00,
  "unrealizedGainLoss": 8430.75,
  "unrealizedGainLossPercentage": 7.21,
  "portfolios": [
    {
      "portfolioId": "prt-20001",
      "marketValue": 80430.75,
      "currency": "EUR"
    },
    {
      "portfolioId": "prt-20002",
      "marketValue": 45000.00,
      "currency": "EUR"
    }
  ]
}
```

Campos sugeridos: - `customerId`: cliente consultado. - `asOf`: instante
de referencia del resumen. - `currency`: moneda de presentación de los
totales. - `totalMarketValue`: valor de mercado agregado. -
`totalCostBasis`: coste base agregado. - `unrealizedGainLoss`: ganancia
o pérdida no realizada. - `unrealizedGainLossPercentage`: porcentaje
según fórmula corporativa. - `portfolios`: carteras incluidas en el
resumen.

Añadir metadatos de conversión de divisa cuando se mezclen monedas, por
ejemplo fecha, proveedor y tipo de cambio aplicado, si corresponde al
negocio.

## 8. Componentes comunes de contrato

Se recomienda mantener componentes compartidos en un
proyecto/repositorio de contratos comunes, por ejemplo:

-   `Problem`: formato estándar de error basado en Problem Details.
-   `Correlation-Id`: identificador de correlación para trazabilidad
    entre APIs.
-   `Page` / `Pagination`: metadatos de paginación.
-   `Money`: si se aprueba como componente canónico corporativo.
-   Parámetros comunes de fecha, ordenación y paginación.
-   Esquemas comunes de errores y respuestas estándar.

Los modelos canónicos de negocio, como `Portfolio` o
`InvestmentPosition`, pueden vivir en un repositorio independiente de
modelos corporativos (`goldenRecords` o equivalente). Las APIs deben
consumir versiones aprobadas y fijadas; no conviene que una modificación
de un modelo compartido cambie automáticamente todos los contratos
consumidores.

## 9. Errores y comportamiento HTTP

Utilizar un esquema de error común y documentarlo en OpenAPI.

  -----------------------------------------------------------------------
  Código                              Uso sugerido
  ----------------------------------- -----------------------------------
  `400 Bad Request`                   Parámetros inválidos o combinación
                                      de filtros no permitida.

  `401 Unauthorized`                  Falta autenticación válida.

  `403 Forbidden`                     El consumidor está autenticado,
                                      pero no tiene autorización.

  `404 Not Found`                     Recurso inexistente o no visible
                                      para el consumidor, conforme a la
                                      política de seguridad.

  `409 Conflict`                      Conflicto con el estado del
                                      recurso, cuando aplique.

  `429 Too Many Requests`             Límite de consumo excedido.

  `500 Internal Server Error`         Error inesperado no clasificado.

  `502 Bad Gateway`                   Respuesta inválida o fallo de
                                      comunicación con una dependencia,
                                      si la política de plataforma lo
                                      define así.

  `503 Service Unavailable`           Servicio temporalmente no
                                      disponible.

  `504 Gateway Timeout`               Una dependencia no responde dentro
                                      del tiempo límite.
  -----------------------------------------------------------------------

Ejemplo de error:

``` json
{
  "type": "https://api.example.com/problems/resource-not-found",
  "title": "Resource not found",
  "status": 404,
  "detail": "The requested portfolio was not found.",
  "instance": "/portfolios/prt-20001",
  "correlationId": "8f61f53d-58e1-4f85-b20f-83b0bffcc9c1"
}
```

El dominio `api.example.com` es ilustrativo; debe sustituirse por el
dominio documental oficial de la organización.

## 10. Seguridad, privacidad y gobierno

-   Definir los mecanismos de autenticación y autorización según los
    estándares de la plataforma; OAuth 2.0/OIDC son opciones habituales,
    pero los flujos y perfiles concretos deben acordarse.
-   Aplicar autorización por cliente, cartera y cuenta, no solo por
    endpoint.
-   Minimizar los datos personales devueltos por las APIs.
-   No registrar tokens, credenciales, datos bancarios sensibles ni
    payloads completos por defecto.
-   Propagar un identificador de correlación sin usarlo como mecanismo
    de autenticación.
-   Definir límites de consumo, timeouts, reintentos y circuit breakers
    para llamadas entre APIs.
-   Documentar clasificación de datos, retención, auditoría y requisitos
    regulatorios aplicables.
-   Para APIs públicas o reguladas de Open Finance, validar los perfiles
    de seguridad, consentimiento, scopes y requisitos regulatorios
    específicos antes de publicar.

## 11. Estructura sugerida del repositorio de cada API

``` text
of-prc-port-investment-summary-v1/
├── design/
│   ├── commons/
│   ├── goldenRecords/
│   ├── parameters/
│   ├── resources/
│   ├── rules/
│   ├── schemas/
│   └── of-prc-port-investment-summary-v1.yaml
├── validation/
│   └── .spectral.yaml
├── package.json
├── package-lock.json
├── redocly.yaml
└── README.md
```

El repositorio almacena el **contrato y sus componentes**, no la
implementación de la API. El pipeline CI de Jenkins puede validar el
contrato con Spectral y Redocly, resolver referencias `$ref`, generar el
contrato consolidado y publicar el artefacto versionado. La
implementación y su ciclo de despliegue pueden gestionarse en
repositorios separados.

## 12. Decisiones pendientes antes de convertir el ejemplo en estándar

1.  Catálogo oficial de dominios y abreviaturas (`port`, `acct`, etc.).
2.  Estrategia de versionado: nombre lógico, URL, artefacto y
    compatibilidad.
3.  Modelo canónico aprobado para clientes, carteras, posiciones,
    cuentas y dinero.
4.  Definición financiera de valoración, coste base, rendimiento y
    conversión de divisas.
5.  Convenciones de paginación, filtros, fechas, errores y correlación.
6.  Autenticación, scopes, consentimiento y autorización a nivel de
    recurso.
7.  Límites de consumo, disponibilidad, timeouts y requisitos de
    observabilidad.
8.  Criterios de compatibilidad y proceso de aprobación de cambios
    contractuales.

------------------------------------------------------------------------

**Aviso:** este documento es un punto de partida didáctico. Los nombres,
endpoints, payloads, códigos de estado y modelos canónicos son
propuestas ilustrativas; no representan un estándar oficial ni un
contrato bancario real.
