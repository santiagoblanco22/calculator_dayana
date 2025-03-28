# Componentes UI del Calculador de Préstamos UVA

Este documento explica la estructura y organización de los componentes de interfaz de usuario utilizados en el Calculador de Préstamos UVA.

## Jerarquía de Componentes

```mermaid
graph TD
    LoanCalculator[LoanCalculator] --> Form[Formulario de Entrada]
    LoanCalculator --> Results[Visualización de Resultados]
    LoanCalculator --> Visualizations[Visualizaciones]

    Form --> InputControls[Controles de Entrada]
    Form --> ValidationMessages[Mensajes de Validación]
    Form --> SubmitButton[Botón de Cálculo]

    Results --> ResultSummary[Resumen de Resultados]
    Results --> ResultDetails[Detalles de Resultados]

    Visualizations --> TabsSystem[Sistema de Pestañas]
    TabsSystem --> Comparisons[Comparaciones]
    TabsSystem --> Evolution[Evolución de Capital]
    TabsSystem --> Composition[Composición de Pagos]
    TabsSystem --> Amortization[Tabla de Amortización]
```

## Controles de Entrada

Los controles de entrada permiten al usuario ingresar los datos necesarios para calcular el préstamo:

```mermaid
graph TD
    InputControls[Controles de Entrada] --> AgeInput[Control de Edad]
    InputControls --> IncomeInput[Control de Ingreso]
    InputControls --> HousingTypeRadio[Radio de Tipo de Vivienda]
    InputControls --> PropertyValueInput[Control de Valor de Propiedad]
    InputControls --> LoanTermInput[Control de Plazo]

    AgeInput --> AgeLabel[Label]
    AgeInput --> AgeInputField[Input Numérico]
    AgeInput --> AgeHelperText[Texto de Ayuda]

    IncomeInput --> IncomeLabel[Label]
    IncomeInput --> IncomeInputField[Input con Formato]
    IncomeInput --> IncomeHelperText[Texto de Ayuda]

    HousingTypeRadio --> HousingTypeLabel[Label]
    HousingTypeRadio --> PermanentOption[Opción Permanente]
    HousingTypeRadio --> NonPermanentOption[Opción No Permanente]

    PropertyValueInput --> PropertyValueLabel[Label]
    PropertyValueInput --> PropertyValueField[Input con Formato]
    PropertyValueInput --> PropertyValueHelperText[Texto de Ayuda]

    LoanTermInput --> LoanTermLabel[Label]
    LoanTermInput --> LoanTermField[Input Numérico]
    LoanTermInput --> LoanTermHelperText[Texto de Ayuda]
```

## Componentes de Validación

Los componentes de validación muestran mensajes de error y advertencia al usuario:

```mermaid
graph TD
    ValidationMessages[Mensajes de Validación] --> ErrorAlerts[Alertas de Error]
    ValidationMessages --> WarningAlerts[Alertas de Advertencia]

    ErrorAlerts --> ErrorIcon[Ícono de Error]
    ErrorAlerts --> ErrorList[Lista de Errores]

    WarningAlerts --> WarningIcon[Ícono de Advertencia]
    WarningAlerts --> WarningList[Lista de Advertencias]
```

## Visualización de Resultados

La estructura de componentes para mostrar los resultados del cálculo:

```mermaid
graph TD
    ResultDisplay[Visualización de Resultados] --> LoanAmountDisplay[Monto del Préstamo]
    ResultDisplay --> PaymentDisplay[Cuota Mensual]
    ResultDisplay --> TermDisplay[Plazo y Cuotas]
    ResultDisplay --> RatesDisplay[Tasas de Interés]
    ResultDisplay --> CFTEADisplay[CFTEA]
    ResultDisplay --> UVAValueDisplay[Valor UVA]

    LoanAmountDisplay --> LoanAmountValue[Valor en Pesos]
    LoanAmountDisplay --> LoanAmountUVA[Valor en UVAs]
    LoanAmountDisplay --> LoanLimitedAlert[Alerta de Límite]

    PaymentDisplay --> PaymentValue[Valor en Pesos]
    PaymentDisplay --> PaymentUVA[Valor en UVAs]

    TermDisplay --> TermYears[Años]
    TermDisplay --> TermPayments[Número de Cuotas]

    RatesDisplay --> TNAValue[Valor TNA]
    RatesDisplay --> TEAValue[Valor TEA]

    CFTEADisplay --> CFTEAValue[Valor CFTEA]

    UVAValueDisplay --> UVAValue[Valor UVA]
    UVAValueDisplay --> UVALastUpdate[Última Actualización]
```

## Sistema de Pestañas y Visualizaciones

Estructura de las pestañas y visualizaciones:

```mermaid
graph TD
    TabsSystem[Sistema de Pestañas] --> TabsList[Lista de Pestañas]
    TabsSystem --> TabsContent[Contenido de Pestañas]

    TabsList --> ComparisonTab[Pestaña Comparaciones]
    TabsList --> EvolutionTab[Pestaña Evolución]
    TabsList --> CompositionTab[Pestaña Composición]
    TabsList --> AmortizationTab[Pestaña Amortización]

    TabsContent --> ComparisonContent[Contenido Comparaciones]
    TabsContent --> EvolutionContent[Contenido Evolución]
    TabsContent --> CompositionContent[Contenido Composición]
    TabsContent --> AmortizationContent[Contenido Amortización]

    ComparisonContent --> IncomeDistributionChart[Gráfico Distribución Ingreso]
    ComparisonContent --> PropertyFinancingChart[Gráfico Financiación]
    ComparisonContent --> MaxLoanComparisonChart[Gráfico Montos Máximos]
    ComparisonContent --> TermImpactChart[Gráfico Impacto Plazo]

    EvolutionContent --> CapitalEvolutionChart[Gráfico Evolución Capital]

    CompositionContent --> PaymentCompositionChart[Gráfico Composición Pagos]

    AmortizationContent --> AmortizationTable[Tabla Amortización]
```

## Componentes de Gráficos

Detalle de los diferentes tipos de gráficos utilizados:

```mermaid
classDiagram
    class ReactApexChart {
        +options: Object
        +series: Array
        +type: string
        +height: number
        +render()
    }

    class PieChart {
        +labels: Array
        +colors: Array
        +legend: Object
        +dataLabels: Object
        +tooltip: Object
    }

    class BarChart {
        +xaxis: Object
        +yaxis: Object
        +plotOptions: Object
        +tooltip: Object
        +dataLabels: Object
    }

    class LineAreaChart {
        +stroke: Object
        +fill: Object
        +markers: Object
        +xaxis: Object
        +yaxis: Array
        +tooltip: Object
    }

    class StackedBarChart {
        +plotOptions: Object
        +dataLabels: Object
        +xaxis: Object
        +yaxis: Object
    }

    ReactApexChart <|-- PieChart
    ReactApexChart <|-- BarChart
    ReactApexChart <|-- LineAreaChart
    ReactApexChart <|-- StackedBarChart
```

## Estructura de Tabla de Amortización

```mermaid
graph TD
    AmortizationTable[Tabla de Amortización] --> TableHeader[Encabezado de Tabla]
    AmortizationTable --> TableBody[Cuerpo de Tabla]

    TableHeader --> HeaderCuota[Cabecera "Cuota"]
    TableHeader --> HeaderMensual[Cabecera "Cuota Mensual"]
    TableHeader --> HeaderInteres[Cabecera "Interés"]
    TableHeader --> HeaderCapital[Cabecera "Capital"]
    TableHeader --> HeaderSaldo[Cabecera "Saldo Restante"]

    TableBody --> TableRows[Filas de Tabla]
    TableRows --> Row1[Fila 1]
    TableRows --> Row2[Fila 2]
    TableRows --> RowN[Fila N...]

    Row1 --> Cell1_1[Número Cuota]
    Row1 --> Cell1_2[Valor Cuota]
    Row1 --> Cell1_3[Interés Pagado]
    Row1 --> Cell1_4[Capital Amortizado]
    Row1 --> Cell1_5[Saldo Pendiente]
```

## Mapa de Interacción del Usuario

Este diagrama muestra las posibles interacciones del usuario con la interfaz:

```mermaid
graph TD
    User([Usuario]) -->|Ingresa| Age[Ingresa Edad]
    User -->|Ingresa| Income[Ingresa Ingreso]
    User -->|Selecciona| HousingType[Selecciona Tipo Vivienda]
    User -->|Ingresa| PropertyValue[Ingresa Valor Propiedad]
    User -->|Ingresa| LoanTerm[Ingresa Plazo]

    Age --> Validate[Validación]
    Income --> Validate
    HousingType --> Validate
    PropertyValue --> Validate
    LoanTerm --> Validate

    Validate -->|Si hay errores| ShowErrors[Mostrar Errores]
    ShowErrors --> User

    Validate -->|Si hay advertencias| ShowWarnings[Mostrar Advertencias]
    ShowWarnings --> User

    User -->|Hace clic| Calculate[Botón Calcular]
    Calculate --> ProcessCalculation[Procesar Cálculo]

    ProcessCalculation --> DisplayResults[Mostrar Resultados]
    DisplayResults --> User

    User -->|Selecciona| ViewComparisons[Ver Comparaciones]
    User -->|Selecciona| ViewEvolution[Ver Evolución]
    User -->|Selecciona| ViewComposition[Ver Composición]
    User -->|Selecciona| ViewAmortization[Ver Amortización]

    ViewComparisons --> ShowComparisonsCharts[Mostrar Gráficos Comparativos]
    ViewEvolution --> ShowEvolutionChart[Mostrar Gráfico Evolución]
    ViewComposition --> ShowCompositionChart[Mostrar Gráfico Composición]
    ViewAmortization --> ShowAmortizationTable[Mostrar Tabla Amortización]

    ShowComparisonsCharts --> User
    ShowEvolutionChart --> User
    ShowCompositionChart --> User
    ShowAmortizationTable --> User
```

## Flujo de Renderizado de Componentes

```mermaid
sequenceDiagram
    participant ReactDOM
    participant LoanCalculator
    participant InputForm
    participant Results
    participant Tabs
    participant Charts

    ReactDOM->>LoanCalculator: Renderizar componente principal
    LoanCalculator->>InputForm: Renderizar formulario
    InputForm->>LoanCalculator: Montar componentes de entrada

    Note over LoanCalculator: Usuario ingresa datos y hace clic en calcular

    LoanCalculator->>LoanCalculator: Validar y calcular

    alt Cálculo exitoso
        LoanCalculator->>Results: Renderizar resultados
        LoanCalculator->>Tabs: Renderizar sistema de pestañas
        Tabs->>Charts: Renderizar gráficos
    else Errores de validación
        LoanCalculator->>InputForm: Mostrar errores/advertencias
    end

    Note over LoanCalculator: Usuario navega entre pestañas

    LoanCalculator->>Tabs: Actualizar pestaña activa
    Tabs->>Charts: Mostrar/ocultar gráficos según pestaña
```

## Estructura de CSS/Clases

```mermaid
graph TD
    CSSStructure[Estructura CSS] --> LayoutClasses[Clases de Layout]
    CSSStructure --> ComponentClasses[Clases de Componentes]
    CSSStructure --> UtilityClasses[Clases Utilitarias]

    LayoutClasses --> Container[Container]
    LayoutClasses --> FlexLayout[Flex Container]
    LayoutClasses --> Grid[Grid]

    ComponentClasses --> CardStyles[Card Styles]
    ComponentClasses --> InputStyles[Input Styles]
    ComponentClasses --> ButtonStyles[Button Styles]
    ComponentClasses --> AlertStyles[Alert Styles]
    ComponentClasses --> TabStyles[Tab Styles]
    ComponentClasses --> TableStyles[Table Styles]

    UtilityClasses --> Spacing[Espaciado]
    UtilityClasses --> Typography[Tipografía]
    UtilityClasses --> Colors[Colores]
    UtilityClasses --> Borders[Bordes]

    CardStyles --> CardHeader[Card Header]
    CardStyles --> CardContent[Card Content]
    CardStyles --> CardTitle[Card Title]

    InputStyles --> InputField[Input Field]
    InputStyles --> InputLabel[Input Label]
    InputStyles --> RadioGroup[Radio Group]

    AlertStyles --> AlertDestructive[Alert Destructive]
    AlertStyles --> AlertWarning[Alert Warning]
    AlertStyles --> AlertInfo[Alert Info]
```
