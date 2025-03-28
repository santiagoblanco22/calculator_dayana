# Validaciones del Calculador de Prestamos UVA

Este documento detalla todas las validaciones implementadas en el Calculador de Prestamos UVA, explicando su proposito, logica y el impacto que tienen en la experiencia del usuario.

## Diagrama General de Validaciones

```mermaid
flowchart TD
    Input[Datos de Entrada] --> ValidationProcess[Proceso de Validacion]
    ValidationProcess --> ValidEntry{Datos Validos?}
    ValidEntry -->|Si| CalculationProcess[Proceder con Calculos]
    ValidEntry -->|No| ErrorDisplay[Mostrar Errores]
    ValidationProcess --> WarningEntry{Hay Advertencias?}
    WarningEntry -->|Si| WarningDisplay[Mostrar Advertencias]
    WarningEntry -->|No| NoWarningDisplay[No Mostrar Advertencias]
```

## Constantes y Umbrales de Validacion

El sistema utiliza las siguientes constantes para las validaciones:

```mermaid
classDiagram
    class ValidationConstraints {
        +MIN_AGE: 18
        +MAX_AGE_AT_COMPLETION: 70
        +MIN_INCOME: 850000
        +MAX_LOAN_TERM: 30
        +INCOME_RATIO: 0.25
    }
```

## Tipos de Validaciones

```mermaid
graph TD
    Validations[Validaciones] --> ErrorValidations[Validaciones de Error]
    Validations --> WarningValidations[Validaciones de Advertencia]

    ErrorValidations --> AgeValidation[Validacion de Edad]
    ErrorValidations --> IncomeValidation[Validacion de Ingreso]

    WarningValidations --> TermValidation[Validacion de Plazo]
```

## Validacion de Edad

```mermaid
flowchart TD
    A[Entrada: Edad] --> B{Edad >= MIN_AGE?}
    B -->|Si| C[Validacion Exitosa]
    B -->|No| D[Error: Edad Minima]

    E[MIN_AGE = 18] --> B
```

**Implementacion:**
```javascript
// Validate age
if (age < MIN_AGE) {
  newErrors.push(`La edad minima para solicitar un prestamo es de ${MIN_AGE} años.`)
}
```

## Validacion de Ingreso

```mermaid
flowchart TD
    A[Entrada: Ingreso Mensual] --> B{Ingreso >= MIN_INCOME?}
    B -->|Si| C[Validacion Exitosa]
    B -->|No| D[Error: Ingreso Minimo]

    E[MIN_INCOME = 850,000] --> B
```

**Implementacion:**
```javascript
// Validate income
if (monthlyIncome < MIN_INCOME) {
  newErrors.push(`El ingreso minimo requerido es de ${formatCurrency(MIN_INCOME)}.`)
}
```

## Validacion de Plazo

```mermaid
flowchart TD
    A[Entradas: Edad, Plazo] --> B[Calcular Plazo Maximo]
    B --> C{Plazo <= Plazo Maximo?}
    C -->|Si| D[Validacion Exitosa]
    C -->|No| E[Advertencia: Ajustar Plazo]
    E --> F[Ajustar Plazo Automaticamente]

    G[MAX_AGE_AT_COMPLETION = 70] --> B
    H[MAX_LOAN_TERM = 30] --> B
```

**Implementacion:**
```javascript
// Validate loan term based on age
const maxTerm = calculateMaxLoanTerm(age)
if (loanTerm > maxTerm) {
  newWarnings.push(`Debido a tu edad (${age} años), el plazo maximo de prestamo es de ${maxTerm} años.`)
  setLoanTerm(maxTerm)
}
```

## Funcion calculateMaxLoanTerm

```mermaid
flowchart TD
    A[Entrada: currentAge] --> B[maxYears = MAX_AGE_AT_COMPLETION - currentAge]
    B --> C[Retornar min(maxYears, MAX_LOAN_TERM)]
```

**Implementacion:**
```javascript
const calculateMaxLoanTerm = (currentAge) => {
  const maxYears = MAX_AGE_AT_COMPLETION - currentAge;
  return Math.min(maxYears, MAX_LOAN_TERM);
};
```

## Flujo de Validacion Completo

```mermaid
sequenceDiagram
    participant U as Usuario
    participant I as Inputs
    participant V as Validador
    participant C as Calculadora

    U->>I: Ingresar datos
    I->>V: Validar datos

    V->>V: Validar edad
    V->>V: Validar ingreso
    V->>V: Validar plazo

    alt Hay errores
        V->>I: Mostrar errores
        I->>U: Ver mensajes de error
    else No hay errores pero hay advertencias
        V->>I: Mostrar advertencias
        V->>I: Ajustar valores (si es necesario)
        V->>C: Proceder con calculos
    else Todo valido
        V->>C: Proceder con calculos
    end
```

## Diagrama de Estados de Validacion

```mermaid
stateDiagram-v2
    [*] --> Inicial
    Inicial --> Validando: Usuario hace clic en "Calcular prestamo"

    Validando --> Error: Hay errores
    Validando --> AdvertenciaAjuste: Hay advertencias y ajustes
    Validando --> Valido: Todo valido

    Error --> Validando: Usuario corrige datos
    AdvertenciaAjuste --> Calculando: Ajustes automaticos aplicados
    Valido --> Calculando: Proceder con calculos

    Calculando --> MostrandoResultados: Calculos completos
    MostrandoResultados --> [*]
```

## Componentes UI de Validacion

Los mensajes de validacion se muestran a traves de componentes de alerta:

```mermaid
graph TD
    Validation[Validaciones] --> ErrorAlerts[Alertas de Error]
    Validation --> WarningAlerts[Alertas de Advertencia]

    ErrorAlerts --> DestructiveAlert[Alert variant="destructive"]
    DestructiveAlert --> ErrorIcon[AlertCircle Icon]
    DestructiveAlert --> ErrorMessages[Lista de Mensajes de Error]

    WarningAlerts --> YellowAlert[Alert className="border-yellow-500..."]
    YellowAlert --> InfoIcon[Info Icon]
    YellowAlert --> WarningMessages[Lista de Mensajes de Advertencia]
```

**Implementacion de UI de error:**
```jsx
{errors.length > 0 && (
  <Alert variant="destructive" className="mb-4">
    <AlertCircle className="h-4 w-4" />
    <AlertDescription>
      <ul className="list-disc pl-5 mt-2">
        {errors.map((error, index) => (
          <li key={index}>{error}</li>
        ))}
      </ul>
    </AlertDescription>
  </Alert>
)}
```

**Implementacion de UI de advertencia:**
```jsx
{warnings.length > 0 && (
  <Alert className="mb-4 border-yellow-500 text-yellow-800 bg-yellow-50">
    <Info className="h-4 w-4" />
    <AlertDescription>
      <ul className="list-disc pl-5 mt-2">
        {warnings.map((warning, index) => (
          <li key={index}>{warning}</li>
        ))}
      </ul>
    </AlertDescription>
  </Alert>
)}
```

## Restricciones en Inputs

Ademas de las validaciones explicitas, se aplican restricciones directamente en los controles de entrada:

```mermaid
graph TD
    InputRestrictions[Restricciones de Input] --> AgeInput[Input de Edad]
    InputRestrictions --> IncomeInput[Input de Ingreso]
    InputRestrictions --> LoanTermInput[Input de Plazo]

    AgeInput --> AgeMin[min={MIN_AGE}]
    AgeInput --> AgeMax[max={MAX_AGE_AT_COMPLETION}]

    IncomeInput --> IncomeFormat[Formateo a moneda]

    LoanTermInput --> LoanTermMin[min="1"]
    LoanTermInput --> LoanTermMax[max={calculateMaxLoanTerm(age)}]
```

**Implementacion de restricciones de edad:**
```jsx
<Input
  id="age"
  type="number"
  min={MIN_AGE}
  max={MAX_AGE_AT_COMPLETION}
  value={age}
  onChange={(e) => setAge(Number(e.target.value))}
  className="text-lg"
/>
```

**Implementacion de restricciones de plazo:**
```jsx
<Input
  id="loanTerm"
  type="number"
  min="1"
  max={calculateMaxLoanTerm(age)}
  value={loanTerm}
  onChange={(e) => setLoanTerm(Number(e.target.value))}
  className="text-lg"
/>
```

## Ajuste Automatico de Plazo

```mermaid
flowchart TD
    A[Edad del usuario cambia] --> B[useEffect para monitorear cambios]
    B --> C[Calcular plazo maximo]
    C --> D{Plazo actual > Plazo maximo?}
    D -->|Si| E[Ajustar plazo al maximo]
    D -->|No| F[No hacer cambios]

    G[El usuario intenta calcular] --> H[validateInputs]
    H --> I[Calcular plazo maximo]
    I --> J{Plazo actual > Plazo maximo?}
    J -->|Si| K[Añadir advertencia y ajustar plazo]
    J -->|No| L[No hacer cambios]
```

**Implementacion de ajuste automatico en useEffect:**
```javascript
// Update max loan term when age changes
useEffect(() => {
  const maxTerm = calculateMaxLoanTerm(age)
  if (loanTerm > maxTerm) {
    setLoanTerm(maxTerm)
  }
}, [age])
```

## Relacion entre Validaciones y Calculos

```mermaid
flowchart TD
    Start[Inicio] --> Validation[Validacion]
    Validation --> IsValid{Valido?}
    IsValid -->|No| ShowErrors[Mostrar Errores]
    IsValid -->|Si| CalcProcess[Proceso de Calculo]

    CalcProcess --> MaxByProperty[Calculo por Propiedad]
    CalcProcess --> MaxByIncome[Calculo por Ingreso]
    MaxByProperty --> LowerValue{Cual es menor?}
    MaxByIncome --> LowerValue
    LowerValue --> FinalAmount[Monto Final]
    FinalAmount --> Results[Resultados]

    LowerValue -->|MaxByIncome| LimitedByIncome[Limitado por Ingreso]
    LimitedByIncome --> ShowWarning[Mostrar Alerta Azul]
```

## Tooltip de Informacion Adicional

El sistema muestra informacion adicional para ayudar al usuario a entender las restricciones:

```mermaid
graph TD
    Tooltips[Textos Informativos] --> AgeTooltip[Texto Informativo de Edad]
    Tooltips --> IncomeTooltip[Texto Informativo de Ingreso]
    Tooltips --> LoanTermTooltip[Texto Informativo de Plazo]
    Tooltips --> HousingTypeTooltip[Texto Informativo de Tipo de Vivienda]

    AgeTooltip --> AgeText["La edad minima es 18 años. El prestamo debe finalizar antes de los 70 años."]
    IncomeTooltip --> IncomeText["Podes sumar tus ingresos y los de tu grupo familiar directo. Minimo: $850,000."]
    LoanTermTooltip --> LoanTermText["Podes pagar tu prestamo entre 1 y {maxTerm} años (segun tu edad)."]
```

## Mensaje de Prestamo Limitado por Ingreso

Cuando el prestamo esta limitado por el ingreso, se muestra un mensaje informativo:

```mermaid
graph TD
    LimitCheck[Verificar Limitacion] --> IsLimited{Limitado por Ingreso?}
    IsLimited -->|Si| ShowBlueAlert[Mostrar Alerta Azul]
    IsLimited -->|No| NoAlert[No Mostrar Alerta]

    ShowBlueAlert --> AlertMessage["El monto del prestamo esta limitado por tu ingreso. La cuota no puede superar el 25% de tus ingresos mensuales."]
```

**Implementacion:**
```jsx
{results.limitedByIncome && (
  <Alert className="mt-3 border-blue-500 text-blue-800 bg-blue-50">
    <Info className="h-4 w-4" />
    <AlertDescription>
      El monto del prestamo esta limitado por tu ingreso. La cuota no puede superar el 25% de tus
      ingresos mensuales ({formatCurrency(results.maxMonthlyPayment)}).
    </AlertDescription>
  </Alert>
)}
```
