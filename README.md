# droot — 高性能根域名推断引擎

droot 是一个**高性能、数据驱动、可扩展的根域名（root-domain）推断引擎**，  
用于从大规模域名列表中准确识别**真实的可注册根域名**。

> 在海量域名中准确识别 root-domain，避免伪 TLD、伪二级域名、异常 TLD、脏数据污染结果。

---

## 技术基础

- **ICANN Root Zone Database** — 权威 TLD 数据来源
- **TLD 分类体系** — original / generic / brand / country / sponsored / infrastructure
- **pseudo-SLD** — 伪二级域名构造器（如 `com.cn`、`net.jp`）
- **pseudo-SLD:CN** — 伪二级域名构造器（中国特有，如 `bj.cn`、`hk.cn`）
- **domain.2 / domain.3** — 结构化域名拆分策略
- **ripgrep（rg）** — 正则表达式高速匹配引擎
- **sort --parallel** — 多核并行排序

---

## 功能特性

### ✔ PSL-like 后缀识别能力

完整的 TLD 分类体系，比 Public Suffix List 更干净、更可控：

| 分类 | 说明 |
|---|---|
| `original` | 原始通用顶级域（.com .net .org 等） |
| `generic` | 新通用顶级域（.app .dev .shop 等） |
| `brand` | 品牌专属顶级域（.google .apple 等） |
| `country` | 国家和地区代码顶级域（.cn .uk .jp 等） |
| `sponsored` | 赞助顶级域（.edu .gov .mil 等） |
| `infrastructure` | 基础设施域（.arpa） |
| `pseudo-sld` | 伪二级域名构造器 |
| `pseudo-sld:cn` | 伪二级域名构造器(中国特有) |
---

### ✔ 精准的 root-domain 推断

通过结构化拆分策略组合使用：

- `domain.2` — 二级结构提取（`example.com`）
- `domain.3` — 三级结构提取（`example.co.uk`）

结合以下分类的剔除 / 保留规则，实现专业级 root-domain 推断：

- `country.country`
- `original.country`
- `sponsored.country`
- `pseudo-sld.country`
- `pseudo-sld:cn.cn`

---

### ✔ 高性能（千万级域名秒级处理）

在 **8 核 CPU** 上的实测性能：

| 数据集 | 规模 | 耗时 |
|---|---|---|
| 测试业务域名 | 381 MB / 1000 万行 | ~17 秒 |
| 实际业务域名 | 100 万行 | ~1.5–2 秒 |

测试数据：Domcop Top 10 million domains
https://www.domcop.com/files/top/top10milliondomains.csv.zip


得益于：

- `sort --parallel` 多核并行排序
- `rg`（ripgrep）高速正则匹配
- 减少 I/O 的结构优化
- 合并 TLD 分类处理，减少中间步骤

---

### ✔ 完全数据驱动

所有推断规则来自外部数据文件，脚本本身不写死任何 TLD：

```
/
├── RootZoneDatabase.csv   ← ICANN 根区分类数据库(基于IANA官方修改)
└── RootZoneDatabase.txt   ← ICANN 根区数据库文本(基于IANA官方，转成小写)
```

如需更新规则，只需替换对应数据文件，无需修改脚本逻辑。

---

### ✔ 干净的输出

输出 root-domain 时，自动过滤以下脏数据，确保可靠可用：

- 空行
- `-domain.com`（前导连字符）
- `domain-.com`（标签尾部连字符）
- `domain.-com`（TLD 前导连字符）
- `domain.com-`（尾部连字符）

---

## Pseudo-SLD（伪二级域名构造器）
## Pseudo-SLD:CN（伪二级域名构造器）

pseudo-SLD 及Pseudo-SLD:CN（中国特有）是各国用于构造二级国家域名的"伪 TLD"。  
真正的注册主体不是在 TLD 下，而是在这个伪二级域名下。

**示例：**

| pseudo-SLD | 原型 | 说明 |
|---|---|---|
| `com.cn` | `com` | 中国商业域 |
| `bj.cn` | `cn` | 中国(北京）二级域 |
| `net.cn` | `net` | 中国网络域 |
| `edu.cn` | `edu` | 中国教育域 |
| `gov.cn` | `gov` | 中国政府域 |
| `co.uk` | `co` | 英国商业域 |
| `net.jp` | `net` | 日本网络域 |

因此 `www.example.com.cn` 的 root-domain 是 `example.com.cn`，而非 `com.cn`。

---

## 依赖

| 工具 | 用途 |
|---|---|
| `bash` 5.0+ | 脚本运行环境 |
| `rg`（ripgrep） | 高速正则过滤 |
| `sort` | 并行排序（支持 `--parallel`） |
| `awk` | 字段处理与查表 |

---

## 配置

**使用前，需修改脚本中的数据库文件路径**，默认路径为：

```bash

DATA="/etc/geo/data/RootZoneDatabase.csv"
# IAIA 官方数据 https://data.iana.org/TLD/tlds-alpha-by-domain.txt
# 使用之前，把内容转为小写
# tr '[:upper:]' '[:lower:]' < tlds-alpha-by-domain.txt \
#   | sort --parallel="$(nproc)" -u -o RootZoneDatabase.txt
TLD="/etc/geo/data/RootZoneDatabase.txt" 

```

如数据文件存放于其他路径，打开 `droot.sh` 修改脚本顶部的路径变量。


---

## 用法

```bash
droot /path/to/input_file /path/to/output_file
```

| 参数 | 说明 |
|---|---|
| `input_file` | 输入文件**绝对路径**，每行一个域名或 URL |
| `output_file` | 输出文件**绝对路径**，每行对应提取后的根域名 |

**示例：**

```bash
droot /data/domains/raw.txt /data/domains/roots.txt
```

输入 `/data/domains/raw.txt`：

```
static.cdn.blog.example.co.uk
foo.bar.com.cn
mail.pku.edu.cn
www.google.com
-bad.example.com
```

输出 `/data/domains/roots.txt`：

```
example.co.uk
bar.com.cn
pku.edu.cn
google.com
example.com
```

> 注意：`-bad.example.com`虽然前导连字符，但 root domain 合法，不属于脏数据，允许保留。
> 脏数据检测是针对 output，不针对 input。
> `-example.com` 才属于脏数据。

---

## 推断逻辑说明

```
输入域名
    │
    ▼
规范化（去协议头 / 路径 / 脏数据过滤）（预先处理，droot 不负责）
    │
    ▼
TLD 校验（对照 RootZoneDatabase.txt）
    │
    ├─ 未知 TLD ──→ 丢弃 / domain.2 回退
    │
    └─ 已知 TLD
          │
          ▼
     TLD.EX 及 pseudo-SLD:CN 查询
    （label[-2] + TLD 是否在需要剔除二级域名表中？）
          │
     ┌────┴────┐
   命中       未命中
     │         │
     ▼         ▼
 domain.3    domain.2
（3 标签）   （2 标签）
```

| 输入 | 输出 | 路径 |
|---|---|---|
| `www.example.com` | `example.com` | domain.2 |
| `a.b.co.uk` | `b.co.uk` | pseudo-SLD 命中 → domain.3 |
| `mail.pku.bj.cn` | `pku.bj.cn` | pseudo-SLD:CN 命中 → domain.3 |
| `www.moe.gov.cn` | `moe.gov.cn` | pseudo-SLD:CN 命中 → domain.3 |

---

## 特殊补丁

主分类管道完成后，还需要两个补充补丁来修正特定结构的域名，
避免通用规则在边界情况下产生的误判。

---

### 补丁 1：`country.country` — 国家域下的国家类二级域

**问题**

某些国家同时将本国国家代码注册为二级域，形成 **SLD 与 TLD 相同**的结构，例如：

```
cn.cn
us.us
jp.jp
```

即域名的 SLD 和 TLD 都是同一个国家代码。
这类结构不在 pseudo-SLD 表中，主管道默认走 `domain.2`，会错误输出：

```
example.cn.cn  →  cn.cn  ← 错（SLD=TLD=cn，注册主体在第三层）
example.us.us  →  us.us  ← 错
example.jp.jp  →  jp.jp  ← 错
```

**补丁行为**

当检测到 **SLD == TLD**（且两者均为 country 类标签）时，强制走 `domain.3`：

```
example.cn.cn  →  example.cn.cn   （domain.3，正确）
example.us.us  →  example.us.us   （domain.3，正确）
example.jp.jp  →  example.jp.jp   （domain.3，正确）
```

**在管道中的位置**

```
主分类管道输出
      │
      ▼
┌─────────────────────────────────┐
│  country.country 补丁           │
│  TLD ∈ country &&              │
│  label[-2] ∈ country_labels ?  │
│  是 → 升级为 domain.3           │
│  否 → 保持主管道结果             │
└─────────────────────────────────┘
```

---

### 补丁 2：`\.cn$` — 中国域名全量兜底

**问题**

`.cn` 下的二级体系极为复杂：

- 功能性 SLD：`com.cn` `net.cn` `org.cn` `gov.cn` `edu.cn` `ac.cn` `mil.cn`
- 省市地方 SLD：`bj.cn` `sh.cn` `gz.cn` `tj.cn` `cq.cn` … 30+ 个

pseudo-SLD 表无论收录多完整，都存在遗漏省市条目的风险。
一旦某个省市 SLD 未被收录，主管道会输出错误的 `domain.2`
这时就需要 pseudo-SLD:CN 再次处理：

```
example.bj.cn  →  bj.cn  ← 错（应为 example.bj.cn）
```

**补丁行为**

对所有匹配 `\.cn$` 的域名，**无论 pseudo-SLD 表是否命中**，
统一强制执行 `domain.3` 提取策略：

```
域名以 .cn 结尾
      │
      ▼
pseudo-sld:cn 查询
      │
      ├─ 未命中 ──→ 保留 domain.2（补丁不介入）
      │
      └─ 命中 ──→ 提取 domain.3
                  label[-3] + label[-2] + ".cn"
```

**示例**

| 输入 | 主管道结果 | `\.cn$` 补丁后 |
|---|---|---|
| `mail.pku.edu.cn` | `pku.edu.cn` ✓ | `pku.edu.cn` ✓（一致） |
| `blog.example.bj.cn` | `bj.cn` ✗ | `example.bj.cn` ✓ |
| `www.moe.gov.cn` | `moe.gov.cn` ✓ | `moe.gov.cn` ✓（一致） |

> **注意**：`gov.cn` / `edu.cn` 本身作为注册主体直接使用时（即输入仅有两个标签），
> 补丁不做升级，保持 `domain.2` 输出，避免越界取空标签。

---

### 两个补丁的完整执行顺序

```
输入域名
    │
    ▼
规范化 → TLD 校验
    │
    ▼
TLD.EX 查询 → domain.2 / domain.3 (整合了补丁 1：country.country 修正)
    │
    ▼
补丁 2：\.cn$ 修正
    │
    ▼
最终 root-domain 输出
```

两个补丁均为**幂等操作**：若主管道结果已正确，补丁不改变输出。


