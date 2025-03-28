# Documentacion del Calculador de Prestamos UVA

Este repositorio contiene la documentacion detallada del Calculador de Prestamos UVA, una aplicacion React que permite a los usuarios simular prestamos hipotecarios en Unidades de Valor Adquisitivo (UVA) en Argentina.

## 📑 Contenido

La documentacion esta organizada en las siguientes secciones:

1. [Arquitectura del Sistema](./diagrams/architecture.md) - Diagramas que muestran la estructura general del sistema.
2. [Formulas Matematicas](./formulas/README.md) - Explicacion detallada de todas las formulas utilizadas en el calculo de prestamos.
3. [Flujo de Datos](./diagrams/data-flow.md) - Diagramas que ilustran como fluyen los datos en la aplicacion.
4. [Componentes UI](./diagrams/components.md) - Estructura de los componentes de la interfaz de usuario.
5. [Funciones Principales](./diagrams/functions.md) - Diagramas explicativos de las funciones clave del sistema.
6. [Validaciones](./diagrams/validations.md) - Logica de validacion de datos de entrada.

## 🔍 Vista Previa del Sistema

El Calculador de Prestamos UVA es una aplicacion web que permite a los usuarios:

- Simular prestamos hipotecarios con distintos plazos y montos
- Verificar la cuota mensual estimada
- Visualizar la evolucion del capital e intereses
- Comparar diferentes escenarios de prestamos
- Analizar la amortizacion del prestamo

## 🧮 Principales Formulas Utilizadas

Las formulas matematicas clave utilizadas en este sistema incluyen:

```mermaid
graph LR
    A["Formula Cuota Mensual"] --> B["PMT = $P \\times r \\times \\frac{(1 + r)^n}{(1 + r)^n - 1}$"]
    C["Formula Monto Maximo"] --> D["P = PMT \\times \\frac{1 - (1 + r)^{-n}}{r}"]
    E["Formula TEA"] --> F["TEA = (1 + \\frac{TNA}{12})^{12} - 1"]
```

Para una explicacion completa de todas las formulas, consulte la [documentacion de formulas](./formulas/README.md).

## 🔄 Flujo Principal del Sistema

```mermaid
flowchart TD
    A[Entrada de Datos] --> B{Validacion}
    B -->|Valido| C[Calculo de Prestamo]
    B -->|Invalido| D[Mostrar Errores]
    C --> E[Mostrar Resultados]
    E --> F[Visualizaciones]
    F --> G1[Comparaciones]
    F --> G2[Evolucion del Capital]
    F --> G3[Composicion de Cuotas]
    F --> G4[Tabla de Amortizacion]
```

## 🛠️ Tecnologias Utilizadas

- React
- Next.js
- TypeScript
- shadcn/ui
- ApexCharts

## 📊 Visualizaciones Disponibles

El sistema ofrece varias visualizaciones para ayudar al usuario a entender mejor su prestamo:

1. Distribucion del ingreso mensual
2. Financiacion de la propiedad
3. Comparacion de montos maximos
4. Impacto del plazo en la cuota
5. Evolucion del saldo de capital
6. Composicion de las cuotas a lo largo del tiempo
7. Tabla de amortizacion
