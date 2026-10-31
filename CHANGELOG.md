# Changelog

## [1.1.0] - 2026-10-31

- Обновление ландшафта speculative decoding: UBTree (2609.39972), DFlash (2602.06036), DARTree (2608.13524).
- Модуль 10: подраздел «UBTree: diversity-first parallel tree drafting» с таблицей авторских замеров (источник/target/hardware/benchmark/budget/baseline), serving-status (ветка asahi/ubtree, upstream PR sglang#44050 открыт — upstream support не утверждается).
- Модуль 15: cross-link — UBTree и DARTree как внешние драфтеры, явное отделение от self-speculation.
- Модуль 16: headline-дивергенция UBTree (6.94× сайт vs 5.84–6.94× аннотация vs 3.428× model card) как пример несопоставимости авторских цифр.
- Новые материалы: papers/ubtree.md, papers/dartree.md, papers/ubtree-runtime.md, reviews/ubtree-2026-10-30.md.
- Модуль 11: serving-рецепт отложен до стабилизации upstream-интеграции SGLang.

## [1.0.0] - 2026-10-30

- Первый публичный релиз курса.
- 18 модулей: от matmul и attention до unified memory и гибридных нод.
- Добавлены runnable-материалы, SVG-иллюстрации, планы, спецификации и исследовательские заметки.
