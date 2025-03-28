# Funciones Principales del Calculador de Préstamos UVA

Este documento explica en detalle las funciones principales utilizadas en el Calculador de Préstamos UVA, mostrando sus entradas, salidas y lógica interna.

## Mapa General de Funciones

```mermaid
graph TD
    LoanCalculator[Componente LoanCalculator] --> FormatFunctions[Funciones de Formateo]
    LoanCalculator --> ValidationFunctions[Funciones de Validación]
    LoanCalculator --> CalculationFunctions[Funciones de Cálculo]
    LoanCalculator --> DataGenerationFunctions[Funciones de Generación de Datos]

    FormatFunctions --> formatCurrency[formatCurrency]
    FormatFunctions --> formatNumberInput[formatNumberInput]

    ValidationFunctions --> validateInputs[validateInputs]
    ValidationFunctions --> calculateMaxLoanTerm[calculateMaxLoanTerm]

    CalculationFunctions --> calculateLoan[calculateLoan]

    DataGenerationFunctions --> generateAmortizationSchedule[generateAmortizationSchedule]
    DataGenerationFunctions --> generateCapitalEvolutionData[generateCapitalEvolutionData]
    DataGenerationFunctions --> generatePaymentCompositionData[generatePaymentCompositionData]
    DataGenerationFunctions --> generateTermImpactData[generateTermImpactData]
```

## Función: formatCurrency

Esta función convierte un número en una cadena formateada como moneda argentina.

```mermaid
graph TD
    Input[Entrada: value: number] --> FormatProcess[Proceso de formateo]
    FormatProcess --> Intl[Usar Intl.NumberFormat]
    Intl --> Options[Opciones: estilo moneda, ARS]
    Options --> Result[Resultado: Cadena formateada]
```

**Firma de la función:**
```javascript
const formatCurrency = (value: number) => {
  return new Intl.NumberFormat("es-AR", {
    style: "currency",
    currency: "ARS",
    maximumFractionDigits: 0,
  }).format(value)
}
```

## Función: calculateMaxLoanTerm

Calcula el plazo máximo de préstamo basado en la edad del solicitante.

```mermaid
flowchart TD
    A[Entrada: currentAge: number] --> B[maxYears = MAX_AGE_AT_COMPLETION - currentAge]
    B --> C{maxYears > MAX_LOAN_TERM?}
    C -->|Sí| D[Usar MAX_LOAN_TERM: 30]
    C -->|No| E[Usar maxYears]
    D --> F[Retornar plazo máximo]
    E --> F
```

**Firma de la función:**
```javascript
const calculateMaxLoanTerm = (currentAge: number) => {
  const maxYears = MAX_AGE_AT_COMPLETION - currentAge
  return Math.min(maxYears, MAX_LOAN_TERM)
}
```

## Función: validateInputs

Valida todas las entradas del usuario y establece errores o advertencias según corresponda.

```mermaid
flowchart TD
    A[Inicio] --> B[Inicializar arrays de errores y advertencias]
    B --> C{Edad < MIN_AGE?}
    C -->|Sí| D[Añadir error de edad]
    C -->|No| E{Ingreso < MIN_INCOME?}
    D --> E
    E -->|Sí| F[Añadir error de ingreso]
    E -->|No| G{Plazo > máximo para edad?}
    F --> G
    G -->|Sí| H[Añadir advertencia de plazo]
    G -->|No| I[No añadir advertencia]
    H --> J[Ajustar plazo al máximo]
    I --> K[Establecer errores y advertencias]
    J --> K
    K --> L{Hay errores?}
    L -->|Sí| M[Retornar falso]
    L -->|No| N[Retornar verdadero]
```

**Firma de la función:**
```javascript
const validateInputs = () => {
  const newErrors: string[] = []
  const newWarnings: string[] = []

  // Validate age
  if (age < MIN_AGE) {
    newErrors.push(`La edad mínima para solicitar un préstamo es de ${MIN_AGE} años.`)
  }

  // Validate income
  if (monthlyIncome < MIN_INCOME) {
    newErrors.push(`El ingreso mínimo requerido es de ${formatCurrency(MIN_INCOME)}.`)
  }

  // Validate loan term based on age
  const maxTerm = calculateMaxLoanTerm(age)
  if (loanTerm > maxTerm) {
    newWarnings.push(`Debido a tu edad (${age} años), el plazo máximo de préstamo es de ${maxTerm} años.`)
    setLoanTerm(maxTerm)
  }

  setErrors(newErrors)
  setWarnings(newWarnings)

  return newErrors.length === 0
}
```

## Función: calculateLoan

Función principal que realiza todos los cálculos del préstamo y genera todos los datos para visualizaciones.

```mermaid
flowchart TD
    A[Inicio] --> B{Validar Entradas}
    B -->|No válido| C[Establecer results a null]
    B -->|Válido| D[Determinar % de financiación]
    D --> E[Calcular monto máximo por propiedad]
    D --> F[Calcular monto máximo por ingreso]
    E --> G[Determinar monto final]
    F --> G
    G --> H[Convertir a UVAs]
    G --> I[Calcular tasas]
    G --> J[Calcular cuota mensual]
    J --> K[Convertir cuota a UVAs]
    J --> L[Calcular total a pagar]

    J --> M[Generar amortización]
    J --> N[Generar evolución capital]
    J --> O[Generar composición pagos]
    J --> P[Generar impacto plazo]

    G --> Q[Generar datos de distribución ingreso]
    G --> R[Generar datos de financiación]
    G --> S[Generar datos de comparación]

    M --> T[Establecer results]
    N --> T
    O --> T
    P --> T
    Q --> T
    R --> T
    S --> T
    H --> T
    I --> T
    J --> T
    K --> T
    L --> T
```

**Estructura simplificada de la función:**
```javascript
const calculateLoan = () => {
  if (!validateInputs()) {
    setResults(null)
    return
  }

  // Cálculos de monto máximo
  const financingPercentage = housingType === "permanent" ? 0.8 : 0.5
  const maxLoanByProperty = propertyValue * financingPercentage
  const maxMonthlyPayment = monthlyIncome * INCOME_RATIO
  const monthlyRate = INTEREST_RATE / 100 / 12
  const totalPayments = loanTerm * 12
  const maxLoanByIncome = maxMonthlyPayment * ((1 - Math.pow(1 + monthlyRate, -totalPayments)) / monthlyRate)
  const loanAmount = Math.min(maxLoanByProperty, maxLoanByIncome)

  // Conversiones y cálculos adicionales
  const loanAmountUVA = loanAmount / uvaValue
  const tna = INTEREST_RATE
  const tea = (Math.pow(1 + tna / 100 / 12, 12) - 1) * 100
  const cftea = tea + 0.19
  const monthlyPayment = (loanAmount * (monthlyRate * Math.pow(1 + monthlyRate, totalPayments))) /
    (Math.pow(1 + monthlyRate, totalPayments) - 1)
  const monthlyPaymentUVA = monthlyPayment / uvaValue
  const totalToPay = monthlyPayment * totalPayments
  const limitedByIncome = maxLoanByIncome < maxLoanByProperty

  // Generar datos para visualizaciones
  const amortizationSchedule = generateAmortizationSchedule(loanAmount, monthlyRate, totalPayments, monthlyPayment)
  const capitalEvolutionData = generateCapitalEvolutionData(loanAmount, monthlyRate, totalPayments, monthlyPayment)
  const paymentCompositionData = generatePaymentCompositionData(loanAmount, monthlyRate, totalPayments, monthlyPayment)
  const termImpactData = generateTermImpactData(loanAmount, monthlyRate)

  // Datos adicionales para gráficos
  const incomeDistributionData = [...]
  const propertyFinancingData = [...]
  const maxLoanComparisonData = [...]

  // Establecer resultados
  setResults({
    loanAmount,
    loanAmountUVA,
    totalPayments,
    monthlyPayment,
    // ...resto de propiedades
  })
}
```

## Función: generateAmortizationSchedule

Genera la tabla de amortización que muestra el desglose de cada cuota.

```mermaid
flowchart TD
    A[Entradas: monto, tasa, cuotas, pago] --> B[Inicializar array schedule]
    B --> C[Inicializar balance = loanAmount]
    C --> D{Para i de 1 a min(totalPayments, 12)}
    D --> E[Calcular interés: interest = balance * monthlyRate]
    E --> F[Calcular capital: principal = monthlyPayment - interest]
    F --> G[Actualizar balance: balance = balance - principal]
    G --> H[Agregar fila a schedule]
    H --> D
    D --> I[Retornar schedule]
```

**Firma de la función:**
```javascript
const generateAmortizationSchedule = (
  loanAmount: number,
  monthlyRate: number,
  totalPayments: number,
  monthlyPayment: number,
) => {
  const schedule = []
  let balance = loanAmount

  for (let i = 1; i <= Math.min(totalPayments, 12); i++) {
    const interest = balance * monthlyRate
    const principal = monthlyPayment - interest
    balance = balance - principal

    schedule.push({
      payment: i,
      monthlyPayment,
      interest,
      principal,
      balance: Math.max(0, balance),
    })
  }

  return schedule
}
```

## Función: generateCapitalEvolutionData

Genera datos para visualizar la evolución del saldo de capital a lo largo del préstamo.

```mermaid
flowchart TD
    A[Entradas: monto, tasa, cuotas, pago] --> B[Inicializar array data]
    B --> C[Inicializar balance = loanAmount]
    C --> D[Agregar punto para año 0 con 100%]
    D --> E{Para cada año de 1 a loanTerm}
    E --> F{Para cada mes de 1 a 12}
    F --> G[Calcular interest y principal]
    G --> H[Actualizar balance]
    H --> F
    F --> I[Calcular porcentaje restante]
    I --> J[Agregar punto a data]
    J --> E
    E --> K[Retornar data]
```

**Estructura simplificada de la función:**
```javascript
const generateCapitalEvolutionData = (
  loanAmount: number,
  monthlyRate: number,
  totalPayments: number,
  monthlyPayment: number,
) => {
  const data = []
  let balance = loanAmount

  // Para cada año
  for (let year = 0; year <= loanTerm; year++) {
    // Caso especial para año 0
    if (year === 0) {
      data.push({
        year,
        balance,
        percentage: 100,
      })
      continue
    }

    // Simular 12 meses
    for (let month = 1; month <= 12; month++) {
      const interest = balance * monthlyRate
      const principal = monthlyPayment - interest
      balance = Math.max(0, balance - principal)
    }

    // Agregar punto para el año actual
    data.push({
      year,
      balance,
      percentage: (balance / loanAmount) * 100,
    })
  }

  return data
}
```

## Función: generatePaymentCompositionData

Genera datos para visualizar cómo se distribuye cada pago entre interés y capital.

```mermaid
flowchart TD
    A[Entradas: monto, tasa, cuotas, pago] --> B[Inicializar array data]
    B --> C[Inicializar balance = loanAmount]
    C --> D{Para años seleccionados hasta loanTerm}
    D --> E[Inicializar yearInterest y yearPrincipal]
    E --> F{Para cada mes de 1 a 12}
    F --> G[Calcular interest y principal]
    G --> H[Acumular en yearInterest y yearPrincipal]
    H --> I[Actualizar balance]
    I --> F
    F --> J[Calcular porcentajes interest/principal]
    J --> K[Agregar datos del año a data]
    K --> D
    D --> L[Retornar data]
```

**Estructura simplificada de la función:**
```javascript
const generatePaymentCompositionData = (
  loanAmount: number,
  monthlyRate: number,
  totalPayments: number,
  monthlyPayment: number,
) => {
  const data = []
  let balance = loanAmount

  // Seleccionar años representativos
  for (let year = 1; year <= loanTerm; year += Math.max(1, Math.floor(loanTerm / 5))) {
    let yearInterest = 0
    let yearPrincipal = 0

    // Simular 12 meses
    for (let month = 1; month <= 12; month++) {
      const interest = balance * monthlyRate
      const principal = monthlyPayment - interest
      yearInterest += interest
      yearPrincipal += principal
      balance = Math.max(0, balance - principal)
    }

    // Calcular porcentajes y agregar datos
    data.push({
      year,
      interest: yearInterest,
      principal: yearPrincipal,
      interestPercentage: (yearInterest / (yearInterest + yearPrincipal)) * 100,
      principalPercentage: (yearPrincipal / (yearInterest + yearPrincipal)) * 100,
    })
  }

  return data
}
```

## Función: generateTermImpactData

Genera datos para visualizar cómo el plazo afecta a la cuota mensual y el costo total.

```mermaid
flowchart TD
    A[Entradas: monto, tasa] --> B[Inicializar array data]
    B --> C{Para plazos de 10 a 30 años cada 5}
    C --> D[Calcular número de pagos]
    D --> E[Calcular cuota mensual]
    E --> F[Calcular total a pagar]
    F --> G[Calcular total de intereses]
    G --> H[Agregar datos a data]
    H --> C
    C --> I[Retornar data]
```

**Firma de la función:**
```javascript
const generateTermImpactData = (loanAmount: number, monthlyRate: number) => {
  const data = []

  // Calcular para diferentes plazos
  for (let term = 10; term <= 30; term += 5) {
    const payments = term * 12
    const payment = (loanAmount * (monthlyRate * Math.pow(1 + monthlyRate, payments))) /
                    (Math.pow(1 + monthlyRate, payments) - 1)

    data.push({
      term,
      payment,
      totalPaid: payment * payments,
      totalInterest: payment * payments - loanAmount,
    })
  }

  return data
}
```

## Diagrama de Secuencia de Llamadas a Funciones

Este diagrama muestra la secuencia de llamadas a funciones cuando un usuario hace clic en "Calcular":

```mermaid
sequenceDiagram
    participant User as Usuario
    participant Calc as LoanCalculator
    participant Valid as validateInputs
    participant MaxTerm as calculateMaxLoanTerm
    participant Format as formatCurrency
    participant Main as calculateLoan
    participant Amort as generateAmortizationSchedule
    participant Evol as generateCapitalEvolutionData
    participant Comp as generatePaymentCompositionData
    participant Term as generateTermImpactData

    User->>Calc: Clic en "Calcular préstamo"
    Calc->>Main: calculateLoan()
    Main->>Valid: validateInputs()
    Valid->>MaxTerm: calculateMaxLoanTerm(age)
    MaxTerm-->>Valid: maxTerm
    Valid->>Format: formatCurrency(MIN_INCOME)
    Format-->>Valid: incomeFmt
    Valid-->>Main: isValid

    alt isValid == true
        Main->>Main: Calcular montos y tasas
        Main->>Amort: generateAmortizationSchedule(...)
        Amort-->>Main: amortizationSchedule
        Main->>Evol: generateCapitalEvolutionData(...)
        Evol-->>Main: capitalEvolutionData
        Main->>Comp: generatePaymentCompositionData(...)
        Comp-->>Main: paymentCompositionData
        Main->>Term: generateTermImpactData(...)
        Term-->>Main: termImpactData
        Main->>Main: Generar datos adicionales
        Main->>Calc: setResults(...)
    else isValid == false
        Main->>Calc: setResults(null)
    end

    Calc->>User: Mostrar resultados o errores
```

## Dependencias entre Funciones

```mermaid
graph TD
    calculateLoan --> validateInputs
    validateInputs --> calculateMaxLoanTerm
    validateInputs --> formatCurrency

    calculateLoan --> generateAmortizationSchedule
    calculateLoan --> generateCapitalEvolutionData
    calculateLoan --> generatePaymentCompositionData
    calculateLoan --> generateTermImpactData

    formatNumberInput -.-> calculateLoan
```

## Diagrama de Flujo de la Función formatNumberInput

```mermaid
flowchart TD
    A[Entrada: value: string] --> B[Aplicar regex para eliminar caracteres no numéricos]
    B --> C[Retornar cadena con solo dígitos]
```

**Firma de la función:**
```javascript
const formatNumberInput = (value: string) => {
  // Remove non-numeric characters
  return value.replace(/\D/g, "")
}
```
