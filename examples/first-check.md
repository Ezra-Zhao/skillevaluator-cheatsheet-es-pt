# 第一次用 SkillEvaluator 演练 | First SkillEvaluator drill: keyless score, then decide

拿你手头一件 skill 把关键动作走一遍，先免钥匙摸底，再决定要不要花成本跑满三层：

1. **先查环境门槛**：确认 Python 在 3.12 至 3.14 之间。装好：`pip install skillevaluator`（或 `uv tool install --python 3.13 skillevaluator`）。
2. **免钥匙打分**：跑 `skillevaluator quality-check ./my-skill`。不用 API key、不用 GPU、不用 Docker，出 0 至 100 分与等级（默认 70 分及格）。先把分低的条目逐条看清依据，别急着跑下一层。
3. **认清这一分的边界**：这只是静态质量分，发布证据仍是 INCOMPLETE——安全没查、实跑没测。把它当体检的第一张化验单，不当全面报告。
4. **查能不能加载**：想查各家 harness 到底能不能加载这件 skill，要用 GitHub 源码版跑 `skillevaluator harness-check ./my-skill`（PyPI 0.9.0 版还没有这条命令，找不到不是装错了）。重点看描述被截断、目录不被扫描这几类坑。
5. **算账再决定跑满**：跑满三层要四样东西——NVIDIA Build key（开发者计划成员免费试用、有条款与限流）、外部扫描器装好、Tier 3 的 Docker 沙箱跑着、模型与托管沙箱用量可能计费。扫描器跑不起来时结果标 INCOMPLETE 并且非零退出，那是没查成，不是没事。
6. **备齐后合并跑**：`skillevaluator doctor --env-mode docker` 先查就绪，再 `skillevaluator validate ./my-skill`；只想跑前两层就加 `--tiers 1,2`。跑完记下这次的实际用量与分数变化，再决定要不要接进 CI（`skillevaluator ci init`）。
