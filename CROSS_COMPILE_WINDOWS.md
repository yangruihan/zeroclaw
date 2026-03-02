# Windows 交叉编译到 Linux (x86_64-unknown-linux-gnu)

## 方案3：手动安装交叉编译工具链

### 方法1：使用 MSYS2 + MinGW-w64（推荐）

#### 1. 安装 MSYS2
1. 下载 MSYS2: https://www.msys2.org/
2. 运行安装程序，安装到 `C:\msys64`
3. 打开 "MSYS2 UCRT64" 终端

#### 2. 安装交叉编译工具链
```bash
# 更新包数据库
pacman -Syu

# 安装 MinGW-w64 交叉编译工具链
pacman -S mingw-w64-x86_64-toolchain

# 安装额外工具
pacman -S mingw-w64-x86_64-pkg-config
```

#### 3. 配置环境变量
在 PowerShell 中添加到 PATH：
```powershell
$env:PATH += ";C:\msys64\mingw64\bin"
```

或永久添加：
```powershell
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\msys64\mingw64\bin", "User")
```

#### 4. 编译项目
```powershell
cd f:\workspace\zeroclawstudy\zeroclaw
cargo build --release --target x86_64-unknown-linux-gnu
```

---

### 方法2：使用 WSL（Windows Subsystem for Linux）

#### 1. 安装 WSL
```powershell
# 以管理员身份运行 PowerShell
wsl --install -d Ubuntu-22.04
```

#### 2. 在 WSL 中编译
```bash
# 进入项目目录
cd /mnt/f/workspace/zeroclawstudy/zeroclaw

# 安装 Rust（如果未安装）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# 添加 Linux 目标
rustup target add x86_64-unknown-linux-gnu

# 编译
cargo build --release --target x86_64-unknown-linux-gnu
```

---

### 方法3：使用 GitHub Actions（自动化）

项目已配置自动构建流程（`.github/workflows/pub-release.yml`）：

#### 触发构建：
1. 推送 tag: `git tag v0.1.8 && git push origin v0.1.8`
2. 或手动触发: GitHub Actions → Pub Release → Run workflow

#### 下载构建产物：
- 访问 Releases 页面
- 下载 `zeroclaw-x86_64-unknown-linux-gnu.tar.gz`

---

## 已配置的编译器设置

`.cargo/config.toml` 已配置使用 Rust 内置 LLD 链接器：

```toml
[target.x86_64-unknown-linux-gnu]
linker = "rust-lld"
rustflags = ["-C", "linker-flavor=ld.lld"]
```

**注意**：某些依赖（如 `secp256k1-sys`, `aws-lc-sys`）需要编译 C 代码，必须安装完整的交叉编译工具链。

---

## 推荐方案

1. **最简单**：使用 GitHub Actions 自动构建（已有配置）
2. **本地编译**：安装 MSYS2 + MinGW-w64
3. **Linux 环境**：使用 WSL

---

## 验证工具链

```powershell
# 检查链接器是否可用
x86_64-linux-gnu-gcc --version

# 或者检查 MSYS2 工具
C:\msys64\mingw64\bin\gcc --version
```
