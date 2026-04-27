# image2-codex-imagegen-skill

Claude Code skill for generating and saving raster images through Codex imagegen; requires Codex CLI and the Claude Code Codex plugin.  
通过 Codex imagegen 生成图片并保存到项目路径的 Claude Code Skill；需要提前安装 Codex CLI 和 Claude Code Codex 插件。

## What it does

This skill documents a practical fallback workflow for image generation from Claude Code:

- delegate image generation prompts to Codex;
- use Codex's native `imagegen` capability;
- save generated bitmap assets into a resolved project path;
- avoid placeholder files;
- verify the final image path before reporting success.

## Requirements

- Claude Code
- Codex CLI installed and authenticated
- Claude Code Codex plugin installed/enabled
- A Codex runtime that supports the built-in `imagegen` skill

Check Codex availability:

```bash
node "~/.claude/plugins/cache/openai-codex/codex/1.0.2/scripts/codex-companion.mjs" setup --json "check codex image generation availability"
```

If Codex is not authenticated, run:

```bash
codex login
```

## Skill name

```yaml
name: image2-codex-imagegen
```

## Files

```text
SKILL.md   # Skill instructions
README.md  # Repository documentation
LICENSE    # MIT License
```

## Usage summary

1. Resolve the output destination:
   - use the user's specified file or directory path;
   - if no destination is specified, ask where to save;
   - only fall back to `PROJECT/fig/` if there is still no fixed location.
2. Run Codex directly with `codex exec --skip-git-repo-check` when the project is not a git repository.
3. Ask Codex to use native image generation and save the real output to the resolved path.
4. Verify the returned file exists before reporting success.

Example command pattern:

```bash
codex exec --skip-git-repo-check \
  -C "/absolute/project/path" \
  --sandbox workspace-write \
  "Try to directly generate an image from this prompt using any Codex-native image generation capability available to you. If you can generate it, save it under <RESOLVED_DESTINATION> and return the full saved path. If you cannot generate images directly, return the exact limitation/error and do not write placeholder files. Prompt: <IMAGE_PROMPT>"
```

## License

MIT
