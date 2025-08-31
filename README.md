# Proyecto Demo: Flask + Pruebas (PyTest) + BDD (Behave) + CI (GitHub Actions) + Performance (Locust)

Objetivo: mostrar un flujo simple y profesional de **integración continua** con pruebas unitarias atómicas, BDD, reporte HTML y una prueba básica de performance.

## Estructura
```
flask-bdd-ci/
├─ app/
│  ├─ __init__.py
│  ├─ main.py
│  └─ auth.py
├─ tests/
│  ├─ test_math.py
│  └─ test_app.py
├─ features/
│  ├─ environment.py
│  ├─ login.feature
│  └─ steps/
│     └─ login_steps.py
├─ performance/
│  └─ locustfile.py
├─ .github/workflows/ci.yml
├─ requirements.txt
├─ .gitignore
└─ README.md
```

## Instalación local
```bash
python -m venv .venv
source .venv/bin/activate  # En Windows: .venv\Scripts\activate
pip install -r requirements.txt
export FLASK_APP=app.main:app
flask run  # http://127.0.0.1:5000
```

## Pruebas unitarias (atómicas)
```bash
pytest -q --cov=app --cov-report=term-missing --html=reports/pytest-report.html --self-contained-html
```

## BDD con Behave (con reporte HTML)
```bash
mkdir -p reports
behave -f html -o reports/behave-report.html
```

## Performance con Locust (headless)
```bash
# Ejecuta 10 usuarios por 30s, tasa de 2 usuarios/s
locust -f performance/locustfile.py --headless -u 10 -r 2 -t 30s --csv=reports/locust --only-summary
```

Los reportes se guardan en `reports/` y pueden adjuntarse como artefactos de CI.

## CI con GitHub Actions
El workflow `ci.yml` compila, ejecuta linters, pruebas unitarias, BDD y performance smoke. Publica artefactos con HTML y CSV.

## Buenas prácticas incluidas
- Commits frecuentes y con mensajes claros (sugerencia en tareas).
- Pruebas **atómicas** y **determinísticas**.
- Separación de responsabilidades (rutas en `main.py`, lógica en `auth.py`).
- Reportes navegables (pytest HTML y behave HTML).
- Pipeline en cada **push** y **pull request**.
- Métricas básicas de coverage, tiempos y errores (pytest/behave) + TPS/latencia (Locust).

## Notas sobre atomicidad de tests
- Cada prueba se ejecuta independiente, sin depender del orden.
- El estado de la app se aísla con el **Flask test client** en unit tests y BDD.

## Simulación "Three Amigos" (Resumen)
- **Producto**: Define criterios de aceptación para login: respuesta 200 en `GET /login`, login exitoso con credenciales válidas, error claro para inválidas.
- **Desarrollo**: Endpoints y validaciones mínimas (no persistencia).
- **QA**: Escenarios Gherkin (+ Outline) y reporte HTML. Integración en CI.

## Métricas / Dashboard
En GitHub Actions:
- **Cobertura** de código (pytest-cov) en consola.
- **HTML de pruebas** (pytest y behave) como artefactos.
- **Performance**: CSV de Locust (TPS, latencia media/p95, tasa de error). Se podrían graficar en un tablero (e.g., GitHub Pages + simple JS, o enviar a un ELK/InfluxDB+Grafana).

## Alertas automáticas
- Falla de job en CI envía notificación estándar (reviewer).
- Umbrales simples (ej.: si latencia p95 > 500ms o tasa de error > 1%) pueden forzar `exit 1` en el job de performance.

> Este proyecto es una realización en Python/Flask del taller que pide CI, BDD, atomicidad, métricas y reporting.


## Jenkins (alternativa a GitHub Actions)
Incluí un `Jenkinsfile` con etapas equivalentes (lint, unit tests con cobertura y HTML, BDD con HTML, smoke de Locust y artefactos). En un Jenkins con agentes Linux estándar debería correr sin plugins exóticos.

## Inicializar repo Git con historia de commits (simulada)
Hay dos scripts que crean el repo, agregan archivos y generan **commits con mensajes** en ramas separadas para que veas una historia ordenada:
- PowerShell (Windows): `scripts/init_git_history.ps1 -RepoName mi-repo`
- Bash (Linux/macOS): `scripts/init_git_history.sh mi-repo`

Luego puedes asociar el remoto y subir:
```bash
git remote add origin <URL-DEL-REPO>
git push -u origin main
```

## Windows helper
- `scripts/windows_setup_and_run.ps1` automatiza: venv, deps, levanta Flask en otra consola, corre pytest/behave/locust y deja reportes en `reports/`.
