# Ojo Digital (FraudGuard AI)

Sistema inteligente para la detección y clasificación de transacciones fraudulentas en compras en línea.

## Descripción

Ojo Digital es un proyecto orientado al desarrollo de un sistema inteligente capaz de analizar transacciones de compras en línea y clasificarlas según su nivel de riesgo de fraude (**bajo, medio o alto**).

El sistema utilizará técnicas de inteligencia artificial y análisis de datos para identificar patrones asociados con operaciones fraudulentas y apoyar la detección temprana de transacciones sospechosas.

## Problema

Las compras en línea son una actividad cotidiana, pero también concentran riesgos de fraude. Identificar operaciones sospechosas es complicado cuando hay que analizar un gran volumen de transacciones en poco tiempo.

Se requiere una herramienta que analice las características de cada transacción, detecte patrones de comportamiento fraudulento y entregue una clasificación de riesgo que facilite su revisión.

## Objetivos

### Objetivo general

Desarrollar un sistema inteligente capaz de analizar transacciones de compras en línea y clasificarlas según su nivel de riesgo de fraude mediante técnicas de inteligencia artificial y análisis de datos.

### Objetivos particulares

- Diseñar una base de datos para almacenar y organizar información de usuarios, transacciones y características relevantes para el análisis de fraude.
- Implementar un módulo de análisis que procese las características de las transacciones e identifique patrones asociados con posible fraude.
- Desarrollar un mecanismo de clasificación que determine el nivel de riesgo (bajo, medio o alto).
- Desarrollar una interfaz web para consultar las transacciones analizadas y visualizar su nivel de riesgo junto con los principales factores del resultado.

## Alcance

El proyecto contempla una plataforma web para el análisis de transacciones de compras en línea que permitirá:

- Registrar y consultar información de transacciones.
- Almacenar información de usuarios y compras.
- Analizar las características de las transacciones.
- Usar un modelo de IA para apoyar la detección de posibles fraudes.
- Clasificar transacciones en riesgo bajo, medio o alto.
- Mostrar los principales factores asociados al resultado.
- Almacenar los resultados de los análisis.
- Consultar el historial de transacciones analizadas.

**Fuera de alcance (versión inicial):** bloqueo automático de pagos, integración con pasarelas de pago reales, análisis en tiempo real a gran escala y cumplimiento certificado PCI-DSS.

## MVP - Producto Mínimo Viable

Plataforma web que recibe los datos principales de una transacción, los procesa con un modelo de IA y muestra su nivel de riesgo (**Bajo / Medio / Alto**), los principales factores considerados y almacena el resultado para consulta posterior.

### Ejemplo de funcionamiento

Entrada de una transacción:

| Campo | Ejemplo |
|---|---|
| Monto de la compra | $8,500 MXN |
| Hora de la transacción | 03:12 a. m. |
| Ubicación | Ciudad distinta a la habitual |
| Dispositivo utilizado | Dispositivo nuevo |
| Compras recientes (24 h) | 6 |
| País habitual de compra | México (transacción desde otro país) |

Salida esperada:

```json
{
  "transaccion_id": 1024,
  "probabilidad_fraude": 0.87,
  "nivel_riesgo": "Alto",
  "factores_principales": [
    "Monto muy superior al promedio del usuario",
    "Compra en horario atípico",
    "País distinto al habitual",
    "Dispositivo no registrado previamente"
  ]
}
```

## Datos

Al ser un proyecto teórico, se propone entrenar y validar el modelo con conjuntos de datos públicos y reales (anonimizados):

| Dataset | Fuente | Descripción |
|---|---|---|
| Credit Card Fraud Detection | Kaggle (ULB) | ~284,807 transacciones de tarjeta de crédito en Europa, con ~492 fraudes (0.17 %). Variables anonimizadas por PCA, más `Time` y `Amount`. |
| IEEE-CIS Fraud Detection | Kaggle | Transacciones de e-commerce con datos de dispositivo, correo, tipo de tarjeta y producto. Más cercano al caso de compras en línea. |
| PaySim | Kaggle | Simulación de transacciones financieras basada en registros reales, útil para pruebas complementarias. |

**Consideraciones sobre los datos**

- Fuerte **desbalance de clases** (el fraude es una fracción mínima): se abordará con técnicas como SMOTE, submuestreo o ponderación de clases.
- Los datasets públicos no incluyen datos personales reales; no se almacenará información sensible (números de tarjeta completos, CVV).
- Para la plataforma web se generará además un conjunto de **datos sintéticos** (usuarios y compras) con los campos del ejemplo de funcionamiento, para poblar la base de datos y las demostraciones.

## Arquitectura propuesta

```
┌──────────────┐     HTTP/JSON     ┌──────────────────┐     ┌─────────────┐
│  Frontend    │ ────────────────▶ │  Backend (API)   │ ──▶ │ Base de     │
│  (Web)       │ ◀──────────────── │  + Módulo de IA  │ ◀── │ datos       │
└──────────────┘                   └──────────────────┘     └─────────────┘
                                          │
                                   ┌──────▼───────┐
                                   │ Modelo ML    │
                                   │ (serializado)│
                                   └──────────────┘
```

1. El usuario captura o consulta una transacción desde la interfaz web.
2. El backend valida los datos y calcula las variables derivadas (ej. promedio histórico del usuario, compras en las últimas 24 h).
3. El modelo devuelve una probabilidad de fraude, que se convierte en nivel de riesgo.
4. El resultado y los factores principales se guardan y se muestran en la interfaz.

**Umbrales de clasificación (propuestos, ajustables tras la validación):**

| Probabilidad de fraude | Nivel de riesgo |
|---|---|
| < 0.30 | Bajo |
| 0.30 – 0.70 | Medio |
| > 0.70 | Alto |

## Herramientas y tecnologías

| Capa | Herramienta | Uso |
|---|---|---|
| Lenguaje | Python 3.11+ | Backend y modelo de IA |
| Análisis de datos | pandas, NumPy | Limpieza y manipulación |
| Machine Learning | scikit-learn, XGBoost, imbalanced-learn | Entrenamiento, evaluación y manejo de desbalance |
| Explicabilidad | SHAP | Factores principales de cada predicción |
| Backend / API | FastAPI | Endpoints REST |
| Base de datos | PostgreSQL (SQLite en desarrollo) + SQLAlchemy | Persistencia |
| Frontend | HTML/CSS/JavaScript (o React) | Interfaz web |
| Exploración | Jupyter Notebook | EDA y experimentos |
| Pruebas | pytest | Pruebas unitarias |
| Control de versiones | Git y GitHub | Ramas, commits, Pull Requests |
| Documentación | Markdown | README y documentos en `/docs` |

## Modelo de base de datos

Entidades principales:

- **usuarios** (`id`, `nombre`, `correo`, `pais_habitual`, `fecha_registro`)
- **transacciones** (`id`, `usuario_id`, `monto`, `fecha_hora`, `ubicacion`, `pais`, `dispositivo`, `compras_recientes`)
- **analisis** (`id`, `transaccion_id`, `probabilidad_fraude`, `nivel_riesgo`, `fecha_analisis`, `version_modelo`)
- **factores_analisis** (`id`, `analisis_id`, `factor`, `peso`)

Relaciones: un usuario tiene muchas transacciones; cada transacción tiene uno o más análisis; cada análisis tiene varios factores asociados.

## Metodología del modelo de IA

1. **Exploración (EDA):** distribución de variables, valores faltantes y análisis del desbalance.
2. **Preprocesamiento:** limpieza, escalado y codificación de variables categóricas.
3. **Ingeniería de características:** hora del día, desviación del monto respecto al promedio del usuario, frecuencia de compras, cambio de país o dispositivo.
4. **Entrenamiento:** Regresión Logística (línea base), Random Forest y XGBoost.
5. **Evaluación:** dado el desbalance, se priorizan *precision*, *recall*, *F1* y *AUC-PR* sobre la exactitud (*accuracy*). Se usará validación cruzada estratificada.
6. **Explicabilidad:** SHAP para identificar los factores que más influyen en cada resultado.
7. **Selección y serialización** del mejor modelo para integrarlo al backend.

## Estructura del repositorio

```
Ojo-Digital/
├── README.md
├── docs/                 # Documentación adicional (diseño, diagramas, decisiones)
├── data/
│   ├── raw/              # Datasets originales (no se versionan)
│   └── processed/        # Datos limpios y sintéticos
├── notebooks/            # Exploración y experimentos
├── src/
│   ├── api/              # Endpoints del backend
│   ├── models/           # Entrenamiento y predicción
│   ├── db/               # Modelos y conexión a base de datos
│   └── utils/            # Funciones auxiliares
├── frontend/             # Interfaz web
├── tests/                # Pruebas
├── requirements.txt
└── .gitignore
```

## Flujo de desarrollo (Git)

- Rama principal: `main` (estable).
- Cada tarea se desarrolla en una rama propia, por ejemplo `docs/actualizar-readme`.
- Mensajes de commit descriptivos, por ejemplo: `docs: documenta estructura, herramientas y datos del proyecto`.
- Los cambios se integran mediante **Pull Request**; al referenciar el issue en el PR (`Closes #N`), este se cierra automáticamente al hacer merge.

```bash
git checkout -b docs/actualizar-readme
git add README.md
git commit -m "docs: documenta estructura, herramientas y datos del proyecto"
git push -u origin docs/actualizar-readme
# Abrir Pull Request en GitHub con "Closes #N" en la descripción
```

## Plan de desarrollo

| Fase | Actividad | Resultado |
|---|---|---|
| 1 | Documentación y estructura inicial | README y repositorio organizado |
| 2 | Diseño de la base de datos | Esquema y datos sintéticos |
| 3 | Exploración y preparación de datos | Dataset limpio y variables derivadas |
| 4 | Entrenamiento y evaluación del modelo | Modelo seleccionado con métricas |
| 5 | Backend y API | Endpoints de análisis y consulta |
| 6 | Interfaz web | Captura, resultados e historial |
| 7 | Pruebas e integración | MVP funcional |

## Riesgos y limitaciones

- **Desbalance de clases:** puede sesgar el modelo hacia la clase "no fraude".
- **Falsos positivos/negativos:** un modelo no es infalible; el resultado es un apoyo a la decisión, no una sentencia.
- **Representatividad de los datos:** los datasets públicos pueden no reflejar el comportamiento de compras en México.
- **Privacidad:** se evitará almacenar datos sensibles y se trabajará con información anonimizada.
- **Deriva del modelo:** los patrones de fraude cambian con el tiempo y requerirían reentrenamiento periódico.
