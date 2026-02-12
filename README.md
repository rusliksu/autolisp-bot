# AutoLISP Bot

AI-powered assistant for structural engineers. Generates AutoLISP scripts for AutoCAD reinforcement detailing.

## What is this?

A system prompt + examples for an AI bot that helps structural engineers (who are NOT programmers) generate AutoLISP scripts by describing tasks in plain language.

**Example:** "Draw a beam cross-section with 4d16 bottom reinforcement, 2d12 top, stirrups d8 step 200mm" → ready `.lsp` script.

## Files

- `system-prompt.md` — System prompt with AutoCAD commands, DXF codes, Russian building codes (SP 63, GOST 34028), reinforcement typification rules
- `examples.md` — 9 example scripts: column grids, beam sections, stirrups, rebar layouts, specifications, mass calculator, rebar typification

## Coverage

### AutoLISP
- Drawing commands (LINE, CIRCLE, DONUT, PLINE, HATCH, INSERT, ARRAY)
- DXF codes for reading/modifying objects
- Layer management for reinforcement drawings
- Interactive input (getpoint, getreal, getint)

### Russian Building Codes
- **GOST 34028-2016** — Rebar grades: diameters, cross-sections, mass per meter
- **SP 63.13330.2018** — Concrete cover, bar spacing, anchorage lengths by concrete class (B15-B35), lap splice lengths
- Rebar classes: A240, A400, A500C, A600

### Reinforcement Typification
- Grouping parameters (diameter, class, count, length, stirrup spacing)
- Naming convention (KR, KP, S, D)
- Position numbering rules

## Status

Work in progress. Waiting for real-world examples from a structural engineering team to refine the prompt.

## License

MIT
