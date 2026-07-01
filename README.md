# Construir un agente con RAG en Oracle AI Database Private Agent Factory

Este laboratorio guía la instalación de **Oracle AI Database Private Agent Factory (DPAF)** y la creación de agentes capaces de consultar datos en una base Oracle usando lenguaje natural. El flujo completo parte de una Autonomous Database, despliega DPAF desde Marketplace, configura modelos de OCI Generative AI y construye agentes de análisis y flujos con Agent Builder.

## Documentos del laboratorio

- [1. Instalación.md](<1. Instalación.md>): despliegue y configuración inicial de DPAF.
- [2. Creación de un agente para análisis de datos.md](<2. Creación de un agente para análisis de datos.md>): carga de CSV y creación de un agente de análisis.
- [3. Creación de un flujo con Agent Builder.md](<3. Creación de un flujo con Agent Builder.md>): construcción de flujos visuales con componentes de entrada, prompt, agente, SQL y archivos.
- [PREMIER-LEAGUE-Creación de un agente para análisis de datos-.md](<PREMIER-LEAGUE-Creación de un agente para análisis de datos-.md>): caso completo con datos de Premier League y agente sobre métricas de jugadores.

## Prerrequisitos

Antes de iniciar, asegúrate de contar con:

- Políticas IAM para que los recursos puedan consumir servicios de IA.
- Un compartment para agrupar los recursos del laboratorio.
- Una Autonomous Database 23ai o 26ai, de tipo Transaction Processing o Data Warehouse.
- Credenciales de OCI configuradas con API Key.
- Una VCN con subred pública y reglas de ingreso para DPAF.
- Wallet de la base de datos.
- Llave pública SSH para el despliegue de la VM de DPAF.

## Paso 1. Descargar la wallet de la base de datos

1. Entra a la consola de OCI y navega a **Autonomous AI Database**.
2. Selecciona el compartment del laboratorio.
3. Abre la Autonomous Database que usarás.
4. En la página de la base de datos, entra a **Database connection**.
5. Descarga la wallet en formato `.zip`.
6. Define una contraseña para la wallet cuando la consola la solicite.

La wallet se usará más adelante para conectar DPAF con la base de datos.

## Paso 2. Crear usuarios y permisos en la base de datos

1. Abre **Database Actions** desde la página de la Autonomous Database.
2. Entra a la consola SQL.
3. Ejecuta el script de creación de usuarios del laboratorio.

El laboratorio crea principalmente:

- `app_factory`: usuario con permisos para instalar y operar DPAF.
- `AAI_RO_APP_FACTORY`: usuario de solo lectura para operaciones de la aplicación.

La contraseña por defecto usada en el documento de instalación es:

```text
ApPF4c7057YWorjksop
```

Puedes cambiarla si mantienes la misma contraseña en los pasos posteriores donde se configure la conexión.

## Paso 3. Desplegar DPAF desde Marketplace

1. En la consola de OCI, navega a **Marketplace > All Applications**.
2. Busca **Oracle AI Database Private Agent Factory**.
3. Selecciona la aplicación y crea el stack.
4. En **Stack information**, deja el nombre y descripción por defecto o ajusta valores descriptivos.
5. En **Configure variables**, completa la configuración general:

```text
Region: us-chicago-1
VM compartment: compartment del laboratorio
VCN compartment: compartment donde está la VCN
Subnet compartment: compartment donde está la subred
```

6. En **Network Configuration**, selecciona:

```text
VCN: VCN creada para el laboratorio
Existing subnet: subred pública
Public or Private subnet: public
```

7. En **Agent Factory VM**, configura:

```text
Availability Domain: cualquiera disponible
Agent Factory server display name: AgentFactoryVM
Agent Factory server shape: VM.Standard.E5.Flex
```

8. Adjunta la llave pública SSH.
9. Revisa la configuración y haz clic en **Create**.
10. Espera a que el job del stack finalice. Al terminar, los logs muestran la URL de DPAF.

## Paso 4. Registrar DPAF y conectar la base de datos

1. Abre la URL generada por el stack.
2. Completa el registro inicial de DPAF.
3. Configura la conexión a la base usando la wallet descargada.
4. Usa esta configuración:

```text
Air-gapped environment: No
Does the database server use a wallet?: Yes
Are the OCI certificates added to the wallet?: Yes
```

5. Prueba la conexión.
6. Continúa cuando aparezca el mensaje de conexión exitosa.

## Paso 5. Configurar los modelos de IA

Durante la instalación, DPAF solicita configurar un modelo de lenguaje y un modelo de embeddings.

Para el modelo de lenguaje:

```yaml
Model id: meta.llama-4-maverick-17b-128e-instruct-fp8
Endpoint: https://inference.generativeai.us-chicago-1.oci.oraclecloud.com
Compartment ID: ocid1.compartment...
User ID: ocid1.user.oc1...
```

Para embeddings:

```yaml
Model id: cohere.embed-multilingual-image-v3.0
Endpoint: https://inference.generativeai.us-chicago-1.oci.oraclecloud.com
Compartment ID: ocid1.compartment...
User ID: ocid1.user.oc1...
```

Si trabajas en otra región, cambia el endpoint y usa modelos disponibles en esa región.

## Paso 6. Validar la plataforma

Cuando la instalación termine, entra al home de DPAF.

![Pantalla principal de DPAF](<AI Private Agent Factory/dpaf home.png>)

Desde esta pantalla puedes crear fuentes de datos, agentes de análisis y flujos con Agent Builder.

## Paso 7. Cargar datos para el agente

El laboratorio incluye dos caminos:

- Cargar archivos CSV desde **Database Actions > Data Load**.
- Ejecutar el script SQL del caso Premier League para crear y llenar tablas automáticamente.

Para carga manual con CSV:

1. Entra a **Database Actions**.
2. Abre **Data Load**.
3. Haz clic en **Load Data**.
4. Arrastra los CSV o usa **Select Files**.
5. Revisa **Review Settings** para validar separador, tipos de datos y nombre de tabla.
6. Haz clic en **Start**.
7. Verifica que se carguen las filas y columnas esperadas.
8. Cierra el panel con **Close**.

Archivos incluidos para práctica:

- [clientes.csv](<datos/cobranza/clientes.csv>)
- [historial_contactos.csv](<datos/cobranza/historial_contactos.csv>)
- [historial_negociaciones.csv](<datos/cobranza/historial_negociaciones.csv>)
- [Manual_Politicas_Cobranza_RAG.pdf](<datos/cobranza/Manual_Politicas_Cobranza_RAG.pdf>)

Para el caso Premier League, ejecuta el script del documento específico. Ese script crea tablas como `EVENT`, `MATCH`, `PLAYER_STATS`, `PREDICT_BY_ANGLE` y `XG_MATRIX`.

## Paso 8. Crear un Data Source en DPAF

1. En DPAF, abre **Data Source** desde el menú izquierdo.
2. Crea un nuevo Data Source de tipo **Database**.
3. Completa:

```text
Nombre: nombre descriptivo
Descripción: propósito de la conexión
Tipo de conexión: Wallet
Database alias: alias terminado en _medium
Usuario: ADMIN
Contraseña: contraseña de la Autonomous Database
```

4. Haz clic en **Test Connection**.
5. Si la prueba es exitosa, guarda con **Add Database Source**.

## Paso 9. Crear el agente de análisis de datos

1. En el menú izquierdo, entra a **Data Analysis Agents**.
2. Haz clic en **Create Agent**.
3. Selecciona el Data Source creado.
4. Busca y selecciona las tablas que usará el agente.
5. Para Premier League, busca `player` y selecciona `ADMIN.PLAYER_STATS`.
6. Configura el agente con un nombre, descripción e instrucciones.

Configuración recomendada para Premier League:

```text
Agent name: Agente de análisis de rendimiento ofensivo de jugadores (xG)
```

```text
Description: Este agente analiza la tabla player_stats para responder consultas de rendimiento ofensivo por jugador y equipo, usando xG, goles, cantidad de tiros y eficiencia por disparo para comparar, rankear y detectar sobre/infra rendimiento.
```

Incluye en la ayuda del agente las columnas principales:

- `PLAYER_ID`: identificador del jugador.
- `PLAYER_NAME`: nombre del jugador.
- `TEAM_NAME`: equipo.
- `TOTAL_XG`: goles esperados acumulados.
- `GOALS`: goles anotados.
- `SHOT_COUNT`: total de remates.
- `XG_PER_SHOT`: calidad promedio del remate.

7. Revisa la configuración.
8. Publica el agente con **Publish Agent**.
9. Abre el agente con **Open Agent**.

## Paso 10. Probar el agente publicado

1. Entra al agente publicado.
2. Ejecuta **Execute Exploration** para generar una exploración automática de los datos.
3. Haz preguntas en lenguaje natural.
4. Revisa la respuesta, las tablas y visualizaciones generadas.
5. Usa el botón **SQL** para inspeccionar la consulta creada por el agente.

Ejemplos de preguntas:

- Top 10 jugadores por goles.
- Quiénes sobre-rinden más respecto a su xG.
- Compara dos jugadores por xG, goles y eficiencia.
- Top por xG por disparo con mínimo 20 tiros.
- Promedio de xG por equipo.

## Paso 11. Crear un flujo simple en Agent Builder

Este flujo sirve para validar que Agent Builder funciona de extremo a extremo.

1. En el menú izquierdo, abre **Agent Builder**.
2. Haz clic en **New Flow**.
3. Arrastra un bloque **Chat input** al lienzo.
4. Arrastra un bloque **Prompt** y configura el template.
5. Arrastra un bloque **Agent**.
6. Configura el agente:

```text
Select LLM to use: openai.gpt-oss-120b (oci)
Temperature: 0.01
Agent description: Agent
```

7. Conecta **Prompt message** del bloque `Prompt` a **Custom instructions** del bloque `Agent`.
8. Conecta **Message** del bloque `Chat input` a **Prompt** del bloque `Agent`.
9. Arrastra un bloque **Chat output**.
10. Conecta **Message** del bloque `Agent` a **Message** del bloque `Chat output`.
11. Guarda con **Save**.
12. Prueba el flujo en **Playground**.

## Paso 12. Crear un flujo completo con SQL y documentos

El flujo completo combina datos estructurados de la base con contenido documental.

1. En **Agent Builder**, crea o edita un flujo.
2. Agrega un componente **SQL query**.
3. Configura:

```sql
SELECT * FROM TU_TABLA
```

4. Activa **Include columns**.
5. Agrega un componente **File upload** y carga un PDF de ejemplo.
6. Agrega un componente **Prompt** con este template:

```text
Responde a la pregunta indicada usando la siguiente información

{{datos}}
{{documentos}}
```

7. Guarda el prompt para habilitar los conectores `datos` y `documentos`.
8. Conecta **Message** del bloque `SQL query` a `datos`.
9. Conecta **Content** del bloque `File upload` a `documentos`.
10. Conecta **Prompt message** del bloque `Prompt` a **Custom instructions** del bloque `Agent`.
11. Conecta **Chat input** al campo **Prompt** del bloque `Agent`.
12. Conecta la salida del `Agent` al **Chat output**.
13. Guarda y publica el flujo.
14. Prueba en **Playground** con preguntas sobre los datos y el documento.

## Resultado esperado

Al completar el laboratorio tendrás:

- DPAF desplegado en OCI.
- Una Autonomous Database conectada mediante wallet.
- Modelos de lenguaje y embeddings configurados.
- Datos cargados en la base.
- Un agente de análisis de datos publicado.
- Un flujo en Agent Builder que combina preguntas, SQL y documentos para responder en lenguaje natural.
