# Skill: clima

Obtén el clima actual de cualquier ciudad usando el servicio gratuito `wttr.in` (sin API key).

## Cuándo usar esta skill

Úsala cuando el usuario pida información del clima, temperatura, pronóstico del tiempo, o condiciones meteorológicas de cualquier lugar.

Ejemplos de triggers:
- "¿qué clima hace en Quito?"
- "dime el pronóstico del tiempo"
- "clima local"
- "temperatura actual en X"
- "/clima Madrid"

## Instrucciones

1. **Detecta la ubicación** del mensaje del usuario:
   - Si el usuario menciona una ciudad explícita, úsala
   - Si dice "local", "aquí", "mi ciudad" o no especifica, usa la ciudad configurada por defecto: **Quito, Ecuador**

2. **Ejecuta el siguiente comando PowerShell** para obtener el clima (reemplaza `CIUDAD` por el nombre de la ciudad, usando `+` para espacios):

```powershell
# Formato resumido (una línea)
(Invoke-WebRequest -Uri "https://wttr.in/CIUDAD?format=3" -UseBasicParsing).Content

# Formato completo con pronóstico de 3 días
(Invoke-WebRequest -Uri "https://wttr.in/CIUDAD?format=j1" -UseBasicParsing).Content | ConvertFrom-Json
```

3. **Presenta la información** de forma clara al usuario:
   - Condición del tiempo (soleado, nublado, lluvia, etc.)
   - Temperatura actual en °C
   - Sensación térmica
   - Humedad
   - Pronóstico para hoy/mañana si el usuario lo pide

### Formato rápido (recomendado para consultas simples)

```powershell
(Invoke-WebRequest -Uri "https://wttr.in/CIUDAD?format=%l:+%c+%t+%h" -UseBasicParsing).Content
```

Parámetros de formato:
- `%l` = ubicación
- `%c` = condición (emoji)
- `%t` = temperatura
- `%h` = humedad
- `%w` = viento
- `%p` = precipitación
- `%m` = fase lunar

### Formato JSON completo

```powershell
$weather = (Invoke-WebRequest -Uri "https://wttr.in/CIUDAD?format=j1" -UseBasicParsing).Content | ConvertFrom-Json
$current = $weather.current_condition[0]

[PSCustomObject]@{
    Ubicacion    = $weather.nearest_area[0].areaName[0].value
    Condicion    = $current.weatherDesc[0].value
    Temperatura  = "$($current.temp_C)°C"
    Sensacion    = "$($current.FeelsLikeC)°C"
    Humedad      = "$($current.humidity)%"
    Viento       = "$($current.windspeedKmph) km/h"
    Visibilidad  = "$($current.visibility) km"
}
```

## Errores comunes

- Si la ciudad no se encuentra, `wttr.in` devolverá un error o datos de otra ciudad. Informa al usuario y pídele que sea más específico.
- Si no hay conexión a internet, el `Invoke-WebRequest` fallará. Comunica el error claramente.
- Ciudades con tildes o caracteres especiales: codifícalas en URL (ej. `Bogotá` → `Bogota`).

## Notas

- `wttr.in` es un servicio público gratuito, no requiere API key.
- Los datos se actualizan aproximadamente cada 30 minutos.
- Soporta nombres de ciudades en español, inglés y otros idiomas.
- También acepta coordenadas GPS: `wttr.in/-0.22,-78.51` para Quito.
