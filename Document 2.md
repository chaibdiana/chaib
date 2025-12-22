# **comparar\_scores\_validacoes.py**
from **future** import annotations

import argparse import re from pathlib import Path from typing import Optional, Dict, Any, List

import pandas as pd
# **----------------------------**
# **Helpers de parsing (arquivo -> metadados)**
# **----------------------------**
def \_normalize\_spaces(s: str) -> str: return re.sub(r"\s+", " ", s).strip()

def extract\_year\_from\_path(path: Path) -> Optional[int]: # pega 2024/2025 do nome da pasta (resultados\_prompts\_2024 etc.) m = re.search(r"(19|20)\d{2}", str(path)) return int(m.group(0)) if m else None

def extract\_mrm\_codes(filename: str) -> List[str]: # captura MRMxx\_yyy e variações; pode ter mais de um no mesmo arquivo # exemplos: MRM24\_957, MRM25\_90, MRM2084 (sem underscore) codes: List[str] = []

\# padrões com underscore (MRM24\_957)\
codes += re.findall(r"MRM\d{2,4}\_\d{1,4}", filename, flags=re.IGNORECASE)\
\
\# padrão sem underscore (MRM2084)\
solo = re.findall(r"\bMRM\d{3,4}\b", filename, flags=re.IGNORECASE)\
for c in solo:\
`    `if not any(c in x for x in codes):\
`        `codes.append(c)\
\
\# normaliza para maiúsculo e remove duplicados mantendo ordem\
codes = [c.upper() for c in codes]\
seen = set()\
out = []\
for c in codes:\
`    `if c not in seen:\
`        `out.append(c)\
`        `seen.add(c)\
return out\


def infer\_parametro\_modelo(filename: str) -> str: s = filename.lower()

if "lgd" in s:\
`    `return "LGD"\
if "fcc" in s or "ccf" in s:\
`    `return "FCC"\
if re.search(r"\bpd\b", s) or "pd\_" in s or " - pd " in s:\
`    `return "PD"\
if "ead" in s:\
`    `return "EAD"\
if "behaviour" in s or "behavior" in s:\
`    `return "BEHAVIOUR"\
if re.search(r"\brr\b", s) or "rrgeneralista" in s:\
`    `return "RR"\
\
return "OUTRO"\


def infer\_segmento(filename: str) -> str: s = filename.lower()

if "varejo" in s:\
`    `return "Varejo"\
if "agro" in s:\
`    `return "Agro"\
if "internacional" in s:\
`    `return "Internacional"\
if "grandes empresas" in s or re.search(r"\bge\b", s) or "carteira\_ge" in s:\
`    `return "Grandes Empresas"\
if "middle market" in s:\
`    `return "Middle Market"\
if re.search(r"\bmiddle\b", s):\
`    `return "Middle"\
if "infra" in s or "energia" in s:\
`    `return "Infra/Energia"\
\
return "Nao\_identificado"\


def infer\_modelo\_nome(filename\_stem: str) -> str: """ Extrai a parte "humana" do nome: - remove prefixos tipo "resultados\_" - se tiver " - " pega o que vem depois - senão usa o que sobrou limpando underscores """ s = filename\_stem s = re.sub(r"^resultados\_?", "", s, flags=re.IGNORECASE)

if " - " in s:\
`    `right = s.split(" - ", 1)[1]\
`    `return \_normalize\_spaces(right.replace("\_", " "))\
\
s = re.sub(r"(MRM\d{2,4}(\_\d{1,4})?)+", "", s, flags=re.IGNORECASE)\
s = s.replace("\_\_", "\_").strip("\_- ")\
s = \_normalize\_spaces(s.replace("\_", " "))\
return s if s else filename\_stem\


def to\_binary\_score(x: Any) -> Optional[int]: """ Converte resposta\_binaria para 0/1 quando possível. Aceita: 1/0, True/False, "sim"/"não", "ok"/"nok", etc. """ if x is None or (isinstance(x, float) and pd.isna(x)): return None

if isinstance(x, bool):\
`    `return int(x)\
\
if isinstance(x, (int,)) and not pd.isna(x):\
`    `return int(x)\
\
s = str(x).strip().lower()\
\
positives = {"1", "true", "t", "sim", "s", "yes", "y", "ok", "conforme", "atende"}\
negatives = {"0", "false", "f", "nao", "não", "n", "no", "nok", "nao atende", "não atende"}\
\
if s in positives:\
`    `return 1\
if s in negatives:\
`    `return 0\
\
m = re.match(r"^\s\*([01])\s\*$", s)\
if m:\
`    `return int(m.group(1))\
\
return None\


def is\_essencial(x: Any) -> Optional[bool]: if x is None or (isinstance(x, float) and pd.isna(x)): return None if isinstance(x, bool): return x if isinstance(x, (int, float)) and not pd.isna(x): try: return bool(int(x)) except Exception: return None s = str(x).strip().lower() if s in {"1", "true", "sim", "s", "yes", "y"}: return True if s in {"0", "false", "nao", "não", "n", "no"}: return False return None

def build\_metadata(file\_path: Path) -> Dict[str, Any]: year = extract\_year\_from\_path(file\_path.parent) or extract\_year\_from\_path(file\_path) stem = file\_path.stem fname = file\_path.name

mrm\_codes = extract\_mrm\_codes(fname)\
segmento = infer\_segmento(fname)\
modelo\_nome = infer\_modelo\_nome(stem)\
\
return {\
`    `"ano\_validacao": year,\
`    `"arquivo": fname,\
`    `"arquivo\_stem": stem,\
`    `"mrm\_codes": ",".join(mrm\_codes) if mrm\_codes else None,\
`    `"segmento": segmento,\
`    `"modelo\_nome": modelo\_nome,\
}\

# **----------------------------**
# **Leitura e empilhamento**
# **----------------------------**
EXPECTED\_COLS = [ "ID", "ID\_Teste\_Original", "Parâmetro", "Essencial", "Fase", "Descrição", "Tipo", "Artigos\_BCB\_303", "Item\_EBA", "CRE36\_Basel", "EBA\_CCF", "pergunta", "resposta\_completa", "resposta\_binaria", "justificativa", "observacoes", "modelo\_utilizado", ]

def \_clean\_text\_series(s: pd.Series) -> pd.Series: s = s.astype("string").str.strip() s = s.replace({"": pd.NA, "nan": pd.NA, "None": pd.NA, "none": pd.NA, "NaN": pd.NA}) return s

def \_normalize\_parametro\_value(x: Any) -> Optional[str]: if x is None or (isinstance(x, float) and pd.isna(x)): return None s = str(x).strip().upper() if s in {"", "NAN", "NONE"}: return None

\# normaliza variações comuns\
\# (ajuste se aparecer algo específico nos seus arquivos)\
s = s.replace("CCF", "FCC")  # se algum vier como CCF, normaliza para FCC\
\
\# remove espaços duplicados\
s = \_normalize\_spaces(s)\
return s\


def read\_one\_excel(path: Path) -> pd.DataFrame: df = pd.read\_excel(path)

\# garante colunas esperadas (se faltar alguma, cria vazia)\
for c in EXPECTED\_COLS:\
`    `if c not in df.columns:\
`        `df[c] = pd.NA\
\
\# metadados do arquivo\
meta = build\_metadata(path)\
for k, v in meta.items():\
`    `df[k] = v\
\
\# normalizações úteis\
df["score\_item"] = df["resposta\_binaria"].apply(to\_binary\_score)\
df["essencial\_bool"] = df["Essencial"].apply(is\_essencial)\
\
\# --- parâmetro (PRIORIDADE: coluna da base "Parâmetro")\
parametro\_base = df["Parâmetro"].apply(\_normalize\_parametro\_value)\
\
\# fallback: se não tiver na base, tenta inferir do nome do arquivo\
fallback = infer\_parametro\_modelo(path.name)\
\
df["parametro"] = pd.Series(parametro\_base, index=df.index).fillna(fallback)\
\
\# opcional: limpa string final\
df["parametro"] = \_clean\_text\_series(df["parametro"]).str.upper()\
\
return df\


def find\_excels(root: Path) -> List[Path]: # busca xlsx e xlsm dentro das pastas resultados\_prompts\_2024 e 2025 targets: List[Path] = [] for folder in ["resultados\_prompts\_2024", "resultados\_prompts\_2025"]: p = root / folder if p.exists(): targets += list(p.rglob("*.xlsx")) targets += list(p.rglob("*.xlsm")) return sorted(set(targets))
# **----------------------------**
# **Agregações e comparação**
# **----------------------------**
GROUP\_KEYS = [ "ano\_validacao", "arquivo", "mrm\_codes", "parametro", # <-- agora vem da coluna Parâmetro (com fallback) "segmento", "modelo\_nome", "modelo\_utilizado", ]

def compute\_scores(df: pd.DataFrame, only\_essenciais: bool) -> pd.DataFrame: d = df.copy() d = d[~d["score\_item"].isna()].copy()

if only\_essenciais:\
`    `d = d[d["essencial\_bool"] == True].copy()\
\
out = (\
`    `d.groupby(GROUP\_KEYS, dropna=False)\
.agg(\
`        `n\_itens=("score\_item", "size"),\
`        `n\_ok=("score\_item", "sum"),\
`        `score=("score\_item", "mean"),\
`    `)\
.reset\_index()\
)\
\
out["score\_pct"] = out["score"] \* 100.0\
return out\


def compare\_years(scores: pd.DataFrame) -> pd.DataFrame: idx = [k for k in GROUP\_KEYS if k != "ano\_validacao"] pivot = scores.pivot\_table(index=idx, columns="ano\_validacao", values="score\_pct", aggfunc="mean") pivot = pivot.reset\_index()

year\_cols = [c for c in pivot.columns if isinstance(c, (int, float))]\
for c in year\_cols:\
`    `pivot.rename(columns={c: f"score\_pct\_{int(c)}"}, inplace=True)\
\
cols = pivot.columns\
if "score\_pct\_2024" in cols and "score\_pct\_2025" in cols:\
`    `pivot["delta\_2025\_vs\_2024"] = pivot["score\_pct\_2025"] - pivot["score\_pct\_2024"]\
else:\
`    `pivot["delta\_2025\_vs\_2024"] = pd.NA\
\
return pivot\


def main() -> None: parser = argparse.ArgumentParser( description="Compara scores 2024 vs 2025 empilhando resultados\_prompts\_\*." ) parser.add\_argument( "--root", type=str, default=".", help="Pasta raiz onde está a pasta AGENTE GAMBA (ou ela mesma).", ) parser.add\_argument( "--out", type=str, default="comparacao\_scores\_validacoes.xlsx", help="Arquivo Excel de saída.", ) args = parser.parse\_args()

root = Path(args.root).resolve()\
\
\# se apontar para pai e existir AGENTE GAMBA dentro, entra nela\
if not (root / "resultados\_prompts\_2024").exists() and (root / "AGENTE GAMBA").exists():\
`    `root = (root / "AGENTE GAMBA").resolve()\
\
files = find\_excels(root)\
if not files:\
`    `raise SystemExit(\
`        `f"Nenhum Excel encontrado em {root}. "\
`        `"Verifique se existem as pastas resultados\_prompts\_2024 e resultados\_prompts\_2025."\
`    `)\
\
dfs: List[pd.DataFrame] = []\
errors = 0\
for f in files:\
`    `try:\
`        `dfs.append(read\_one\_excel(f))\
`    `except Exception as e:\
`        `errors += 1\
`        `print(f"[ERRO] Falha ao ler {f.name}: {e}")\
\
if not dfs:\
`    `raise SystemExit("Nenhum arquivo pôde ser lido com sucesso.")\
\
base = pd.concat(dfs, ignore\_index=True)\
\
\# scores\
scores\_total = compute\_scores(base, only\_essenciais=False)\
scores\_ess = compute\_scores(base, only\_essenciais=True)\
\
\# comparação 2025 vs 2024\
comp\_total = compare\_years(scores\_total)\
comp\_ess = compare\_years(scores\_ess)\
\
out\_path = Path(args.out).resolve()\
with pd.ExcelWriter(out\_path, engine="openpyxl") as writer:\
`    `base.to\_excel(writer, index=False, sheet\_name="base\_empilhada")\
`    `scores\_total.to\_excel(writer, index=False, sheet\_name="scores\_total")\
`    `scores\_ess.to\_excel(writer, index=False, sheet\_name="scores\_essenciais")\
`    `comp\_total.to\_excel(writer, index=False, sheet\_name="comparacao\_total")\
`    `comp\_ess.to\_excel(writer, index=False, sheet\_name="comparacao\_essenciais")\
\
print(f"\nOK! Arquivo gerado em: {out\_path}")\
if errors:\
`    `print(f"Atenção: {errors} arquivo(s) falharam na leitura (veja logs acima).")\


if **name** == "**main**": main()







cd "/c/Users/SEU\_USUARIO/Área de Trabalho/AGENTE GAMBA"

python -m venv .venv

source .venv/Scripts/activate

python -m pip install --upgrade pip

pip install pandas openpyxl

python comparar\_scores\_validacoes.py --root "." --out "comparacao\_scores\_validacoes.xlsx"

