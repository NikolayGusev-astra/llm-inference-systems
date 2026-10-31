# [2608.13524] DARTree: Speculative Diffusion Decoding with Autoregressive Draft Trees

URL: https://arxiv.org/abs/2608.13524

- VILA Lab, MBZUAI. Код: https://github.com/VILA-Lab/DARTree
- Статус: препринт; авторские замеры.

## Механика

- Training-free метод: расширяет **предобученную autoregressive correction head от цепочек к деревьям**;
- строит fixed-width дерево кандидатов: все узлы каждой глубины расширяются и скорируются одним батчем;
- best-first pruning выбирает verification tree — AR-head inference отделена от последовательных heap-операций;
- задача: позиционные распределения diffusion-драфтеров — маргинальные, не обусловленные токенами вдоль пути; DARTree добавляет каузальную коррекцию вдоль ветвей дерева.

## Заявленные результаты (авторские, не независимые)

- 7 бенчмарков (math/code/chat), 4 конфигурации «модель × температура»: наивысшая средняя acceptance length и speedup во всех четырёх;
- до 12.97 принятых токенов за раунд верификации: +98.6% против DFlash, +27.9% против Domino в той же конфигурации;
- до 9.73× lossless speedup против локально замеренного AR-декодинга.

## Место в ландшафте

- Ближайший сосед UBTree (2609.39972): оба строят verification tree. Отличие: DARTree training-free (расширяет готовую AR head), UBTree обучает proposer/selector с tree-native лоссом.
- DFlash (2602.06036) — diffusion block drafter, от которого DARTree наследует постановку (маргинальные position-wise распределения), но добавляет каузальную коррекцию по ветвям.
- Оба — внешние драфтеры, не самоспекуляция (см. модуль 15 курса).