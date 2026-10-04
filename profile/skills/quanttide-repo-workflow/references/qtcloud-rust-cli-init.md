# qtcloud 应用 Rust CLI 初始化（src/cli）

参考实例：qtcloud-econ/src/cli（2026-08-09 建立，qtcloud-econ-cli，命令名 `qtcloud-econ`）。
参考模板：qtcloud-devops/src/cli（lib+bin 分离、clap derive、tests/ 目录）。

## 结构

```
src/cli/
├── Cargo.toml        # [lib] crate-type lib + [[bin]]（同 qtcloud-devops 模式）
├── .gitignore        # /target
├── src/lib.rs        # pub mod <module>;（领域模块从库暴露，便于单测）
├── src/<module>.rs   # serde 模型 + load() + #[cfg(test)] 单测
└── src/main.rs       # clap Parser/Subcommand，命令按领域模块组织
```

Cargo.toml 要点：

```toml
[package]
name = "qtcloud-<domain>-cli"          # 如 qtcloud-econ-cli
version = "0.1.0"
edition = "2021"
license = "Apache-2.0"

[lib]
name = "qtcloud_<domain>_cli"          # lib 名下划线
path = "src/lib.rs"
crate-type = ["lib"]

[[bin]]
name = "qtcloud-<domain>"              # 命令名带连字符，如 qtcloud-econ
path = "src/main.rs"

[dependencies]
clap = { version = "4", features = ["derive"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

## 子命令模式（mechanism 实例）

```
qtcloud-<domain> mechanism list                # 列出（id/名称/四要素计数）
qtcloud-<domain> mechanism show <id>           # 详情（参与者/策略/规则/目标分节打印）
```

- `main.rs`：`#[command(name, about, version, disable_help_subcommand(true))]` + `global = true` 的 `--path` 参数（种子数据路径覆盖）
- `mechanism.rs`：`#[derive(Deserialize)]` 模型对齐 JSON 种子（`#[serde(rename = "player_id")]` 处理 snake_case 字段）；`load(path) -> Result<Vec<M>, String>` 返回 String 错误便于 CLI 直接打印
- 测试放模块内 `#[cfg(test)] mod tests`：单测构造样例对象测 summarize；集成测从 `CARGO_MANIFEST_DIR` 相对路径读真实种子

## 关键坑

1. **共享种子数据路径（踩过）**：CLI 与 Studio 读**同一份**种子数据（单一事实源），位置在 `src/studio/assets/data/`。CLI 的 `default_path()` 必须是**仓库根相对** `"src/studio/assets/data/<file>.json"`——写成 `"assets/data/..."` 会从仓库根运行时报 `No such file or directory`。测试里用 `Path::new(env!("CARGO_MANIFEST_DIR")).join("../../src/studio/assets/data/...")`（CARGO_MANIFEST_DIR = src/cli，上两级=仓库根）。
2. **cargo test 输出多段**：lib/bin 各有一个 "running N tests" 块——`tail -5` 可能只看到最后一段的 0 tests 误判"没测试"；用 `grep -E "running|test result"` 看全。
3. **fmt 差异**：write_file 写入的 .rs 可能触发 rustfmt 格式 lint 报错（Diff 提示），提交前统一 `cargo fmt`。
4. **首次 build 慢**：clap 等依赖首次编译约 1 分钟，正常。

## 验证

```bash
export PATH="$HOME/.cargo/bin:$PATH"
cd <repo>/src/cli && cargo fmt && cargo build && cargo test 2>&1 | grep -E "test result"
# 运行验证（从仓库根）：
cd <repo> && ./src/cli/target/debug/qtcloud-<domain> mechanism list
```
