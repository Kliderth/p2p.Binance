# p2p.Binance
>No existe api oficial de precios binance  p2p.
>la comunidad a logrado determinar cual es la api interna para entrar a la web p2p pero no es fácil entender su funcionamiento.
>
>Quiero plantearlo de forma sencilla para que cualquiera lo pueda comprender
# Binance P2P — Documentación del Endpoint de Búsqueda de Anuncios

>**Consideraciones:** El endpoint funciona con el encabezado que envia la información de navegador
>y con un archivo .JSON que debe incluir la información que se busca (token, moneda, información de anuncio, entre otras)
>dado que la web esta en constante actualización este endpoint puede cambiar en cualquier momento.
>El uso de esta información es responsabilidad exclusiva del usuario final.

---

> **Advertencia:** Este es un endpoint **interno no oficial** de Binance.
> Fue descubierto inspeccionando el tráfico de red del navegador (DevTools → Network)
> en [p2p.binance.com](https://p2p.binance.com) mientras se usa la plataforma normalmente.
> Binance **no publica documentación oficial** para esta API. Puede cambiar o
> desaparecer sin previo aviso.

---

## ¿Cómo se descubrió?

1. Abrir [https://p2p.binance.com](https://p2p.binance.com) en Chrome/Firefox.
2. Abrir **DevTools** → pestaña **Network** → filtrar por `XHR` o `Fetch`.
3. Aplicar cualquier filtro en la interfaz (moneda, tipo de operación, etc.).
4. La solicitud `adv/search` aparece en la lista de peticiones con su payload y respuesta completos.

---

## Endpoint

```
POST https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search
```

### Cabeceras requeridas

| Cabecera         | Valor                              |
|------------------|------------------------------------|
| `Content-Type`   | `application/json`                 |
| `User-Agent`     | Cualquier User-Agent de navegador  |

> No requiere autenticación ni API Key para consultas públicas de anuncios.

---

## Cuerpo de la solicitud (Request Body)

```json
{
  "asset":          "USDT",
  "countries":      [],
  "fiat":           "VES",
  "page":           1,
  "payTypes":       [],
  "publisherType":  null,
  "rows":           10,
  "tradeType":      "SELL"
}
```

### Parámetros

| Campo           | Tipo            | Obligatorio | Descripción |
|-----------------|-----------------|-------------|-------------|
| `asset`         | `string`        | ✅ Sí       | Criptomoneda a operar. Valores comunes: `"USDT"`, `"BTC"`, `"ETH"`, `"BNB"`, `"BUSD"`. |
| `fiat`          | `string`        | ✅ Sí       | Moneda fiat. Ejemplos: `"VES"` (bolívar venezolano), `"COP"` (peso colombiano), `"USD"`, `"EUR"`, `"ARS"`, `"PEN"`, `"MXN"`. |
| `tradeType`     | `string`        | ✅ Sí       | Tipo de operación desde la perspectiva del **comerciante**:<br>`"SELL"` → el comerciante **vende** cripto (el usuario la compra).<br>`"BUY"` → el comerciante **compra** cripto (el usuario la vende). |
| `page`          | `integer`       | ✅ Sí       | Número de página (paginación). Comienza en `1`. |
| `rows`          | `integer`       | ✅ Sí       | Cantidad de anuncios por página. Máximo recomendado: `20`. |
| `payTypes`      | `array<string>` | No          | Filtrar por métodos de pago. Array vacío `[]` = todos. Valores posibles: `"Banesco"`, `"Bancamiga"`, `"Nequi"`, `"Bancolombia"`, `"Daviplata"`, `"Zelle"`, `"PayPal"`, etc. |
| `countries`     | `array<string>` | No          | Filtrar por país del comerciante. Array vacío `[]` = todos. Código ISO-3166 Alpha-2 en mayúsculas, ej. `["VE"]`, `["CO"]`. |
| `publisherType` | `string\|null`  | No          | Tipo de publicador. `null` = todos. `"merchant"` = solo comerciantes certificados. |

---

## Respuesta (Response)

```json
{
  "code": "000000",
  "message": null,
  "messageDetail": null,
  "data": [ /* array de anuncios */ ],
  "total": 1483,
  "success": true
}
```

### Campos de la respuesta raíz

| Campo           | Tipo              | Descripción |
|-----------------|-------------------|-------------|
| `code`          | `string`          | `"000000"` = éxito. Cualquier otro valor es error. |
| `data`          | `array<Anuncio>`  | Lista de anuncios que coinciden con el filtro. |
| `total`         | `integer`         | Número total de anuncios disponibles (para paginación). |
| `success`       | `boolean`         | `true` si la solicitud fue exitosa. |

---

### Objeto `Anuncio`

Cada elemento de `data` tiene la siguiente estructura:

```json
{
  "adv": { ... },
  "advertiser": { ... }
}
```

#### Sub-objeto `adv` (el anuncio)

| Campo                    | Tipo     | Descripción |
|--------------------------|----------|-------------|
| `advNo`                  | `string` | ID único del anuncio. |
| `tradeType`              | `string` | `"SELL"` o `"BUY"`. |
| `asset`                  | `string` | Criptomoneda (`"USDT"`, etc.). |
| `fiatUnit`               | `string` | Moneda fiat (`"VES"`, `"COP"`, etc.). |
| `price`                  | `string` | Precio por unidad de cripto en fiat. Llega como string, convertir a float. |
| `surplusAmount`          | `string` | USDT disponibles para operar en este anuncio. |
| `tradableQuantity`       | `string` | Cantidad real negociable ahora mismo. |
| `minSingleTransAmount`   | `string` | Monto mínimo de la transacción en fiat. |
| `maxSingleTransAmount`   | `string` | Monto máximo de la transacción en fiat. |
| `tradeMethods`           | `array`  | Métodos de pago aceptados (ver abajo). |
| `isOnline`               | `integer`| `1` = comerciante en línea, `0` = fuera de línea. |

#### Sub-objeto `tradeMethods[n]`

| Campo        | Tipo     | Descripción |
|--------------|----------|-------------|
| `identifier` | `string` | Nombre del método de pago (`"Banesco"`, `"Nequi"`, `"Bancolombia"`, etc.). |
| `tradeMethodName` | `string` | Nombre legible del método. |

#### Sub-objeto `advertiser` (el comerciante)

| Campo               | Tipo     | Descripción |
|---------------------|----------|-------------|
| `userNo`            | `string` | ID único del comerciante. |
| `nickName`          | `string` | Nombre visible del comerciante. |
| `monthFinishRate`   | `number` | Tasa de completado en los últimos 30 días (0.0 a 1.0). Multiplicar × 100 para obtener %. |
| `monthOrderCount`   | `integer`| Órdenes en los últimos 30 días. |
| `tradeCount`        | `integer`| Total de operaciones históricas. Este campo llega como string en algunas respuestas. |
| `positiveRate`      | `number` | Porcentaje de valoraciones positivas (0.0 a 1.0). |
| `userType`          | `string` | `"user"` = usuario regular, `"merchant"` = comerciante certificado. |
| `isOnline`          | `boolean`| Si el comerciante está conectado. |
| `lastOnlineTime`    | `integer`| Timestamp Unix (ms) de la última conexión. |

---

## Ejemplo completo

### Request

```bash
curl -X POST "https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search" \
  -H "Content-Type: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -d '{
    "asset": "USDT",
    "countries": [],
    "fiat": "VES",
    "page": 1,
    "payTypes": [],
    "publisherType": null,
    "rows": 5,
    "tradeType": "SELL"
  }'
```

### Response (resumida)

```json
{
  "code": "000000",
  "data": [
    {
      "adv": {
        "advNo": "11451849236953948160",
        "tradeType": "SELL",
        "asset": "USDT",
        "fiatUnit": "VES",
        "price": "732.00",
        "tradableQuantity": "2600.00",
        "minSingleTransAmount": "1500000.00",
        "maxSingleTransAmount": "1871000.00",
        "tradeMethods": [
          { "identifier": "Banesco", "tradeMethodName": "Banesco" }
        ]
      },
      "advertiser": {
        "nickName": "PlimardoPAYMENTS",
        "monthFinishRate": 0.91,
        "monthOrderCount": 120,
        "tradeCount": "1580",
        "userType": "user",
        "isOnline": true
      }
    }
  ],
  "total": 1483,
  "success": true
}
```

---

## Casos de uso comunes

| Objetivo | `tradeType` | Interpretación |
|----------|-------------|----------------|
| Ver a qué precio puedo **comprar** USDT con Bs | `"SELL"` | El comerciante vende USDT, tú pagas Bs |
| Ver a qué precio puedo **vender** mis USDT por Bs | `"BUY"` | El comerciante compra USDT, tú recibes Bs |
| Precio de mercado de referencia | Promedio de los top 10 de ambos tipos | Spread entre compra y venta |

---

## Limitaciones conocidas

| Limitación | Detalle |
|------------|---------|
| **No oficial** | Binance puede cambiar la ruta, campos o agregar autenticación sin aviso. |
| **Rate limit** | No está documentado. Esperar ≥ 1 segundo entre solicitudes es recomendable. |
| **Sin WebSocket** | No hay suscripción en tiempo real; hay que hacer polling. |
| **Paginación máxima** | `rows` máximo práctico: 20. Valores mayores pueden devolver error o ser ignorados. |
| **Datos string** | Campos numéricos como `price` y `tradeCount` llegan como `string`, no `number`. Convertir con `float()` / `int()`. |

---

## Otros endpoints P2P descubiertos por la comunidad

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/bapi/c2c/v2/friendly/c2c/adv/search` | POST | **Este endpoint** — buscar anuncios. |
| `/bapi/c2c/v2/friendly/c2c/order/list` | POST | Historial de órdenes (requiere auth). |
| `/bapi/c2c/v1/private/c2c/order/detail` | GET | Detalle de una orden (requiere auth). |

> Los endpoints marcados como "requiere auth" necesitan cookies de sesión de Binance y no son accesibles sin autenticación.
