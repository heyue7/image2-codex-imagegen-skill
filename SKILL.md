---
name: image2-codex-imagegen
description: Use when the user asks Claude to generate or save a raster image through Codex, especially after Claude-side image generation tools, gpt-image CLI, custom base URLs, or OpenAI-compatible image endpoints fail.
---

# Image2 Codex Imagegen

## Overview

Use Codex CLI's built-in `imagegen` capability as a fallback path for raster image generation when Claude-side image CLI endpoints fail or are unavailable.

Core principle: ask Codex to use its native image generation capability, save the generated file into the project, and report the exact path.

## When to Use

Use this for:
- User asks Claude to send an image generation prompt to Codex.
- `gpt-image` CLI fails due missing image accounts, proxy errors, or unavailable compatible endpoints.
- User needs a bitmap asset saved into a project path.
- Current directory may not be a git repository.

Do not use this for:
- Editing source SVG/vector/code-native assets.
- Cases where the user explicitly requires a specific OpenAI-compatible base URL or API key path.
- Placeholder images; if Codex cannot generate, report the exact limitation.

## Quick Reference

| Goal | Command pattern |
|---|---|
| Run outside git repo | `codex exec --skip-git-repo-check -C "$PWD" ...` |
| Allow project file writes | `--sandbox workspace-write` |
| Ask Codex to save image | Use the resolved destination path |
| Resolve destination | User-specified path → ask if unspecified → default only if still no fixed location |
| Avoid placeholders | Tell it: `do not write placeholder files` |
| Verify output | Check Codex final path and file exists |

## Workflow

1. Ensure Codex is available and authenticated:

```bash
codex --version
codex login status
```

If your Codex CLI does not support `login status`, run a small non-interactive check instead:

```bash
codex exec --skip-git-repo-check "Reply with: Codex is available."
```

2. Resolve the output destination before running Codex:
   - If the user specified a file or directory path, use that exact destination.
   - If the user did not specify a destination, ask where to save the image.
   - If there is still no fixed location after that, use the project default `PROJECT/fig/`.

3. Run Codex directly, not through a worktree agent, if the current project is not a git repository:

```bash
codex exec --skip-git-repo-check \
  -C "/absolute/project/path" \
  --sandbox workspace-write \
  "Try to directly generate an image from this prompt using any Codex-native image generation capability available to you. If you can generate it, save it under <RESOLVED_DESTINATION> and return the full saved path. If you cannot generate images directly, return the exact limitation/error and do not write placeholder files. Prompt: <IMAGE_PROMPT>"
```

4. Treat Codex output as status:
   - Success: Codex returns a real image path, usually after copying from `~/.codex/generated_images/...` into the resolved destination.
   - Failure: report the exact Codex limitation or error.

5. Verify before claiming success:

```bash
ls -l "<RETURNED_IMAGE_PATH>"
```

## Known Successful Pattern

Codex may load its own `imagegen` skill and use the built-in image tool, then copy the generated file from:

```text
~/.codex/generated_images/<session-id>/<generated-file>.png
```

into the resolved destination path, for example:

```text
PROJECT/fig/anime-fashion-cafe-portrait.png
```

This path does not depend on the Claude-side `gpt-image` CLI or a custom OpenAI-compatible `base_url`.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Using a subagent with worktree isolation in a non-git directory | Use `codex exec --skip-git-repo-check` directly |
| Assuming Codex auth means Claude's image CLI works | They are separate paths |
| Defaulting to `PROJECT/fig/` when the user did not specify a path | Ask for the destination first; use default only when no fixed location remains |
| Letting Codex create placeholders | Explicitly forbid placeholders |
| Reporting generation success without checking the file | Run `ls -l` or equivalent on the returned path |
| Omitting `-C` | Set Codex working root to the intended project |
