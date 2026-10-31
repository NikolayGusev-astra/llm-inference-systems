# [2609.39972] UBTree: Parallel Tree Drafting via Unigram and Bigram Models for Speculative Decoding

URL: https://arxiv.org/abs/2609.39972

- Дата: v1 30.09.2026, v2 28.10.2026. Inclusion AI + UIUC + Purdue.
- Статус: препринт; независимых реплик в первоисточниках не зафиксировано на 30.10.2026.

## Механика

- **Unigram proposer** предлагает top-`k` кандидатов независимо для каждой позиции (обучен стандартным cross-entropy);
- **Bigram selector** за один проход оценивает переходы между соседними кандидатами;
- дешёвый **search** превращает оценки в verification tree;
- selector обучается **tree-native**: renormalized KL на high-temperature трассах — супервизия шире greedy-пути, ранжирует альтернативные ветви;
- target верифицирует дерево → lossless-контракт speculative decoding сохраняется.

## Место в ландшафте

- UBTree — **внешний tree drafter**: отдельный обученный draft checkpoint рядом с target. Не self-speculative (см. модуль 15 курса).
- Отличие от DARTree (2608.13524): DARTree расширяет предобученную AR correction head к деревьям training-free; UBTree обучает proposer/selector и делает tree-native training.
- Отличие от DFlash (2602.06036): DFlash — параллельный block drafter, коммитящийся в один путь; UBTree покрывает многие ветви, сохраняя параллельность.
- DSpark — фронтовой baseline на production-оценке (5.36× против 4.50× на Ling3-Flash 124B, SGLang TP4).

## Заявленные результаты (все — авторские, не независимые)

| Источник | target | Setup | Результат |
|---|---|---|---|
| arXiv abs | Qwen3-4B/8B | 7 стандартных бенчмарков, greedy | avg speedup 5.84–6.94× |
| arXiv abs | Qwen3-4B/8B | 7 бенчмарков × конфигурации | 28/28 wins над DARTree |
| inclusion-ai.org/ubtree | Qwen3-8B | greedy, 7 benchmarks | 6.94× avg speedup |
| inclusion-ai.org/ubtree | Ling3-Flash 124B | SGLang, TP4 | 5.36× vs 4.50× DSpark |
| inclusion-ai.org/ubtree | Ling3-Flash 124B | LongBench-v2, промпты до 260K | +37.6% acceptance length vs DSpark |
| model card B16 | Ling3-Flash | TP4, concurrency 1, BF16, greedy, tree 32, 6 датасетов | avg 9.16 acceptance / 3.428× E2E / 1009.6 tok/s |

Model card (Ling3-Flash-B16): acceptance length = возвращённые токены / verify-вызовы target (с anchor); TPS = возвращённые токены / полное клиентское HTTP-время (с prefill); E2E speedup = TPS / AR-TPS. Acceptance и TPS — медианы трёх прогонов; AR — один прогон на бенчмарк.

## Драфт-чекпоинт (model card)

- Парный к target inclusionAI/Ling-3.0-flash; не самостоятельная модель.
- 481,983,774 параметра в BF16; 5 full-attention слоёв DFlash-бэкбона; selector обучен отдельно.
- Block size 16 (anchor-inclusive), candidate top-k 64, дефолтный verification tree 64 non-root nodes + anchor.
- Serving example SGLang использует tree budget 32: `--speculative-algorithm UBTREE --speculative-num-draft-tokens 32`.

## Ссылки

- Статья: https://arxiv.org/abs/2609.39972
- Сайт: https://www.inclusion-ai.org/ubtree/
- Model card: https://huggingface.co/inclusionAI/Ling-3.0-flash-UBTree
- Target: https://huggingface.co/inclusionAI/Ling-3.0-flash
- Runtime-статус: https://github.com/sgl-project/sglang/pull/44050 (см. papers/ubtree-runtime.md)
- Соседи: papers/ubtree-runtime.md, papers/dartree.md