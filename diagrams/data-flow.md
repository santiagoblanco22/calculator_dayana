# Flujo de Datos del Calculador de Préstamos UVA

Este documento explica cómo fluyen los datos a través del sistema, desde la entrada del usuario hasta las visualizaciones finales.

## Diagrama General de Flujo de Datos

```mermaid
flowchart TD
    User([Usuario]) -->|Ingresa Datos| Inputs[Formulario de Entrada]
    Inputs -->|Valida| Validator[Validador de Datos]
    Validator -->|Si es válido| Calculator[Calculadora de Préstamo]
    Validator -->|Si no es válido| Errors[Mensajes de Error]
    Errors -->|Muestra| User

    Calculator -->|Genera| Results[Resultados del Préstamo]
    Results -->|Muestra| User
    Results -->|Alimenta| Visualizations[Visualizaciones]
    Visualizations -->|Muestra| User
```

## Flujo de Estados

El componente `LoanCalculator` gestiona varios estados que evolucionan durante la interacción del usuario:

```mermaid
stateDiagram-v2
    [*] --> FormularioInicial
    FormularioInicial --> Validando: Usuario hace clic en "Calcular préstamo"
    Validando --> MostrandoErrores: Validación fallida
    MostrandoErrores --> Validando: Usuario corrige datos
    Validando --> Calculando: Validación exitosa
    Calculando --> MostrandoResultados: Cálculos completos
    MostrandoResultados --> ViendoComparaciones: Usuario selecciona pestaña
    MostrandoResultados --> ViendoEvolucion: Usuario selecciona pestaña
    MostrandoResultados --> ViendoComposicion: Usuario selecciona pestaña
    MostrandoResultados --> ViendoAmortizacion: Usuario selecciona pestaña
    ViendoComparaciones --> MostrandoResultados: Usuario vuelve a resultados
    ViendoEvolucion --> MostrandoResultados: Usuario vuelve a resultados
    ViendoComposicion --> MostrandoResultados: Usuario vuelve a resultados
    ViendoAmortizacion --> MostrandoResultados: Usuario vuelve a resultados
    MostrandoResultados --> Validando: Usuario modifica datos
```

## Diagrama de Variables de Estado

El componente mantiene múltiples variables de estado que afectan los cálculos:

```mermaid
classDiagram
    class Estados {
        uvaValue: number
        monthlyIncome: number
        propertyValue: number
        loanTerm: number
        housingType: string
        age: number
        results: object
        lastUpdate: string
        errors: string[]
        warnings: string[]
        showAmortizationTable: boolean
    }

    class Constantes {
        MIN_AGE: number
        MAX_AGE_AT_COMPLETION: number
        MIN_INCOME: number
        MAX_LOAN_TERM: number
        INCOME_RATIO: number
        INTEREST_RATE: number
    }

    class Resultados {
        loanAmount: number
        loanAmountUVA: number
        totalPayments: number
        monthlyPayment: number
        monthlyPaymentUVA: number
        totalToPay: number
        totalToPayUVA: number
        tna: number
        tea: number
        cftea: number
        maxLoanByProperty: number
        maxLoanByIncome: number
        limitedByIncome: boolean
        maxMonthlyPayment: number
        amortizationSchedule: array
        capitalEvolutionData: array
        paymentCompositionData: array
        termImpactData: array
        incomeDistributionData: array
        propertyFinancingData: array
        maxLoanComparisonData: array
    }

    Estados ..> Resultados : almacena en
    Constantes --> Estados : afecta
```

## Flujo de Validación de Datos

```mermaid
flowchart TD
    Start([Inicio]) --> ValidateAge{Edad >= 18?}
    ValidateAge -->|No| AgeError[Error: Edad mínima 18 años]
    ValidateAge -->|Sí| ValidateIncome{Ingreso >= 850000?}
    ValidateIncome -->|No| IncomeError[Error: Ingreso mínimo requerido]
    ValidateIncome -->|Sí| ValidateTerm{Plazo <= Máximo?}
    ValidateTerm -->|No| TermWarning[Advertencia: Plazo ajustado]
    ValidateTerm -->|Sí| Valid[Datos válidos]
    TermWarning --> AdjustTerm[Ajustar plazo]
    AdjustTerm --> Valid

    Valid --> Calculate[Calcular préstamo]
    AgeError --> ShowErrors[Mostrar errores]
    IncomeError --> ShowErrors
```

## Flujo Detallado de Cálculo

```mermaid
flowchart TD
    Start([Iniciar Cálculo]) --> FinancingPercent[Determinar % financiación]
    FinancingPercent --> MaxLoanProperty[Calcular máximo por propiedad]
    Start --> MaxPayment[Calcular pago máximo por ingreso]
    MaxPayment --> MaxLoanIncome[Calcular máximo por ingreso]
    MaxLoanProperty --> CompareLimits{Comparar límites}
    MaxLoanIncome --> CompareLimits
    CompareLimits --> FinalLoanAmount[Determinar monto final]
    FinalLoanAmount --> CalculateUVA[Convertir a UVAs]
    FinalLoanAmount --> CalculateRates[Calcular tasas]
    FinalLoanAmount --> CalculatePayment[Calcular cuota mensual]
    CalculatePayment --> PaymentUVA[Convertir cuota a UVAs]
    CalculatePayment --> TotalToPay[Calcular total a pagar]

    CalculatePayment --> GenerateAmortization[Generar amortización]
    CalculatePayment --> GenerateEvolution[Generar evolución capital]
    CalculatePayment --> GenerateComposition[Generar composición pagos]
    FinalLoanAmount --> GenerateTermImpact[Generar impacto plazos]

    GenerateDistribution[Generar distribución ingresos]
    GenerateFinancing[Generar financiación propiedad]
    GenerateComparison[Generar comparación montos]

    SetResults[Establecer resultados]
```

## Flujo de Generación de Gráficos

```mermaid
flowchart LR
    Results[Resultados del cálculo] --> IncomeChart[Gráfico Distribución Ingresos]
    Results --> FinancingChart[Gráfico Financiación Propiedad]
    Results --> ComparisonChart[Gráfico Comparación Montos]
    Results --> TermImpactChart[Gráfico Impacto Plazo]
    Results --> EvolutionChart[Gráfico Evolución Capital]
    Results --> CompositionChart[Gráfico Composición Pagos]
    Results --> AmortizationTable[Tabla Amortización]

    subgraph Charts [Visualizaciones]
        IncomeChart
        FinancingChart
        ComparisonChart
        TermImpactChart
        EvolutionChart
        CompositionChart
        AmortizationTable
    end

    Charts --> Tabs[Organización en Pestañas]
```

## Transformación Detallada de Datos

Este diagrama muestra cómo se transforman los datos desde la entrada hasta cada visualización:

```mermaid
graph TD
    subgraph Entrada
        Age[Edad]
        Income[Ingreso Mensual]
        HousingType[Tipo de Vivienda]
        PropertyValue[Valor Propiedad]
        LoanTerm[Plazo Préstamo]
    end

    subgraph Cálculos
        MaxTerm[Plazo Máximo]
        MaxLoanProperty[Máximo por Propiedad]
        MaxLoanIncome[Máximo por Ingreso]
        FinalLoan[Monto Final Préstamo]
        MonthlyPayment[Cuota Mensual]
        InterestRates[Tasas de Interés]
    end

    subgraph Datos_Visualización
        AmortizationData[Datos Amortización]
        EvolutionData[Datos Evolución]
        CompositionData[Datos Composición]
        TermImpactData[Datos Impacto Plazo]
        DistributionData[Datos Distribución Ingreso]
        FinancingData[Datos Financiación]
        ComparisonData[Datos Comparación]
    end

    Age --> MaxTerm
    Income --> MaxLoanIncome
    HousingType --> MaxLoanProperty
    PropertyValue --> MaxLoanProperty
    LoanTerm --> FinalLoan
    MaxTerm --> FinalLoan

    MaxLoanProperty --> FinalLoan
    MaxLoanIncome --> FinalLoan
    FinalLoan --> MonthlyPayment
    FinalLoan --> InterestRates

    MonthlyPayment --> AmortizationData
    FinalLoan --> AmortizationData

    MonthlyPayment --> EvolutionData
    FinalLoan --> EvolutionData
    LoanTerm --> EvolutionData

    MonthlyPayment --> CompositionData
    FinalLoan --> CompositionData
    LoanTerm --> CompositionData

    FinalLoan --> TermImpactData

    Income --> DistributionData
    MonthlyPayment --> DistributionData

    PropertyValue --> FinancingData
    FinalLoan --> FinancingData

    MaxLoanProperty --> ComparisonData
    MaxLoanIncome --> ComparisonData
```

## Flujo de Actualización de UI

```mermaid
sequenceDiagram
    actor Usuario
    participant Form as Formulario
    participant Calculator as Calculadora
    participant Visualizations as Visualizaciones

    Usuario->>Form: Modifica valores de entrada
    Form->>Form: Actualiza estados locales
    Form->>Calculator: Clic en "Calcular préstamo"
    Calculator->>Calculator: Validar entradas

    alt Datos válidos
        Calculator->>Calculator: Ejecutar cálculos
        Calculator->>Visualizations: Actualizar datos
        Visualizations->>Usuario: Mostrar resultados y gráficos
    else Datos inválidos
        Calculator->>Form: Mostrar errores/advertencias
        Form->>Usuario: Visualizar mensajes
    end

    Usuario->>Visualizations: Navegar entre pestañas
    Visualizations->>Usuario: Mostrar diferentes visualizaciones
```
