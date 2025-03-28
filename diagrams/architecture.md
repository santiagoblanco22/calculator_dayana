# Arquitectura del Calculador de Préstamos UVA

Este documento explica la arquitectura general del Calculador de Préstamos UVA, detallando sus componentes, interacciones y patrones de diseño.

## Arquitectura General

```mermaid
graph TD
    Client[Cliente/Navegador] --> NextJSApp[Aplicación Next.js]
    NextJSApp --> Components[Componentes React]
    NextJSApp --> Rendering[Client-Side Rendering]

    Components --> UI[Componentes UI]
    Components --> CoreLogic[Lógica de Negocio]
    Components --> Charts[Componentes de Gráficos]

    UI --> InputControls[Controles de Entrada]
    UI --> ResultDisplay[Visualización de Resultados]

    subgraph "Framework y Librerías"
        direction LR
        TypeScript --> React
        React --> NextJS
        ShadcnUI --> UI
        ApexCharts --> Charts
    end
```

## Estructura de Componentes

El sistema sigue un patrón de diseño basado en componentes, con una clara separación entre la interfaz de usuario y la lógica del negocio:

```mermaid
classDiagram
    class LoanCalculator {
        +useState() estados
        +useEffect() efectos
        +formatCurrency() formateo
        +calculateMaxLoanTerm() cálculo plazo
        +validateInputs() validación
        +generateAmortizationSchedule() amortización
        +generateCapitalEvolutionData() evolución capital
        +generatePaymentCompositionData() composición pagos
        +generateTermImpactData() impacto plazos
        +calculateLoan() cálculo principal
        +formatNumberInput() formateo
        +render() UI
    }

    class UI_Components {
        +Input
        +Button
        +Label
        +RadioGroup
        +Card
        +Alert
        +Tabs
        +Table
    }

    class ChartComponents {
        +ReactApexChart
        +ApexOptions
    }

    LoanCalculator --> UI_Components : usa
    LoanCalculator --> ChartComponents : usa
```

## Arquitectura de Datos

La estructura de datos del sistema muestra cómo se organizan y relacionan los diferentes objetos de datos:

```mermaid
erDiagram
    USER-INPUT ||--o{ LOAN-CALCULATION : provides-data-for
    LOAN-CALCULATION ||--|{ VISUALIZATION-DATA : generates

    USER-INPUT {
        number age
        number monthlyIncome
        string housingType
        number propertyValue
        number loanTerm
    }

    LOAN-CALCULATION {
        number loanAmount
        number loanAmountUVA
        number monthlyPayment
        number monthlyPaymentUVA
        number totalPayments
        number totalToPay
        number totalToPayUVA
        number tna
        number tea
        number cftea
        boolean limitedByIncome
    }

    VISUALIZATION-DATA {
        array amortizationSchedule
        array capitalEvolutionData
        array paymentCompositionData
        array termImpactData
        array incomeDistributionData
        array propertyFinancingData
        array maxLoanComparisonData
    }
```

## Ciclo de Vida del Componente

Este diagrama ilustra el ciclo de vida del componente principal LoanCalculator:

```mermaid
sequenceDiagram
    participant User
    participant React
    participant LoanCalculator
    participant ChildComponents

    React->>LoanCalculator: Inicialización
    LoanCalculator->>LoanCalculator: Definir estados (useState)
    LoanCalculator->>LoanCalculator: Definir constantes
    LoanCalculator->>LoanCalculator: Definir funciones helper

    LoanCalculator->>React: Renderizar UI inicial
    React->>ChildComponents: Renderizar componentes hijos
    ChildComponents->>User: Mostrar interfaz

    User->>LoanCalculator: Interactuar (cambiar inputs)
    LoanCalculator->>LoanCalculator: Actualizar estados

    User->>LoanCalculator: Hacer clic en "Calcular"
    LoanCalculator->>LoanCalculator: validateInputs()

    alt Validación exitosa
        LoanCalculator->>LoanCalculator: calculateLoan()
        LoanCalculator->>LoanCalculator: Generar datos de visualización
        LoanCalculator->>LoanCalculator: Actualizar estado results
        LoanCalculator->>React: Re-renderizar con resultados
    else Validación fallida
        LoanCalculator->>LoanCalculator: Actualizar errores/advertencias
        LoanCalculator->>React: Re-renderizar con mensajes
    end

    React->>ChildComponents: Actualizar componentes hijos
    ChildComponents->>User: Mostrar resultados/errores

    User->>LoanCalculator: Cambiar pestañas de visualización
    LoanCalculator->>React: Re-renderizar con visualización seleccionada
    React->>ChildComponents: Actualizar componentes de visualización
    ChildComponents->>User: Mostrar visualización
```

## Arquitectura de Flujos de Cálculo

El siguiente diagrama muestra cómo se organizan los diferentes cálculos en el sistema:

```mermaid
graph TD
    subgraph "Entrada y Validación"
        Input[Datos de Entrada] --> Validation[Validación]
        Validation --> CalculationProcess[Proceso de Cálculo]
    end

    subgraph "Cálculos Principales"
        CalculationProcess --> MaxLoanCalc[Cálculo Monto Máximo]
        CalculationProcess --> RatesCalc[Cálculo de Tasas]
        MaxLoanCalc --> PaymentCalc[Cálculo de Cuota]
        RatesCalc --> CFTEACalc[Cálculo de CFTEA]
    end

    subgraph "Generación de Datos para Visualizaciones"
        PaymentCalc --> AmortizationGen[Generación Amortización]
        PaymentCalc --> EvolutionGen[Generación Evolución]
        PaymentCalc --> CompositionGen[Generación Composición]
        MaxLoanCalc --> TermImpactGen[Generación Impacto Plazo]
        MaxLoanCalc --> DistributionGen[Generación Distribución]
        MaxLoanCalc --> FinancingGen[Generación Financiación]
        MaxLoanCalc --> ComparisonGen[Generación Comparación]
    end

    subgraph "Resultados"
        AmortizationGen --> Results[Resultados]
        EvolutionGen --> Results
        CompositionGen --> Results
        TermImpactGen --> Results
        DistributionGen --> Results
        FinancingGen --> Results
        ComparisonGen --> Results
        PaymentCalc --> Results
        CFTEACalc --> Results
    end
```

## Arquitectura de Visualizaciones

Este diagrama muestra cómo se organizan las diferentes visualizaciones:

```mermaid
graph TB
    Results[Resultados de Cálculo] --> Tabs[Sistema de Pestañas]

    Tabs --> ComparisonsTab[Pestaña Comparaciones]
    Tabs --> EvolutionTab[Pestaña Evolución]
    Tabs --> CompositionTab[Pestaña Composición]
    Tabs --> AmortizationTab[Pestaña Amortización]

    subgraph "Pestaña Comparaciones"
        ComparisonsTab --> IncomeDistChart[Gráfico Distribución Ingreso]
        ComparisonsTab --> FinancingChart[Gráfico Financiación]
        ComparisonsTab --> MaxLoanChart[Gráfico Montos Máximos]
        ComparisonsTab --> TermImpactChart[Gráfico Impacto Plazo]
    end

    subgraph "Pestaña Evolución"
        EvolutionTab --> CapitalEvolutionChart[Gráfico Evolución Capital]
    end

    subgraph "Pestaña Composición"
        CompositionTab --> PaymentCompositionChart[Gráfico Composición Pagos]
    end

    subgraph "Pestaña Amortización"
        AmortizationTab --> AmortizationTable[Tabla Amortización]
    end

    IncomeDistChart --> PieChart1[Gráfico Circular]
    FinancingChart --> PieChart2[Gráfico Circular]
    MaxLoanChart --> BarChart1[Gráfico de Barras]
    TermImpactChart --> BarChart2[Gráfico de Barras]
    CapitalEvolutionChart --> ComboChart[Gráfico Combo Área/Línea]
    PaymentCompositionChart --> StackedBarChart[Gráfico de Barras Apiladas]
    AmortizationTable --> Table[Tabla Paginada]
```

## Arquitectura de UI

Este diagrama ilustra la estructura jerárquica de los componentes de UI:

```mermaid
graph TD
    Root[div root] --> MainContainer[div container]

    MainContainer --> FormResultContainer[div flex container]
    FormResultContainer --> InputForm[Formulario de entrada]
    FormResultContainer --> ResultCard[Card de resultados]

    MainContainer --> VisualizationContainer[div visualizaciones]
    VisualizationContainer --> TabsComponent[Componente Tabs]

    InputForm --> AgeInput[Control Edad]
    InputForm --> IncomeInput[Control Ingreso]
    InputForm --> HousingTypeRadio[Radio Tipo Vivienda]
    InputForm --> PropertyValueInput[Control Valor Propiedad]
    InputForm --> LoanTermInput[Control Plazo]
    InputForm --> CalculateButton[Botón Calcular]

    ResultCard --> ResultsDisplay[Resultados Principales]
    ResultCard --> ResultsDetails[Detalles Adicionales]

    TabsComponent --> TabsList[Lista de Tabs]
    TabsComponent --> TabsContent[Contenido de Tabs]

    TabsList --> Tab1[Tab Comparaciones]
    TabsList --> Tab2[Tab Evolución]
    TabsList --> Tab3[Tab Composición]
    TabsList --> Tab4[Tab Amortización]

    TabsContent --> Content1[Contenido Comparaciones]
    TabsContent --> Content2[Contenido Evolución]
    TabsContent --> Content3[Contenido Composición]
    TabsContent --> Content4[Contenido Amortización]
```

## Comunicación entre Componentes

```mermaid
sequenceDiagram
    participant Parent as LoanCalculator
    participant InputControls as Controles de Entrada
    participant Results as Visualización Resultados
    participant Charts as Gráficos y Visualizaciones

    Parent->>InputControls: Props (valores, handlers)
    InputControls->>Parent: Eventos (onChange, onClick)

    Parent->>Parent: Procesar datos (calculateLoan)

    Parent->>Results: Props (resultados calculados)
    Parent->>Charts: Props (datos para gráficos)

    Charts->>Charts: Renderizar visualizaciones

    Parent->>Parent: Actualizar estado (useState)
    Parent->>InputControls: Re-renderizar con nuevos props
    Parent->>Results: Re-renderizar con nuevos props
    Parent->>Charts: Re-renderizar con nuevos props
```
