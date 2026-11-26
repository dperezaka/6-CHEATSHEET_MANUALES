| APLICACIÓN                 | TIPO                     | USO                                                                                                 |
| -------------------------- | ------------------------ | --------------------------------------------------------------------------------------------------- |
| pip                        | Instalación de paquetes  | Instala paquetes de PyPI                                                                            |
| venv                       | Entornos virtuales       | Crea entornos aislados por proyecto                                                                 |
| pipx                       | Herramientas de terminal | Instala aplicaciones de línea de comandos, cada una en un entorno aislado                           |
| Poetry                     | Gestión de proyectos     | Gestiona dependencias con archivo de bloqueo, el entorno y la publicación de paquetes               |
| uv                         | Gestión de proyectos     | Gestor todo en uno y muy rápido; sustituye a pip, venv, pipx, Poetry y pyenv                        |
| pyproject.toml             | Configuración            | Archivo estándar con los metadatos, las dependencias y los ajustes de las herramientas              |
| Type hints                 | Lenguaje                 | Anotaciones de tipo que documentan el código y permiten detectar errores                            |
| mypy                       | Comprobación de tipos    | Verifica los tipos sin ejecutar el código                                                           |
| Pyright / ty               | Comprobación de tipos    | Alternativas a mypy, más rápidas                                                                    |
| Pydantic                   | Validación de datos      | Valida y convierte datos en tiempo de ejecución (APIs, JSON, configuración)                         |
| pydantic-settings          | Configuración            | Carga la configuración y los secretos desde variables de entorno y archivos `.env`                  |
| Ruff                       | Linter y formateador     | Analiza el estilo y los errores, ordena los *imports* y formatea; sustituye a flake8, isort y black |
| Bandit                     | Seguridad                | Detecta vulnerabilidades en tu código                                                               |
| pip-audit                  | Seguridad                | Detecta vulnerabilidades conocidas en las dependencias                                              |
| pytest                     | Pruebas                  | Marco estándar de pruebas                                                                           |
| pytest-cov                 | Pruebas                  | Mide la cobertura de las pruebas                                                                    |
| Hypothesis                 | Pruebas                  | Pruebas basadas en propiedades que generan casos automáticamente                                    |
| nox / tox                  | Pruebas                  | Ejecutan las pruebas en varias versiones de Python                                                  |
| pre-commit                 | Automatización           | Ejecuta las comprobaciones antes de cada *commit*                                                   |
| GitHub Actions / GitLab CI | Integración continua     | Ejecuta pruebas y comprobaciones en cada cambio subido                                              |
| Docker                     | Despliegue               | Empaqueta la aplicación con su entorno de forma reproducible                                        |
| MkDocs / Sphinx            | Documentación            | Generan la documentación del proyecto                                                               |
| structlog / logging        | Registro                 | Registros estructurados para producción                                                             |
