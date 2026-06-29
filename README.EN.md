[中文版本](README.md)        [Change Log](change.log.md)

# droot — High‑Performance Root‑Domain Inference Engine 

droot is a **high‑performance, data‑driven, extensible root‑domain inference engine**,  
used to accurately identify **real registrable root‑domains** from large‑scale domain lists.

> Accurately identify root‑domains in massive domain datasets, avoiding fake TLDs, pseudo‑SLDs, abnormal TLDs, and dirty data contamination.

---

## Terminology

- TLD (Top-Level Domain)
- SLD (Second-Level Domain)

> These are the two core hierarchical levels that form Internet domain names.

---

## Technical Foundations
---

### ✔ Fully data‑driven — all inference rules come from external data files:

- RootZoneDatabase.csv ← Root zone database classification, based on [IANA official](https://www.iana.org/domains/root/db), modified with [TLD WIKI](https://en.wikipedia.org/wiki/List_of_Internet_top-level_domains) and [MIIT China Internet Domain System](https://domain.miit.gov.cn/)
- RootZoneDatabase.txt ← Root zone database text, converted to lowercase from [IANA official](https://data.iana.org/TLD/tlds-alpha-by-domain.txt)

> To update rules, simply modify the corresponding data files — no need to change script logic.  
> RootZoneDatabase.csv cannot directly use the official file.  
> The official source only contains two categories: generic and country.

---

### ✔ Data Classification

- **ICANN Root Zone Database** — authoritative TLD data source  
- **TLD classification system** — original / generic / brand / country / sponsored / infrastructure  
- **pseudo-sld** — pseudo second‑level domain constructors (global, e.g., `com.cn`, `net.jp`)  
- **pseudo-sld:cn** — pseudo second‑level domain constructors (China‑specific, e.g., `bj.cn`, `hk.cn`)  
- **domain.2 / domain.3** — structured domain splitting strategy  
- **ripgrep (rg)** — high‑speed regex matching engine  
- **sort --parallel** — multi‑core parallel sorting  

> pseudo-sld:cn data source:

- [MIIT China Internet Domain System](https://domain.miit.gov.cn/), removing entries duplicated with pseudo‑sld to reduce I/O:
    - ac
    - com
    - edu
    - gov
    - mil
    - net
    - org

- [MIIT China Internet Domain System](https://domain.miit.gov.cn/), retaining provincial SLDs:

    |sld:cn|Province Name|
    |---|---|
    |bj|Beijing|
    |tj|Tianjin|
    |sh|Shanghai|
    |cq|Chongqing|
    |he|Hebei|
    |sx|Shanxi|
    |nm|Inner Mongolia|
    |ln|Liaoning|
    |jl|Jilin|
    |hl|Heilongjiang|
    |js|Jiangsu|
    |zj|Zhejiang|
    |ah|Anhui|
    |fj|Fujian|
    |jx|Jiangxi|
    |sd|Shandong|
    |ha|Henan|
    |hb|Hubei|
    |hn|Hunan|
    |gd|Guangdong|
    |gx|Guangxi|
    |hi|Hainan|
    |sc|Sichuan|
    |gz|Guizhou|
    |yn|Yunnan|
    |xz|Tibet|
    |sn|Shaanxi|
    |gs|Gansu|
    |qh|Qinghai|
    |nx|Ningxia|
    |xj|Xinjiang|
    |tw|Taiwan|
    |hk|Hong Kong|
    |mo|Macau|
    
    Special attention: distinguish between **Shaanxi (sn)** and **Shanxi (sx)**.

- Custom entries:
    - coop

---

## Features

### ✔ PSL‑like suffix recognition capability

A complete TLD classification system, cleaner and more controllable than the Public Suffix List:

| Category | Description |
|---|---|
| `original` | Original generic TLDs (.com .net .org etc.) |
| `generic` | New generic TLDs (.app .dev .shop etc.) |
| `brand` | Brand TLDs (.google .apple etc.) |
| `country` | Country‑code TLDs (.cn .uk .jp etc.) |
| `sponsored` | Sponsored TLDs (.edu .gov .mil etc.) |
| `infrastructure` | Infrastructure TLD (.arpa) |
| `pseudo-sld` | Pseudo second‑level domain constructors (global) |
| `pseudo-sld:cn` | Pseudo second‑level domain constructors (China‑specific) |

---

### ✔ Accurate root‑domain inference

Using structured splitting strategies:

- `domain.2` — extract two‑label structures (`example.com`)
- `domain.3` — extract three‑label structures (`example.co.uk`)

Combined with exclusion/retention rules for:

- `country.country`
- `original.country`
- `sponsored.country`
- `pseudo-sld.country`
- `pseudo-sld:cn.cn`

---

### ✔ High performance (10M domains processed in seconds)

📊 Benchmark  
- Test environment: standard multi‑core CPU / NVMe SSD  
- Dataset: [Domcop Top 10 Million Domains](https://www.domcop.com/files/top/top10milliondomains.csv.zip)

| Dataset | Size | Time |
|---|---|---|
| Test domain dataset | 381 MB / 10M lines | ~17 seconds |
| Real business dataset | 1M lines | ~1.5–2 seconds |

Thanks to:

- `sort --parallel` multi‑core parallel sorting  
- `rg` high‑speed regex matching  
- I/O‑reduction structural optimization  
- Merged TLD classification to reduce intermediate steps  

---

### ✔ Clean output

When outputting root‑domains, automatically filters dirty data:

- Empty lines  
- `-domain.com` (leading hyphen)  
- `domain-.com` (label ending with hyphen)  
- `domain.-com` (TLD starting with hyphen)  
- `domain.com-` (trailing hyphen)  

---

## Pseudo second‑level domain constructors — “pseudo TLDs” used to build second‑level country domains

- pseudo-sld (global)
- pseudo-sld:cn (China‑specific)

The actual registrable entity is not under the TLD, but under this pseudo‑SLD.

**Examples:**

| pseudo-sld | Base | Description |
|---|---|---|
| `com.cn` | `com` | China commercial domain |
| `net.cn` | `net` | China network domain |
| `edu.cn` | `edu` | China education domain |
| `gov.cn` | `gov` | China government domain |
| `co.uk` | `co` | UK commercial domain |
| `net.jp` | `net` | Japan network domain |

| pseudo-sld:cn | Base | Description |
|---|---|---|
| `bj.cn` | `cn` | China (Beijing) second‑level domain |
| `hk.cn` | `cn` | China (Hong Kong) second‑level domain |

Thus, the root‑domain of `www.example.com.cn` is `example.com.cn`, not `com.cn`.

---

## 🧬 Dependencies

| Tool | Purpose |
|---|---|
| `bash` 5.0+ | Script runtime |
| `rg` (ripgrep) | High‑speed regex filtering |
| `sort` | Parallel sorting (`--parallel`) |
| `awk` | Field processing and lookup |

---

## 🛠️ Configuration

**Before using, modify the database file paths in the script**, default:

```bash
DATA="/etc/geo/data/RootZoneDatabase.csv"
# IANA official data https://data.iana.org/TLD/tlds-alpha-by-domain.txt
# Convert to lowercase before use:
# tr '[:upper:]' '[:lower:]' < tlds-alpha-by-domain.txt \
#   | sort --parallel="$(nproc)" -u -o RootZoneDatabase.txt
TLD="/etc/geo/data/RootZoneDatabase.txt"
````

If the data files are stored in another path, open `droot.sh` and modify the path variables at the top of the script.

---

## 🚀 Usage
> The script accepts two parameters: input file path (original domain list) and output file path (refined base root domain dataset).
 
```bash
droot /path/to/input_file /path/to/output_file
```
 
| Parameter | Description |
|---|---|
| `input_file` | Input file **absolute path**, one domain or URL per line |
| `output_file` | Output file **absolute path**, one extracted root domain per line |
 
**Example:**
 
```bash
droot /data/domains/raw.txt /data/domains/roots.txt
```
 
Input `/data/domains/raw.txt`:
 
```
static.cdn.blog.example.co.uk
foo.bar.com.cn
mail.pku.edu.cn
www.google.com
-bad.example.com
```
 
Output `/data/domains/roots.txt`:
 
```
example.co.uk
bar.com.cn
pku.edu.cn
google.com
example.com
```
 
> Note: Although `-bad.example.com` has a leading hyphen, the root domain is valid and does not count as dirty data, so it is allowed to be retained.
> Dirty data detection applies to the output, not the input.
> Only `-example.com` counts as dirty data.
 
---
 
## Inference Logic Explanation
 
```
Domain ends with .cn
      │
      ▼
pseudo-sld:cn query
      │
      │
      ├─ Hit ——→ Remove from domain.2
      │    │      label[-2] + ".cn"
      │    └─ ——→ Extract from domain.3
      │            label[-3] + label[-2] + ".cn"
      │
      └─ Miss ——→ Do not process domain.2 and domain.3 (patch does not intervene)
 
```
 
**Examples**
 
| Input | Main Pipeline Result | After `\.cn$` Patch |
|---|---|---|
| `mail.pku.edu.cn` | `pku.edu.cn` ✓ | `pku.edu.cn` ✓ (consistent) |
| `blog.example.bj.cn` | `bj.cn` ✗ | `example.bj.cn` ✓ |
| `www.moe.gov.cn` | `moe.gov.cn` ✓ | `moe.gov.cn` ✓ (not processed) |
 
> **Note**: `gov.cn` / `edu.cn` etc. are handled by pseudo-sld; pseudo-sld:cn does not process them.
 
---
 
### Complete Execution Order of the Two Patches
 
```
Input domain
    │
    ▼
Normalization: TLD validation
    │
    ▼
TLD.EX query → domain.2 / domain.3 (Patch 1: country.country correction integrated)
    │
    ▼
Patch 2: \.cn$ correction
    │
    ▼
Final root-domain output
```
 
