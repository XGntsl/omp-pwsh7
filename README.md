# omp-pwsh7 (fork)

Fork of [kouhe3/omp-pwsh7](https://github.com/kouhe3/omp-pwsh7) — a persistent PowerShell 7 `pwsh` tool
extension for [omp](https://github.com/can1357/oh-my-pi) (oh-my-pi).

一个给 omp 增加"常驻 PowerShell 7 会话"工具（`pwsh`）的扩展。一次启动，变量 / 模块 / 当前目录跨调用保持，
所以 `Import-Module` 只付一次代价，热调用约 20 ms（冷启动约 450 ms）。

## Why this fork exists / 为什么要 fork

Upstream `package.json` does not declare `omp.extensions`, so

上游 `package.json` 没有声明 `omp.extensions` 字段，因此：

```sh
omp plugin install 'github:kouhe3/omp-pwsh7#bun'
```

copies the files and installs the dependencies, but **never loads the extension factory** — the `pwsh` tool is
not registered and **no error is reported**. This fork adds that one field:

文件会被装进 `plugins/node_modules/omp-pwsh7`、依赖也会装好，但**扩展工厂函数不会被发现** —— `pwsh` 工具不注册，
而且**没有任何报错**。本 fork 只补这一个字段：

```json
"omp": { "extensions": ["./index.ts"] }
```

Nothing else is changed. / 其他内容一律未改。

## Install / 安装

```sh
omp plugin install 'github:XGntsl/omp-pwsh7#main'
```

Then **restart omp** — extension factories are initialized at session startup; `/reload-plugins` does not
rebuild them.

装完**重启 omp**：扩展工厂只在会话启动时初始化，`/reload-plugins` 不会重建它。

Requirements / 前置条件：

- PowerShell 7 (`pwsh` on `PATH`; the executable name is hardcoded, there is no setting for a custom path
  and no Windows PowerShell 5.1 fallback)
- Windows for the timeout/abort path (it kills the process tree with `taskkill`)
- omp ≥ 18.2 (verified on omp 18.3.0 / pwsh 7.6.6 / bun 1.4.2)

## Update / 更新

```sh
omp plugin install --force 'github:XGntsl/omp-pwsh7#main'
```

`--force` is required: without it the command prints `✔ Installed` but is a no-op, because the previous
commit stays pinned in `plugins/bun.lock`.

必须带 `--force`：不带的话命令会打印 `✔ Installed` 但实际什么都没做，旧 commit 仍被 `plugins/bun.lock` 钉住。

## Notes / 注意事项

- The tool is named `pwsh`. It collides with any other `pwsh` custom tool
  (`~/.omp/agent/tools/pwsh.ts`); when both load, one is **silently dropped**.
- State persists: `Set-Location` drifts and stray variables survive. Pass the `session` argument to get a
  separate process pool when you need a clean session.
- `exit N` inside a command does not surface an exit code (the session survives); native commands do
  (`cmd /c exit 7` → `exitCode: 7`).
- Plugin installs pull only `dependencies` (`shiki`, ~14 MB). Running `bun install` inside a clone also
  pulls `devDependencies`, which drags in the whole omp tree (~800 MB) — use `bun install --production`.

## Sync with upstream / 同步上游

```sh
git remote add upstream https://github.com/kouhe3/omp-pwsh7
git fetch upstream
git rebase upstream/bun        # the patch touches only package.json
git push --force-with-lease origin main
```
