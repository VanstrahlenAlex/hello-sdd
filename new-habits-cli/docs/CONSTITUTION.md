# Constitución — new-habits-cli
Principios innegociables: toda spec, plan y tarea debe cumplirlos.

1. **Stack mínimo**: Python 3.12+ y solo biblioteca estándar; única dependencia
   de desarrollo permitida, `pytest`. Verificable en `pyproject.toml`.
2. **La spec manda**: nada se implementa si no está en la spec activa; si falta
   una decisión, se detiene el trabajo y se pregunta antes de codificar.
3. **Núcleo sin interfaz**: `core` no imprime, no lee teclado ni toca disco;
   `cli` solo parsea, llama y formatea. El core se testea sin la CLI.
4. **Tests como puerta**: cada tarea cierra con `pytest -q` en verde y un test
   por regla de rachas y por comando. Prohibido avanzar en rojo.
5. **Datos locales y legibles**: un único JSON con campo `version`; sin red ni
   base de datos. Cambiar el formato exige spec y ruta de migración.
6. **Idioma**: identificadores y código en inglés; mensajes de usuario, ayuda
   y documentación en español.
