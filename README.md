# Clasificación de sentimientos en español — FastAPI + Docker


Este notebook contiene un proyecto aplicado en el que selecciona un dataset público, ya sea de clasificación o regresión, realiza su preparación y versionamiento, entrena un modelo de aprendizaje automático, ajusta hiper parámetros y registra los experimentos utilizando MLflow.  


## Requisitos

Python 3.11, 4 GB de RAM recomendados (más para descargar/cargar el modelo), acceso a Internet en la primera carga y, opcionalmente, Docker y Google Cloud SDK. La inferencia usa CPU.

## Ejecución local

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
pip install -r requirements-dev.txt
uvicorn app.main:app --host 0.0.0.0 --port 8080
```

Swagger: http://localhost:8080/docs; OpenAPI: http://localhost:8080/openapi.json

## API

- `GET /health`: liveness, no descarga ni valida pesos.
- `GET /ready`: inicializa el modelo y devuelve 503 si falla.
- `GET /models`: modelo activo y modelos comparados en el notebook.
- `POST /predict`: `{"text":"Me encanta este producto"}`; devuelve `sentiment`, `raw_label`, `score` y `model`. Texto de 1 a 5000 caracteres, sin cadenas solo de espacios.

```bash
curl -X POST http://localhost:8080/predict -H 'Content-Type: application/json' -d '{"text":"Me encanta este producto"}'
```

## Docker

```bash
docker build -t clasificador-sentimientos:1.0 .
docker run --rm -p 8080:8080 --memory=4g clasificador-sentimientos:1.0
sh scripts/smoke_test.sh
```

**Nota:** la primera predicción descargará pesos desde Hugging Face. El contenedor requiere salida a Internet; el inicio en frío puede tardar. No es apropiado para información confidencial sin revisión de seguridad y gobernanza.

## Pruebas

```bash
python -m pytest -q
```

Las pruebas unitarias de `POST /predict` usan un *mock*; no prueban la calidad ni la descarga real del modelo. Consultar `docs/validacion_pruebas.md`.

## Despliegue

Ver `docs/manual_despliegue_cloud_run.md`. La CI prueba la API y construye la imagen; **no publica automáticamente en GitHub ni despliega a Cloud Run**.

## Licencias y atribución


Verificar la licencia y términos del modelo de Hugging Face y del dataset original antes de su distribución o uso comercial. El notebook original incluye experimentos MLflow separados de la API.
