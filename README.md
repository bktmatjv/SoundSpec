# SoundSpec Compiler

Trabajo parcial de **Teoría de Compiladores** (Sección 13962).

**Integrantes**
- Vanessa Barrientos Villalta – U202413287
- Matias Javier Del Castillo Mendoza – U202411305
- Vivianne Fatima Ríos Hasegawa – U202410135

SoundSpec es un lenguaje para describir pistas musicales (`track`), efectos (`effect`) y mezclas (`mix`). Este repositorio contiene el **avance de la implementación del compilador**: el frontend completo (análisis léxico, sintáctico y semántico) sobre ANTLR 4.13.2 con target Python.

## Estado del avance

| Fase | Estado | Archivos |
|---|---|---|
| Análisis léxico | Completo | `grammar/SoundSpecLexer.g4`, `soundspec/lexico.py` |
| Análisis sintáctico | Completo | `grammar/SoundSpecParser.g4`, `soundspec/sintactico.py` |
| Análisis semántico (SEM-01 a SEM-06) | Completo | `soundspec/semantico.py` |
| Plan de validación (34 casos) | Completo | `tests/` |
| Generación de código con LLVM (Hito 3) | Pendiente | — |

### Errores semánticos verificados

| Código | Error | Regla |
|---|---|---|
| SEM-01 | Identificador redeclarado | Cada track, effect y mix tiene un nombre único |
| SEM-02 | Referencia no declarada | La pista y los efectos de un mix deben estar declarados |
| SEM-03 | Nota fuera del rango audible | 20 Hz ≤ frecuencia ≤ 20 000 Hz |
| SEM-04 | Compás incompleto | Las duraciones de `notes` deben completar compases enteros |
| SEM-05 | Categoría incorrecta en mix | La fuente debe ser un track y la cadena solo admite effects |
| SEM-06 | Duración inválida | Numerador y denominador deben ser mayores que 0 |

## Ejemplo del lenguaje

```
track BassLine { bpm: 120; wave: SAWTOOTH; notes: [C2:1/4, E2:1/4, G2:1/2]; }
effect Reverb1 { delay: 200ms; decay: 0.5; }
mix Song = BassLine -> Reverb1;
```

## Estructura

```
grammar/      Gramáticas ANTLR (lexer y parser)
soundspec/    Paquete Python del compilador (+ generated/ con el parser de ANTLR)
examples/     Programas SoundSpec (incluye "Estrellita, ¿dónde estás?")
tests/        Plan de validación: aceptación, léxico, sintáctico, semántico, integración
scripts/      generar_parser.sh
```

## Cómo ejecutarlo

Requisitos: Python 3.10+ y Java 11+.

```bash
pip install -r requirements.txt
./scripts/generar_parser.sh          # descarga ANTLR y genera soundspec/generated/
python -m soundspec examples/estrellita.soundspec
python -m soundspec examples/error_semantico.soundspec
pytest -v                            # ejecuta los 34 casos del plan de validación
```

## Notas de diseño

- Los tipos de onda y las notas (`C4`, `Bb2`) se declaran antes que `ID` en el lexer, por lo que funcionan como palabras reservadas.
- `ID` solo admite letras ASCII, dígitos y guion bajo; `PadAtmósfera` produce un error léxico.
- El semántico usa dos recorridos del árbol: el primero llena la tabla de símbolos y el segundo valida, de modo que un `mix` puede aparecer antes de las declaraciones que referencia.
