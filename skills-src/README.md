# Skills Source

本目录包含独立的、按主题划分的技能实现。构建流水线根据显式配置处理这些技能。

## 使用方法

为你的技能创建一个子目录，并添加 `SKILL.md` 文件。

```
skills-src/
  my-skill/
    SKILL.md
```

每个 `SKILL.md` 文件的 frontmatter 都应以 `name` 和 `description` 开头，其中 `name` 必须与子目录名一致。

如需帮助创建技能内容，请参阅 [`skill-creator`](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md?plain=1) 技能。

每个技能目录只能有一个 `SKILL.md` 文件，但可以在 `resources` 或 `examples` 子目录中添加额外资源或示例。例如：

```
skills-src/
  my-skill/
    SKILL.md
    resources/
      my-skill-1.txt
      my-skill-2.txt
    examples/
      demo.html
      demo.js
      demo.css
```