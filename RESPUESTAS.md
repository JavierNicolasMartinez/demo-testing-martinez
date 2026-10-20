# Respuestas de la guía práctica

**Alumno:** Martínez, Javier Nicolás  
**Materia:** Taller de Lenguajes de Programación IV — Unidad 4

## 1. Conocer la aplicación

- La regla para calcular el precio final está en `src/services/precios.ts`.
- La ruta HTTP está definida en `src/app.ts`.
- El servicio se ocupa de validar los valores y calcular el total; la aplicación Express recibe la solicitud, invoca el servicio y traduce el resultado o el error a una respuesta HTTP.
- La guía indica `GET /healthy`, mientras que el proyecto original ya exponía `GET /salud`. Se conservaron ambas rutas para respetar la guía sin quitar la ruta que ya existía.

## 2. Diagnóstico inicial

La primera ejecución de `npm test` tuvo **4 pruebas aprobadas y 1 fallida**. La prueba fallida exigía que un descuento del 150 % lanzara `RangeError`, pero la función original no validaba el descuento ni lanzaba excepciones. Aplicaba directamente la fórmula y devolvía **-500** para un precio de 1.000.

Que la función termine de ejecutarse no significa que cumpla la regla de negocio. Vitest falla porque esperaba una excepción que nunca se produjo.

## 3. Validaciones y pruebas unitarias

El servicio ahora rechaza con `RangeError`:

- Precios negativos o no finitos.
- Descuentos negativos, mayores que 100 o no finitos.

Para entradas válidas se conserva la fórmula original. Se agregaron pruebas para el descuento del 100 %, el precio negativo, el descuento negativo y `NaN` como precio. El descuento del 100 % produce cero; los demás casos inválidos de la tabla provocan `RangeError`.

`Number.isFinite` también rechaza `NaN`, por lo que la prueba adicional confirma el desafío de la guía.

## 4. Prueba de integración

La prueba con Supertest envía un descuento del 150 % a la ruta HTTP y comprueba que recibe **HTTP 400** y una propiedad `error`. Es una prueba de integración porque recorre la ruta de Express y el servicio de cálculo juntos; las pruebas unitarias, en cambio, llaman directamente a la función.

## 5. Pruebas, tipos e integración continua

- `npm test` ejecuta los casos automatizados y comprueba comportamientos concretos.
- `npm run typecheck` ejecuta `tsc --noEmit`: analiza errores de tipos sin ejecutar las pruebas ni generar JavaScript.
- Conviene ejecutar ambos porque verifican clases distintas de problemas.
- GitHub Actions automatiza esas verificaciones al integrar cambios, ayuda a detectar fallas temprano y deja el resultado registrado. No garantiza por sí solo que no haya defectos.

## 6. Preguntas de cierre

1. Una comprobación manual requiere que una persona ejecute los pasos e interprete el resultado; una prueba automatizada repite verificaciones definidas en código de forma rápida y consistente. Las pruebas manuales siguen siendo útiles para explorar y evaluar aspectos de uso.
2. La primera falla se debía a que la implementación no validaba el rango del descuento y, por ello, no lanzaba el `RangeError` esperado.
3. Los casos inválidos verifican que el sistema rechace entradas fuera de las reglas, en lugar de producir resultados incorrectos que podrían parecer válidos.
4. `tests/precios.test.ts` comprueba directamente la función de cálculo. `tests/api.test.ts` comprueba la interacción entre la aplicación HTTP y el servicio.
5. Reejecutar todas las pruebas después de un cambio permite comprobar tanto el comportamiento nuevo como los anteriores y detectar regresiones.

## 7. Registro de resultados

| Comprobación | Resultado | Observación |
| --- | --- | --- |
| `GET /healthy` y `GET /salud` | HTTP 200 | La ruta de la guía y la ruta original responden `{ "estado": "ok" }`. |
| Primera ejecución de `npm test` | 4 aprobadas, 1 fallida | Falló el rechazo del descuento del 150 %. |
| Corrección del servicio | Correcta | Entradas inválidas generan `RangeError`. |
| Pruebas unitarias ampliadas | Correctas | Casos de límite, negativos y `NaN`. |
| Integración con descuento del 150 % | HTTP 400 | La respuesta contiene `error`. |
| Ejecución final de `npm test` | 11 aprobadas | 7 unitarias y 4 de integración. |
| `npm run typecheck` | Correcto | `tsc --noEmit` finalizó sin errores. |
