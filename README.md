<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/images/logo-header-dark.png">
    <img alt="MCP for Unity" src="docs/images/logo-header-light.png" width="400">
  </picture>
</p>

<div align="center">

[English](README.md) <img src="docs/images/connector.svg" alt="↔" height="14"> [简体中文](docs/i18n/README-zh.md) &nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp; [Discord](https://discord.gg/y4p8KfzrN4) <img src="docs/images/connector.svg" alt="↔" height="14"> [Wiki](https://coplaydev.github.io/unity-mcp/)

#### Proudly sponsored and maintained by [Aura](https://www.tryaura.dev/) — the AI assistant for Unreal & Unity.
##### And don't miss [Godot AI](https://github.com/hi-godot/godot-ai), the new open source project from the makers of MCP for Unity.

</div>

<p align="center"><b>Create your Unity apps with LLMs.</b> MCP for Unity bridges AI assistants — Claude, Codex, VS Code, local LLMs, and more — with your Unity Editor via <a href="https://modelcontextprotocol.io/introduction">Model Context Protocol</a>. Give your LLM the tools to manage assets, control scenes, edit scripts, run tests, and automate your game dev workflows.</p>

<p align="center">
  <img alt="MCP for Unity building a scene" src="docs/images/building_scene.gif">
</p>

---

<!-- recent-updates:start -->
<details>
<summary><strong>Recent Updates</strong></summary>

* **[v10.0.0](https://github.com/CoplayDev/unity-mcp/releases/tag/v10.0.0)** (2026-06-30)
* **[v9.7.3](https://github.com/CoplayDev/unity-mcp/releases/tag/v9.7.3)** (2026-06-15)
* **[v9.7.1](https://github.com/CoplayDev/unity-mcp/releases/tag/v9.7.1)** (2026-05-24)
* **[v9.7.0](https://github.com/CoplayDev/unity-mcp/releases/tag/v9.7.0)** (2026-05-22)
* **[v9.6.8](https://github.com/CoplayDev/unity-mcp/releases/tag/v9.6.8)** (2026-04-27)

Full history: [Release Notes](https://coplaydev.github.io/unity-mcp/releases).

</details>
<!-- recent-updates:end -->

---

> **Fork notice.** This is a personal fork of [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp)
> by [Seungpyo1007](https://github.com/Seungpyo1007). All credit for MCP for Unity goes to the original authors and
> contributors. The fork adds one feature — the **Blender Bridge** described below — and otherwise tracks upstream
> `beta`. For the canonical project, docs and support, use the upstream repo.

## What's different in this fork

Upstream ships an informational "Blender → Unity Handoff" row and expects an AI client to orchestrate
[BlenderMCP](https://github.com/ahujasid/blender-mcp) and MCP for Unity separately, step by step. This fork lets the
**Unity Editor talk to the BlenderMCP addon socket directly**, so a Blender → Unity handoff is a single call and works
even without an AI client attached.

| | Upstream | This fork |
|---|---|---|
| Blender → Unity handoff | `blender-to-unity` skill: the AI exports via BlenderMCP, then imports via `import_model_file`, then places and rescales (6 manual steps) | `blender_bridge` tool, action `import_model`: export → import → place → normalize in one call |
| Blender settings in the Unity UI | Blender app detection only | **Generative tab → Blender Bridge** panel: addon socket host/port + Test Connection, blender-mcp checkout, Blender addons folder, Sync Addon / Check Updates / Import Selection |
| Keeping the Blender addon current | — | `check_updates` (git fetch of your blender-mcp checkout, behind/ahead per remote, addon md5 compare) and `sync_addon` (copy `addon.py` into Blender, with backup) |
| Editor menu | — | `Window → MCP for Unity → Blender Bridge` (Import Selection, Import Whole Scene, Viewport Screenshot, Settings) |
| CLI | — | `unity-mcp blender status / scene-info / object-info / screenshot / run-python / import-model / check-updates / sync-addon` |

### `blender_bridge` actions

| Action | What it does |
|---|---|
| `status` | Blender reachable? checkout configured? installed addon in sync with the checkout? |
| `scene_info`, `object_info` | Read Blender's scene / one object |
| `screenshot` | Viewport → PNG under `Library/BlenderBridge` (or under `Assets/` with `output_folder`) |
| `run_python` | Execute Python inside Blender, return stdout |
| `import_model` | Export `object_names` (children included) / `selection_only` / whole scene as **GLB** (keeps PBR, emission, animation) or FBX, import through the shared model pipeline, place at `position`, scale so the largest dimension is `target_size` meters |
| `check_updates` | `git fetch` the blender-mcp checkout, report commits behind `upstream`/`origin`, compare addon md5 |
| `sync_addon` | Copy the checkout's `addon.py` into Blender's addons folder (backs up the old file) |

Requirements: Blender running with the BlenderMCP addon connected (N panel → *Connect to MCP server*, default socket
`127.0.0.1:9876`); GLB import needs the glTFast package. The tool lives in the `asset_gen` group.

### Installing the fork

1. Unity → Package Manager → Add from git URL:
   `https://github.com/Seungpyo1007/unity-mcp.git?path=/MCPForUnity#feat/blender-bridge`
2. Point your MCP client at the fork's server instead of the PyPI package, e.g. in `.mcp.json`:
   ```json
   "args": ["--from", "git+https://github.com/Seungpyo1007/unity-mcp.git@feat/blender-bridge#subdirectory=Server",
            "mcp-for-unity", "--transport", "stdio"]
   ```
   (or set *Advanced → Server Source* in the MCP for Unity window and re-run *Configure*). The tool is built into the
   package, so it only appears when the fork's server is running.
3. On Windows, `git config --global core.longpaths true` — the git checkout otherwise fails on upstream's long test paths.
4. `Window → MCP for Unity → Generative → Blender Bridge`: set the blender-mcp checkout (optional; enables Sync Addon /
   Check Updates), press *Test Connection*.

Everything below this line is the upstream README.

---

## What it does

Control the Unity Editor in natural language from any MCP client — create scenes & GameObjects, edit C# scripts, manage assets, run tests, profile, and build. 48 focused MCP tool entrypoints, any client, free & MIT.

**[Browse the full tool catalog →](https://coplaydev.github.io/unity-mcp/reference/tools/)**

---

## Quickstart

**Requirements:** Unity **2021.3 LTS → 6.x** · Python **3.10+** (via [`uv`](https://docs.astral.sh/uv/)). Works with **any MCP client** — Claude Desktop & Code, Cursor, VS Code, Windsurf, Cline, Gemini CLI, and more.

1. **Install** — Unity → Package Manager → Add from git URL:
   `https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#main` &nbsp;_(pin `#v10.0.0` for this release, or `openupm add com.coplaydev.unity-mcp`)_
2. **Configure** — `Window → MCP for Unity → Configure All Detected Clients`.
3. **Prompt** — *"Create a cube at the origin and add a Rigidbody."* The cube appears in seconds.

---

## Community

- [Discord](https://discord.gg/y4p8KfzrN4) — chat with maintainers and other contributors
- [Issues](https://github.com/CoplayDev/unity-mcp/issues) — bugs and feature requests
- [Discussions](https://github.com/CoplayDev/unity-mcp/discussions) — design ideas and broader questions
- Security: see [SECURITY.md](SECURITY.md) for private reporting

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Branch off `beta`, not `main`. The full dev setup, testing, and release process live in the [Contributing](https://coplaydev.github.io/unity-mcp/contributing/dev-setup) docs.

## Advanced

- **Multiple Unity instances** — [Multi-Instance Routing](https://coplaydev.github.io/unity-mcp/guides/multi-instance)
- **Tool groups (vfx / animation / ui / testing / etc.)** — [Tool Groups](https://coplaydev.github.io/unity-mcp/guides/tool-groups)
- **v10 asset generation and upgrade notes** — [v10 Migration](https://coplaydev.github.io/unity-mcp/migrations/v10)
- **Roslyn script validation** — [Roslyn Validation](https://coplaydev.github.io/unity-mcp/guides/roslyn)
- **Remote-hosted server with auth** — [Remote Server Auth](https://coplaydev.github.io/unity-mcp/guides/remote-server-auth)

## Star History

[![Star History Chart](https://star-history.dera.page/svg?repos=CoplayDev/unity-mcp&type=Date)](https://star-history.dera.page/#CoplayDev/unity-mcp&Date)

## Citation

If MCP for Unity helped your research, please cite it.

```bibtex
@inproceedings{wu2025mcpunity,
  author    = {Wu, Shutong and Barnett, Justin P.},
  title     = {{MCP-Unity}: {Protocol-Driven} Framework for Interactive {3D} Authoring},
  year      = {2025},
  isbn      = {9798400721366},
  publisher = {Association for Computing Machinery},
  address   = {New York, NY, USA},
  url       = {https://doi.org/10.1145/3757376.3771417},
  doi       = {10.1145/3757376.3771417},
  series    = {SA Technical Communications '25}
}
```

## Unity AI Tools by Aura

Aura offers 2 AI tools for Unity:
- **MCP for Unity** is available freely under the MIT license.
- **Aura for Unity** is a premium Unity/Unreal AI assistant built for game devs.

## Disclaimer

This project is a free and open-source tool for the Unity Editor, and is not affiliated with Unity Technologies.

---

**License:** MIT — see [LICENSE](LICENSE).
