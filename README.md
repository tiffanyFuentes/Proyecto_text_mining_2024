# Proyecto_text_mining_2024
Proyecto de la materia optativa Text Mining 2024 de FaMAF

# Instrucciones para Ejecutar el Script de Generación de Resúmenes

Este repositorio contiene un script que utiliza un modelo LLM para procesar documentos judiciales y generar un resumen estructurado dividido en **Datos**, **Síntesis**, y **Sumarios**. Sigue estas instrucciones para configurar y ejecutar el script.

## Requisitos Previos
1. **Python 3.9+**
2. Instalación de las siguientes bibliotecas:
   ```bash
   pip install transformers rouge-score
   ```
3. Descomprime los archivos:
   - `Analisis-20241201T194507Z-001.zip`: Contiene el archivo de entrenamiento y otros datos necesarios.
   - `BASE-20241201T193944Z-001.zip`: Contiene los documentos judiciales a analizar.

## Configuración Inicial
1. **Descomprime los archivos** en el directorio raíz del repositorio:
   ```bash
   unzip Analisis-20241201T194507Z-001.zip
   unzip BASE-20241201T193944Z-001.zip
   ```
   - Esto creará las carpetas `Analisis/` y `BASE/` en el directorio raíz.

## Opciones de Ejecución
El script puede ejecutarse de dos maneras:

### 1. Selección Automática de un Archivo
Si no especificas un archivo de entrada, el script seleccionará aleatoriamente uno de los documentos listados en el archivo `Entrenamiento.txt` dentro de la carpeta `Analisis/`.

Comando:
```bash
python script.py
```

### 2. Especificar un Archivo de Entrada
Puedes pasar un archivo específico como argumento al ejecutar el script. Asegúrate de que el archivo exista dentro de la carpeta `BASE/`.

Comando:
```bash
python script.py ruta/al/archivo.txt
```

## Salida del Script
El script generará un resumen estructurado con las siguientes secciones:
- **Datos**: Extrae información clave como fecha, sede, jueces, etc.
- **Síntesis**: Resume los puntos procesales clave.
- **Sumarios**: Genera un sumario jurisprudencial abstracto.

La salida será un archivo en la misma ubicación del documento procesado, con el nombre:
```
nombredelarchivo_resumen_llama.txt
```
Por ejemplo, si procesas el archivo `documento1.txt`, la salida será:
```
BASE/documento1_resumen_llama.txt
```

## Consideraciones Importantes
- Durante la generación de los resúmenes, **el script ejecuta tres prompts distintos por cada sección** (Datos, Síntesis y Sumarios). El usuario debe seleccionar manualmente cuál de los tres resultados considera mejor para incluir en el resumen final.
- Aunque el proceso está bastante automatizado, el output generado por el modelo **puede incluir repeticiones o una estructura imperfecta**. Es necesario revisar y ajustar manualmente el texto final para asegurar su calidad.
- Si tienes problemas con la selección automática, verifica que el archivo `Entrenamiento.txt` contenga las rutas correctas de los documentos judiciales dentro de la carpeta `BASE/`.

## Ejemplo de Ejecución
### Sin archivo de entrada (selección automática):
```bash
python proyecto.py
```
Salida esperada:
```
Generando resultados para las tres secciones del resumen...
DATOS:
...
SÍNTESIS:
...
SUMARIOS:
...
Resumen final guardado en: BASE/documento_seleccionado_resumen_llama.txt
```

### Con archivo específico:
```bash
python proyecto.py BASE/documento1.txt
```
Salida esperada:
```
Generando resultados para las tres secciones del resumen...
DATOS:
...
SÍNTESIS:
...
SUMARIOS:
...
Resumen final guardado en: BASE/documento1_resumen_llama.txt
```

