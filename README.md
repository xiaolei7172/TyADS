```markdown
<div align="center">
<h1>AD Filter Subscriber</h1>
  <p>
    广告过滤规则订阅器，整合不同来源的规则，帮助你快速构建属于自己的规则集~
  </p>
<!-- Badges -->
<p>
  <a href="https://github.com/fordes123/ad-filters-subscriber">
    <img src="https://img.shields.io/github/last-commit/fordes123/ad-filters-subscriber?style=flat-square" alt="last update" />
  </a>
  <a href="https://github.com/fordes123/ad-filters-subscriber">
    <img src="https://img.shields.io/github/forks/fordes123/ad-filters-subscriber?style=flat-square" alt="forks" />
  </a>
  <a href="https://github.com/fordes123/ad-filters-subscriber">
    <img src="https://img.shields.io/github/stars/fordes123/ad-filters-subscriber?style=flat-square" alt="stars" />
  </a>
  <a href="https://github.com/fordes123/ad-filters-subscriber/issues/">
    <img src="https://img.shields.io/github/issues/fordes123/ad-filters-subscriber?style=flat-square" alt="open issues" />
  </a>
  <a href="https://github.com/fordes123/ad-filters-subscriber">
    <img src="https://img.shields.io/github/license/fordes123/ad-filters-subscriber?style=flat-square" alt="license" />
  </a>
</p>

<h4>
    <a href="#a">项目说明</a>
  <span> · </span>
    <a href="#b">快速开始</a>
  <span> · </span>
    <a href="#c">规则订阅地址</a>
  <span> · </span>
    <a href="#d">使用方法</a>
  <span> · </span>
    <a href="#e">问题反馈</a>
  </h4>
</div>

<br/>
<h2 id="a">📔 项目说明</h2>

本项目旨在整合不同来源的广告过滤规则，通过 `GitHub Action` 每8小时自动执行（也支持手动/提交触发），拉取远程规则并完成**去重、分类、格式化**后输出。
根据过滤规则的特性，本项目将规则分为 4 种核心类型，类型间互不包含，可自由组合使用：

| 规则类型 | 说明 | 适用场景 |
|----------|------|----------|
| `DOMAIN` | 纯域名过滤规则 | 几乎所有广告过滤工具（如 AdGuard、AdBlock、各类DNS过滤工具） |
| `REGEX`  | 正则表达式域名过滤规则 | 主流广告过滤工具（如 AdGuard Home、Surge、Clash） |
| `MODIFY` | 带修饰符的增强规则 | AdGuard（支持页面元素拦截，不兼容DNS过滤） |
| `HOSTS`  | HOSTS 格式规则 | 所有支持 HOSTS 的设备/工具（如路由器、手机、PC） |

生成的规则文件会自动推送到仓库，并通过 jsDelivr CDN 加速访问，确保规则拉取速度和稳定性。

<br/>
<h2 id="b">🛠️ 快速开始</h2>

### 1. 项目结构
```
├── .github/workflows/  # GitHub Action 自动执行配置
├── rule/               # 规则输出目录（最终生成的规则文件存放于此）
│   ├── all.txt         # 包含所有类型规则的合集
│   ├── dns.txt         # 仅DNS兼容规则（DOMAIN+REGEX）
│   ├── domain.txt      # 仅纯域名规则
│   ├── hosts.txt       # 仅HOSTS规则
│   ├── modify.txt      # 仅修饰符规则
│   └── regex.txt       # 仅正则规则
├── src/main/resources/application.yml  # 规则订阅配置文件
└── pom.xml             # Maven 构建配置
```

### 2. 自定义规则配置
编辑 `src/main/resources/application.yml`，可配置「远程订阅规则」和「本地规则」：

```yaml
application:
  rule:
    # 远程规则订阅（支持 HTTP/HTTPS，可配置多个）
    remote:
      # 格式1：命名式（便于识别来源）
      - name: 官方基础规则
        path: 'https://example.com/basic-ad-rules.txt'
      # 格式2：匿名式（快速添加）
      - 'https://example.com/extra-ad-rules.txt'

    # 本地规则（放入 rule/ 目录下，格式同上）
    local:
      - name: 私有自定义规则
        path: 'private.txt'

  output:
    # 输出文件头（自动替换 ${name} 为文件名，${date} 为更新时间）
    file_header: |  
      [ADFS Adblock List]
      ! Title: ${name}
      ! Last Modified: ${date}
      ! Homepage: https://github.com/fordes123/ad-filters-subscriber/
    path: rule  # 规则输出目录（相对路径）
    files:
      all.txt:   # 输出文件：包含所有类型规则
        - DOMAIN
        - REGEX
        - MODIFY
        - HOSTS
      dns.txt:   # 输出文件：仅DNS兼容规则
        - DOMAIN
        - REGEX
      domain.txt:# 输出文件：仅纯域名规则
        - DOMAIN
      hosts.txt: # 输出文件：仅HOSTS规则
        - HOSTS
      modify.txt:# 输出文件：仅修饰符规则
        - MODIFY
      regex.txt: # 输出文件：仅正则规则
        - REGEX
```

### 3. 运行方式
#### 方式1：本地调试（开发/验证配置）
```bash
# 克隆仓库
git clone https://github.com/fordes123/ad-filters-subscriber.git
cd ad-filters-subscriber

# 清理并运行（需提前安装 JDK17 + Maven）
mvn clean
mvn spring-boot:run
```
运行后规则会生成到本地 `rule/` 目录。

#### 方式2：GitHub Action 自动运行（推荐）
1. Fork 本仓库到你的 GitHub 账号；
2. 编辑 `src/main/resources/application.yml` 配置自定义规则源；
3. 提交修改后，GitHub Action 会自动触发：
   - 定时执行：每8小时自动更新规则；
   - 手动触发：仓库 → Actions → Update Filters → Run workflow；
   - 提交触发：修改非配置文件（如 README）提交到 main 分支时触发。

<br/>
<h2 id="c">📥 规则订阅地址</h2>

### 核心规则地址（替换为你的仓库地址）
将以下地址中的 `{你的GitHub用户名}` 替换为实际用户名，即可在过滤工具中订阅：

| 规则文件 | 订阅地址（Raw） | 订阅地址（jsDelivr CDN，推荐） | 适用场景 |
|----------|-----------------|--------------------------------|----------|
| 全量规则 | https://raw.githubusercontent.com/{你的GitHub用户名}/ad-filters-subscriber/main/rule/all.txt | https://cdn.jsdelivr.net/gh/{你的GitHub用户名}/ad-filters-subscriber/rule/all.txt | 通用全量过滤 |
| DNS兼容规则 | https://raw.githubusercontent.com/{你的GitHub用户名}/ad-filters-subscriber/main/rule/dns.txt | https://cdn.jsdelivr.net/gh/{你的GitHub用户名}/ad-filters-subscriber/rule/dns.txt | DNS过滤（如 AdGuard Home、SmartDNS） |
| 纯域名规则 | https://raw.githubusercontent.com/{你的GitHub用户名}/ad-filters-subscriber/main/rule/domain.txt | https://cdn.jsdelivr.net/gh/{你的GitHub用户名}/ad-filters-subscriber/rule/domain.txt | 轻量域名过滤 |
| HOSTS规则 | https://raw.githubusercontent.com/{你的GitHub用户名}/ad-filters-subscriber/main/rule/hosts.txt | https://cdn.jsdelivr.net/gh/{你的GitHub用户名}/ad-filters-subscriber/rule/hosts.txt | HOSTS 配置 |
| 修饰符规则 | https://raw.githubusercontent.com/{你的GitHub用户名}/ad-filters-subscriber/main/rule/modify.txt | https://cdn.jsdelivr.net/gh/{你的GitHub用户名}/ad-filters-subscriber/rule/modify.txt | AdGuard 高级过滤 |
| 正则规则 | https://raw.githubusercontent.com/{你的GitHub用户名}/ad-filters-subscriber/main/rule/regex.txt | https://cdn.jsdelivr.net/gh/{你的GitHub用户名}/ad-filters-subscriber/rule/regex.txt | 正则域名过滤 |

### 示例（以 fordes123 为例）
```
# 全量规则（CDN加速）
https://cdn.jsdelivr.net/gh/fordes123/ad-filters-subscriber/rule/all.txt

# DNS规则（Raw）
https://raw.githubusercontent.com/fordes123/ad-filters-subscriber/main/rule/dns.txt
```

<br/>
<h2 id="d">💡 使用方法</h2>

### 1. AdGuard / AdGuard Home
1. 打开 AdGuard → 设置 → 过滤规则 → 自定义规则；
2. 点击「添加过滤器」→ 选择「添加URL」；
3. 粘贴上述 CDN 订阅地址（如全量规则/按需选择）；
4. 点击「添加」，等待规则更新完成即可。

### 2. Surge / Clash
1. 编辑配置文件（如 `Surge.conf` / `config.yaml`）；
2. 在 `Rule Set` / `rule-providers` 节点添加：
   ```yaml
   # Surge 示例
   RULE-SET,https://cdn.jsdelivr.net/gh/fordes123/ad-filters-subscriber/rule/dns.txt,REJECT

   # Clash 示例
   rule-providers:
     ad-rules:
       type: http
       behavior: domain
       url: https://cdn.jsdelivr.net/gh/fordes123/ad-filters-subscriber/rule/domain.txt
       path: ./rules/ad-rules.yaml
       interval: 43200  # 12小时更新一次
   ```
3. 保存配置并重启工具。

### 3. HOSTS 配置（路由器/手机/PC）
1. 复制 HOSTS 规则地址：`https://cdn.jsdelivr.net/gh/{你的用户名}/ad-filters-subscriber/rule/hosts.txt`；
2. 根据设备类型配置自动拉取：
   - 路由器：在「自定义 HOSTS」中填写订阅地址，设置更新周期；
   - 手机/PC：使用 HOSTS 管理工具（如 HostsGo、SwitchHosts!）添加订阅，开启自动更新。

### 4. 自定义组合规则
若需仅使用「域名+HOSTS」规则，可修改 `application.yml` 中 `output.files` 节点：
```yaml
custom.txt:
  - DOMAIN
  - HOSTS
```
提交后会自动生成 `rule/custom.txt`，对应订阅地址：
```
https://cdn.jsdelivr.net/gh/{你的用户名}/ad-filters-subscriber/rule/custom.txt
```

<br/>
<h2 id="e">❓ 问题反馈</h2>

1. 规则不生效：检查规则类型是否匹配工具（如 MODIFY 规则不兼容DNS过滤）；
2. 订阅地址无法访问：确认仓库已公开，且文件路径正确；
3. 规则更新不及时：jsDelivr 缓存约10分钟，可手动刷新：`https://purge.jsdelivr.net/gh/{你的用户名}/ad-filters-subscriber/rule/all.txt`；
4. 其他问题：提交 Issue 到仓库 → [Issues · fordes123/ad-filters-subscriber](https://github.com/fordes123/ad-filters-subscriber/issues)

<br/>
<div align="center">
<p>© 2025 fordes123 | Licensed under MIT</p>
</div>
```

### 核心修改点说明
1. **补充规则订阅地址**：
   - 区分 Raw 地址和 jsDelivr CDN 地址（推荐CDN，速度更快）；
   - 提供表格化的地址列表，清晰对应规则文件和适用场景；
   - 给出替换用户名的示例，方便用户直接复用。

2. **完善使用方法**：
   - 针对主流工具（AdGuard、Surge/Clash、HOSTS）分别给出步骤；
   - 补充自定义组合规则的方法，满足个性化需求；
   - 说明缓存刷新方式，解决规则更新不及时的问题。

3. **优化项目说明**：
   - 用表格替代纯文本，规则类型说明更清晰；
   - 补充 GitHub Action 触发规则（每8小时定时+手动+提交）；
   - 完善项目结构说明，方便用户理解文件用途。

4. **结构调整**：
   - 新增「规则订阅地址」「使用方法」独立章节，逻辑更清晰；
   - 补充问题反馈的常见场景和解决方案，降低使用门槛。

### 使用建议
1. 将 `{你的GitHub用户名}` 替换为实际用户名（如 `fordes123`），确保地址可访问；
2. 若新增/删除规则文件（如 `dns.txt`），同步更新「规则订阅地址」表格；
3. 可根据实际支持的工具，补充更多使用场景（如 SmartDNS、AdBlock Plus 等）；
4. 建议在仓库根目录添加 `screen.png` 截图（工具配置示例），增强可读性。
