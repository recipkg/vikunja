# Vikunja 配方（提取中）

```toml
format = "recipkg/1"

[package]
name = "vikunja"
version = "0.1.3"
kind = "software"
description = "Vikunja 自托管任务管理产品的重建配方；当前仍在提取。"
entry = "BUILD_PROMPT.md"

[dependencies.quality]
git = "https://github.com/recipkg/recipkg.git"
rev = "41e5b55c6612a9a5a10e499b5ace6a1a3fed0f2b"
subdir = "recipes/software-quality"

[dependencies.design]
git = "https://github.com/recipkg/design-directions.git"
rev = "6fdba350d1effa2d50dc0185095bb91ca7a699ee"
```

来源：[go-vikunja/vikunja](https://github.com/go-vikunja/vikunja)，源码提交 `e37ebc0d10db366ff111bc4bb74a696d7cfc912e`。上游 README 标注主体为 AGPL-3.0-or-later，`desktop/` 为 GPL-3.0-or-later，背景图另有许可。此草案不代表配方完整或已独立重建，见 `COVERAGE.md`。
