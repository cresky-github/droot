# droot — A High‑Performance Root‑Domain Inference Engine

droot 是一个 **高性能、数据驱动、可扩展的 Root‑Domain 推断引擎**，  
用于从大规模域名列表中识别 **真实的根域名（root-domain）**。

它基于：

- ICANN Root Zone Database  
- TLD 分类体系（original / generic / brand / country / sponsored / infrastructure）  
- pseudo‑SLD（伪二级域名构造器）  
- 结构化域名拆分（domain.2 / domain.3）  
- 正则表达式匹配引擎（ripgrep）  
- 高性能排序（sort --parallel）

droot 的目标是：

> **在海量域名中准确识别 root-domain，避免伪 TLD、伪二级域名、异常 TLD、脏数据污染结果。**

---

## Features

### ✔ PSL‑like 后缀识别能力  
droot 拥有完整的 TLD 分类体系：

- original  
- generic  
- brand  
- country-code  
- sponsored  
- infrastructure  
- pseudo‑sld（伪二级域名构造器）  
- all  

比 Public Suffix List 更干净、更可控。

---

### ✔ 精准的 root-domain 推断  
通过结构化拆分：

- `domain.2.dset` → 二级结构  
- `domain.3.dset` → 三级结构  

并结合：

- original.country  
- sponsored.country  
- pseudo‑sld.country  

的剔除/保留规则，实现专业级 root-domain 推断。

---

### ✔ 高性能（百万级域名秒级处理）  
在 8 核 CPU 上：

- **110MB / 690 万行域名 → 12 秒**
- **100 万行实际业务域名 → 1.5–2 秒**

得益于：

- sort --parallel  
- rg（ripgrep）  
- 减少 I/O 的结构优化  
- 合并 TLD 分类处理  

---

### ✔ 完全数据驱动  
所有规则来自：

- RootZoneDatabase.csv  
- pseudo‑sld 列表  
- TLD 分类文件  

脚本本身不写死任何 TLD。

---

### ✔ 干净的输出  
自动清洗：

- 空行  
- `-domain.com`  
- `domain-.com`  
- `domain.-com`  
- `domain.com-`  

确保输出 root-domain 可靠可用。

---

## Pseudo‑SLD（伪二级域名构造器）

pseudo‑sld 是各国用于构造二级国家域名的“伪 TLD”，例如：
com -> com.cn
net -> net.jp
