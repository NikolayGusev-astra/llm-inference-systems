# UBTree runtime: состояние SGLang-интеграции

Статус на 30.10.2026.

## Итог

**Рабочий путь — ветка `asahi/ubtree`; native upstream-интеграция — открытый PR. До merge курс не утверждает upstream support.**

## Открытый PR: sgl-project/sglang#44050

- Ссылка: https://github.com/sgl-project/sglang/pull/44050
- Состояние на 30.10.2026: **open**, [WIP]; автор AsaHIXx; создан 2026-10-13, обновлён 2026-10-22; +7730/−189 в 43 файлах.
- Название: «Adds the SGLang kernel and runtime primitives required by UBTree».
- Labels: speculative-decoding, jit-kernel, memory-pool.
- Что добавляет (по описанию PR):
  - загрузку draft checkpoint и построение proposal tree;
  - selector caches;
  - **factorized KDA tree verification**: один общий pre-tree recurrent state + root-to-node факторы, без материализации полного состояния на узел; rejected branches не коммитятся;
  - **fused causal convolution + tree transform** (один fused kernel, меньше промежуточного трафика);
  - **accepted-path state commit** (только принятый root-to-leaf путь складывается в персистентное состояние);
  - **fixed-width CUDA Graph decode** для UBTree-пути (стабильные буферы tree metadata, accepted-path commit и target KV movement в хвосте графа);
  - **FA3 tree-verification metadata** (QLEN-only tree mask и т.п.).
- Мотивация из PR: для моделей с KDA linear attention сохранение полного recurrent state на каждый узел дерева даёт значительный memory traffic и overhead на rejected-ветвях.

## Рабочий путь (по странице проекта и model card)

Ветка: https://github.com/AsaHIXx/sglang/tree/asahi/ubtree

Требования (по model card/сайту): Python 3.11+, CUDA; для Ling3-Flash — TP4 (4 GPU), attention-backend fa3.

Пример запуска (batch-size-1 конфигурация из model card; проверять флаги на конкретной версии SGLang перед использованием):

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 \
SGLANG_OPT_FA3_VERIFY_METADATA_IN_GRAPH=1 \
SGLANG_OPT_UBTREE_FA3_TREE_ONLY_MASK=1 \
python -m sglang.launch_server \
  --model-path /path/to/ling3-flash \
  --served-model-name ling3-flash \
  --tp-size 4 \
  --host 0.0.0.0 \
  --port 30000 \
  --attention-backend fa3 \
  --page-size 1 \
  --context-length 4096 \
  --max-total-tokens 4096 \
  --max-running-requests 1 \
  --mem-fraction-static 0.80 \
  --max-prefill-tokens 4096 \
  --chunked-prefill-size 4096 \
  --disable-radix-cache \
  --mamba-backend triton \
  --linear-attn-backend triton \
  --linear-attn-verify-backend triton \
  --mamba-ssm-dtype float32 \
  --cuda-graph-backend-prefill disabled \
  --cuda-graph-backend-decode full \
  --cuda-graph-bs-decode 1 \
  --disable-cuda-graph-padding \
  --tool-call-parser ling3 \
  --reasoning-parser ling3 \
  --trust-remote-code \
  --speculative-algorithm UBTREE \
  --speculative-draft-model-path /path/to/ling3-flash-ubtree \
  --speculative-num-draft-tokens 32
```

Draft-чекпоинт: https://huggingface.co/inclusionAI/Ling-3.0-flash-UBTree
Target: https://huggingface.co/inclusionAI/Ling-3.0-flash

## Ограничения, которые курс фиксирует

- Не обещать поддержку произвольных target: чекпоинт парный (inclusionAI/Ling-3.0-flash), draft не самостоятельная модель.
- Числа 5.36× (сайт) и 3.428× E2E (model card) — авторские замеры, не независимые; в бенчмарк model card: block size 16, TP4, concurrency 1, BF16, greedy no-thinking, tree 32.
- До merge PR модуль 11 не получает полноценный serving-рецепт: только этот отложенный блок с явным статусом.

## Что проверять перед публикацией

1. Статус PR #44050 (open → merged?) и ветки asahi/ubtree.
2. Совпадение launch-флагов с конкретной версией SGLang.
3. Lossless-контроль на своём железе перед переносом цифр авторов.