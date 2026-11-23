# SkillEvaluator 速查表 | SkillEvaluator Cheatsheet | Guía rápida de SkillEvaluator

> 📚 **Colección / Coleção**: esta hoja forma parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 hojas de referencia en un solo repositorio. / Esta folha faz parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 folhas de referência em um só repositório. ⭐

**NVIDIA 给 skill 做体检的三层框架——先免钥匙打个分，再看第一屏的成本卡，实跑对照的账算清了再跑**

> 一页看懂 NVIDIA/SkillEvaluator：NVIDIA 开源的 agent skill 评测框架，属 NVIDIA Verified Skills 流水线。它评的是 skill 这个工件本身，不是模型也不是 agent：写得规范吗、安全吗、各家 harness 真能加载吗、和别的 skill 有没有语义重复、装上到底有没有用。这张卖的是三层各答一问、免钥匙第一步和成本卡——跑满三层要什么、哪里会标 INCOMPLETE，都在第一屏说清。

- 上游项目：[NVIDIA/SkillEvaluator](https://github.com/NVIDIA/SkillEvaluator)（2026-11-17 实测 3,253 ★ / 460 forks，11 个 release，Apache-2.0，根目录只有英文 README）
- 平台：Python 3.12–3.14 的 CLI；可接 CI（GitHub / GitLab）；也可装进 Claude Code、Codex 等 agent 里直接问它
- 本页结构：中文主文 → English → Español → Português
- 同系列：给 agent 用的审计方法论看 [security-audit 速查表](https://github.com/Ezra-Zhao/security-audit-cheatsheet-es-pt)；这张评的是 skill 工件本身的质量与净贡献，两张互补

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 第一屏·三层各答一问卡

| 层 | 回答的问题 | 怎么跑（原仓口径） |
|---|---|---|
| Tier 1 验证 | 写得规范吗、安全吗、各家 harness 能加载吗 | `skillevaluator tier1 ./my-skill --no-llm` 做静态检查；`skillevaluator harness-check ./my-skill` 专查各家 harness 到底能不能加载（只在 GitHub 源码版、PyPI 0.9.0 版还没有）。全量静态覆盖要另装外部扫描器，安全扫描由同厂的 SkillSpector 提供 |
| Tier 2 查重 | 指令内部、skill 之间有没有语义重复 | `skillevaluator tier2 ./my-skill`，靠 embeddings 与对话模型比对；加 `--catalog ./skill-catalog.json` 才做 skill 之间的横向比对，没跑横向比对时报告会明说 |
| Tier 3 实跑 | 装上到底有没有用 | `skillevaluator tier3 ./my-skill`：同一批任务装与不装各跑一遍，差值就是这件 skill 的净贡献（Skill Lift）。要 provider 访问、一个受支持的 agent 与跑着的沙箱（默认 Docker） |

三层互相独立，可以只跑一层。合并跑：先 `skillevaluator doctor --env-mode docker` 查就绪状态，再 `skillevaluator validate ./my-skill`——不带 `--tiers` 默认三层全跑，`--tiers 1,2` 跳过实跑。门槛口径（原仓自述）：Tier 1 是 `validate` 的硬门；Tier 2 默认也拦，除非加 `--no-block-on-dedup`；Tier 3 只有加 `--block-on-agent-eval` 才拦。

## 免钥匙第一步卡

| 口径 | 实际情况（原仓自述） |
|---|---|
| 一条命令 | `pip install skillevaluator` 后跑 `skillevaluator quality-check ./my-skill`（或 `uv tool install --python 3.13 skillevaluator`） |
| 什么都不用备 | 不用 API key、不用扫描器、不用 GPU、不用 Docker |
| 出什么 | 0 至 100 分加一个等级，附每条发现的依据；及格线默认 70 分 |
| 这一分的边界 | 只是静态质量分。发布证据仍标 INCOMPLETE——安全与实跑影响要另查，别拿这一分当全面体检报告 |

## 第一屏·成本卡（本张的命门）

| 口径 | 实际情况（原仓自述） |
|---|---|
| 跑满三层要 NVIDIA Build key | 开发者计划成员可在 build.nvidia.com 免费试用 NVIDIA 托管端点，有条款与限流；key 覆盖 Tier 2 的 embeddings 与对话、Tier 3 的受支持 agent |
| 要装外部扫描器 | Tier 1 全量静态覆盖要按装法备好系统扫描器（Semgrep 可选）；`--no-llm` 免的是模型检查，免不了扫描器要求 |
| Tier 3 要 Docker 沙箱 | 还要模型与托管沙箱用量，可能计费，先看原仓的 runtime 与成本说明再跑 |
| 扫描器跑不起来不是没事 | 扫描器跑不起来时结果标 INCOMPLETE 并且非零退出——是「没查成」，不是「查过没事」。别把没查成读成通过 |

全量装法：`pip install "skillevaluator[all]"`，再 `export SKILL_EVAL_LLM_PROVIDER=nv_build` 与 `export NVIDIA_API_KEY='nvapi-...'`，然后 `skillevaluator validate ./my-skill`。

## 三条入口一行版

| 入口 | 命令 | 适合谁 |
|---|---|---|
| CLI 直跑 | `skillevaluator quality-check ./my-skill` 先免钥匙打分，备齐后再 `skillevaluator validate ./my-skill` | 手头有一件 skill 想先摸底的人 |
| 接进 CI | 在仓库里跑 `skillevaluator ci init`，它开一个设置 PR，之后只查变更过的 skill | skill 多、要守住每次改动质量的仓库 |
| 装进 agent | `npx skills add NVIDIA/SkillEvaluator`，然后直接问 agent「我的 skill 怎么样」 | 日常在 Claude Code、Codex 里干活、懒得切终端的人 |

harness-check 查的是装不上还不自知的坑：它按各家 harness 的版本化档案逐个核——描述被截断、目录不被扫描、描述长度超了某家的上限，都会逐项报出来（原仓示例里六家有两家加载不了）。这件检查只在 GitHub 源码版里，PyPI 的 0.9.0 版还没有，装 PyPI 版找不到这条命令不是你装错了。

## 诚实标注区

- **支持级别 Experimental**：社区支持、尽力而为，无 SLA、无 NVIDIA 企业支持（原仓 SUPPORT 口径）。拿它当评测参考可以，当生产门禁前先想清这一条。
- **版本口径**：当前 PyPI 版 0.9.0；harness-check 还只在源码版。原仓带同行评议论文（arXiv 2608.20614），其口径是静态分与实跑效果的相关性很低——这正是它要分三层的原因：写得好不等于装上有用（论文口径，证据 B）。
- **势头数据**：3,253 ★ / 460 forks、11 个 release（2026-11-17 原仓页与 gh API 实测，证据 A）；今日第三方榜位在列、单日约 +873 至 +953 星为第三方统计口径（证据 B）。
- **先免钥匙、后算账**：第一步不要钱也不要 key，先打分；要跑满三层，先把 key、扫描器、Docker 与可能的计费用量这四笔账算清，再决定跑不跑。

---

## English (summary)

SkillEvaluator (3,253 stars / 460 forks as of 2026-11-17, 11 releases, Apache-2.0) is NVIDIA's open-source framework for evaluating agent skills and plugins, part of the NVIDIA Verified Skills pipeline. It evaluates the skill artifact itself — not the model, not the agent — in three independent tiers, each answering one question. Tier 1, validation: is the skill safe, well-formed and harness-compatible (`skillevaluator tier1 ./my-skill --no-llm`, plus `harness-check`, which checks whether each harness will actually load the skill — description truncation, unscanned directories, per-harness length limits; source-only for now, not in PyPI 0.9.0). Tier 2, deduplication: does guidance repeat inside the skill or overlap other skills, checked with embeddings and a chat model (`--catalog` enables inter-skill comparison, and the report says so when that part did not run). Tier 3, live agent evaluation: the same tasks run with and without the skill, and the difference is the skill's net contribution, its Skill Lift. Start keyless: `pip install skillevaluator`, then `skillevaluator quality-check ./my-skill` needs no API key, no scanner, no GPU and no Docker, and prints a score out of 100 with a grade (passing defaults to 70) — but that score is static quality only, publication evidence stays INCOMPLETE, and security and agent impact need the other checks. Cost card, read before running all tiers: the full pipeline needs an NVIDIA Build key (free prototyping for eligible Developer Program members, subject to terms and rate limits), external scanners installed (a scanner that cannot run reports INCOMPLETE and exits nonzero — that means "not checked", not "checked and fine"), and a running Docker sandbox for Tier 3, where model and managed-sandbox usage can incur charges. Combined runs go through `skillevaluator doctor --env-mode docker` then `skillevaluator validate ./my-skill` (`--tiers 1,2` skips live evaluation; Tier 1 always gates, Tier 2 gates unless `--no-block-on-dedup`, Tier 3 gates only with `--block-on-agent-eval`). Three entry points: the CLI directly, CI via `skillevaluator ci init` (opens a setup PR and then checks only changed skills), or inside your agent via `npx skills add NVIDIA/SkillEvaluator` — just ask "Is my skill any good?". Honest notes: support level is Experimental, community-supported with no SLA and no NVIDIA enterprise support; the bundled paper's point is that static scores correlate weakly with live effect, which is exactly why the tiers are separate. Requires Python 3.12–3.14.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

**El chequeo médico de las skills según NVIDIA, en tres capas — primero la puntuación sin llave, y lee la tarjeta de costos antes de correr la evaluación en vivo.**

SkillEvaluator (3.253 estrellas / 460 forks a 17 de noviembre de 2026, 11 releases, Apache-2.0) es el marco abierto de NVIDIA para evaluar skills y plugins de agentes, parte de la línea NVIDIA Verified Skills. Evalúa el artefacto skill en sí — no el modelo ni el agente — en tres capas independientes, cada una con su pregunta. Capa 1, validación: ¿está bien formada, es segura y la cargan los distintos harness? (`skillevaluator tier1 ./my-skill --no-llm`, más `harness-check`, que comprueba si cada harness la carga de verdad: descripciones truncadas, directorios que no se escanean, límites de longitud por harness; por ahora solo en la versión de código fuente, no en la 0.9.0 de PyPI). Capa 2, deduplicación: ¿la guía se repite dentro de la skill o se solapa con otras? Se compara con embeddings y un modelo de conversación; con `--catalog` se activa la comparación entre skills, y el informe dice cuándo esa parte no corrió. Capa 3, evaluación en vivo: las mismas tareas corren con y sin la skill, y la diferencia es su contribución neta, el Skill Lift.

Primer paso sin llave: `pip install skillevaluator` y luego `skillevaluator quality-check ./my-skill` — sin clave API, sin escáner, sin GPU y sin Docker — da una nota de 0 a 100 con su nivel (el aprobado por defecto es 70). Pero esa nota es solo calidad estática: la evidencia de publicación queda INCOMPLETE, y la seguridad y el impacto real se comprueban aparte. Tarjeta de costos, la pantalla clave de esta hoja: correr las tres capas pide una clave de NVIDIA Build (pruebas gratuitas para miembros elegibles del programa de desarrolladores, con términos y límites), escáneres externos instalados (un escáner que no puede correr marca INCOMPLETE y sale con código distinto de cero: significa «no comprobado», no «comprobado y bien»), y un sandbox Docker en marcha para la capa 3, donde el uso de modelos y sandboxes gestionados puede generar cargos. La corrida combinada es `skillevaluator doctor --env-mode docker` y después `skillevaluator validate ./my-skill` (`--tiers 1,2` salta la evaluación en vivo). Tres entradas: la CLI directa, CI con `skillevaluator ci init` (abre un PR de configuración y luego solo revisa las skills que cambiaron), o dentro de tu agente con `npx skills add NVIDIA/SkillEvaluator`, preguntándole «¿qué tal es mi skill?». Notas honestas: el nivel de soporte es Experimental, comunitario y sin SLA ni soporte empresarial de NVIDIA; el artículo que lo acompaña sostiene que la nota estática se correlaciona poco con el efecto real — justo por eso las capas van por separado. Requiere Python 3.12 a 3.14. Esta página es una síntesis con mis propias palabras a partir de la documentación pública del proyecto, no una traducción de su README.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

**O check-up das skills segundo a NVIDIA, em três camadas — primeiro a nota sem chave, e leia o cartão de custos antes de rodar a avaliação ao vivo.**

O SkillEvaluator (3.253 estrelas / 460 forks em 17 de novembro de 2026, 11 releases, Apache-2.0) é o framework aberto da NVIDIA para avaliar skills e plugins de agentes, parte da linha NVIDIA Verified Skills. Ele avalia o artefato skill em si — não o modelo nem o agente — em três camadas independentes, cada uma com a sua pergunta. Camada 1, validação: está bem formada, é segura e os harness carregam? (`skillevaluator tier1 ./my-skill --no-llm`, mais o `harness-check`, que confere se cada harness carrega de verdade: descrições truncadas, diretórios que não são varridos, limites de tamanho por harness; por ora só na versão de código-fonte, não na 0.9.0 do PyPI). Camada 2, deduplicação: a orientação se repete dentro da skill ou se sobrepõe a outras? A comparação usa embeddings e um modelo de conversa; com `--catalog` entra a comparação entre skills, e o relatório avisa quando essa parte não rodou. Camada 3, avaliação ao vivo: as mesmas tarefas rodam com e sem a skill, e a diferença é a contribuição líquida dela, o Skill Lift.

Primeiro passo sem chave: `pip install skillevaluator` e depois `skillevaluator quality-check ./my-skill` — sem chave de API, sem scanner, sem GPU e sem Docker — dá uma nota de 0 a 100 com o nível (a aprovação padrão é 70). Mas essa nota é só qualidade estática: a evidência de publicação fica INCOMPLETE, e segurança e impacto real se conferem à parte. Cartão de custos, a tela-chave desta folha: rodar as três camadas pede uma chave do NVIDIA Build (testes gratuitos para membros elegíveis do programa de desenvolvedores, com termos e limites), scanners externos instalados (um scanner que não roda marca INCOMPLETE e sai com código diferente de zero: significa «não verificado», não «verificado e ok»), e um sandbox Docker de pé para a camada 3, onde o uso de modelos e sandboxes gerenciados pode gerar cobrança. A rodada combinada é `skillevaluator doctor --env-mode docker` e depois `skillevaluator validate ./my-skill` (`--tiers 1,2` pula a avaliação ao vivo). Três entradas: a CLI direta, CI com `skillevaluator ci init` (abre um PR de configuração e depois só revisa as skills que mudaram), ou dentro do seu agente com `npx skills add NVIDIA/SkillEvaluator`, perguntando «minha skill é boa?». Notas honestas: o nível de suporte é Experimental, comunitário e sem SLA nem suporte empresarial da NVIDIA; o artigo que o acompanha sustenta que a nota estática se correlaciona pouco com o efeito real — justamente por isso as camadas são separadas. Requer Python 3.12 a 3.14. Esta página é uma síntese com as minhas próprias palavras a partir da documentação pública do projeto, não uma tradução do README dele.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [NVIDIA/SkillEvaluator](https://github.com/NVIDIA/SkillEvaluator)（2026-11-17 实测 3,253 ★ / 460 forks，11 个 release，许可证 Apache-2.0；官方文档 docs.nvidia.com/skills/skillevaluator，星数为同日 GitHub 实测）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文，不冒充官方；三层口径、命令与成本以 2026-11-17 的上游 README 为准，上游迭代后请以官方文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao（本仓自有内容；上游项目为 Apache-2.0）
