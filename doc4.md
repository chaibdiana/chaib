from \_\_future\_\_ import annotations

import argparse  
import re  
from pathlib import Path  
from typing import Optional, Any, List

import pandas as pd

\# \==========================================================  
\# Helpers gerais  
\# \==========================================================

def \_normalize\_spaces(s: str) \-\> str:  
    return re.sub(r"\\s+", " ", s).strip()

def extract\_year\_from\_path(path: Path) \-\> Optional\[int\]:  
    m \= re.search(r"(19|20)\\d{2}", str(path))  
    return int(m.group(0)) if m else None

def extract\_mrm\_codes(filename: str) \-\> List\[str\]:  
    codes: List\[str\] \= \[\]

    codes \+= re.findall(r"MRM\\d{2,4}\_\\d{1,4}", filename, flags=re.IGNORECASE)  
    solo \= re.findall(r"\\bMRM\\d{3,4}\\b", filename, flags=re.IGNORECASE)

    for c in solo:  
        if not any(c in x for x in codes):  
            codes.append(c)

    codes \= \[c.upper() for c in codes\]  
    seen, out \= set(), \[\]  
    for c in codes:  
        if c not in seen:  
            out.append(c)  
            seen.add(c)  
    return out

def infer\_segmento(filename: str) \-\> str:  
    s \= filename.lower()

    if "varejo" in s:  
        return "Varejo"  
    if "agro" in s:  
        return "Agro"  
    if "internacional" in s:  
        return "Internacional"  
    if "grandes empresas" in s or re.search(r"\\bge\\b", s):  
        return "Grandes Empresas"  
    if "middle market" in s:  
        return "Middle Market"  
    if re.search(r"\\bmiddle\\b", s):  
        return "Middle"  
    if "infra" in s or "energia" in s:  
        return "Infra/Energia"

    return "Nao\_identificado"

def infer\_modelo\_nome(filename\_stem: str) \-\> str:  
    s \= re.sub(r"^resultados\_?", "", filename\_stem, flags=re.IGNORECASE)

    if " \- " in s:  
        return \_normalize\_spaces(s.split(" \- ", 1)\[1\].replace("\_", " "))

    s \= re.sub(r"(MRM\\d{2,4}(\_\\d{1,4})?)+", "", s, flags=re.IGNORECASE)  
    s \= \_normalize\_spaces(s.replace("\_", " ").strip("- "))  
    return s or filename\_stem

\# \==========================================================  
\# Conversões de respostas  
\# \==========================================================

def to\_binary\_score(x: Any) \-\> Optional\[int\]:  
    if x is None or (isinstance(x, float) and pd.isna(x)):  
        return None

    if isinstance(x, bool):  
        return int(x)

    if isinstance(x, int):  
        return x

    s \= str(x).strip().lower()

    if s in {"1", "true", "sim", "yes", "ok", "conforme", "atende"}:  
        return 1  
    if s in {"0", "false", "nao", "não", "nok", "nao atende"}:  
        return 0

    return None

def is\_essencial(x: Any) \-\> Optional\[bool\]:  
    if x is None or (isinstance(x, float) and pd.isna(x)):  
        return None

    if isinstance(x, bool):  
        return x

    s \= str(x).strip().lower()  
    if s in {"1", "true", "sim", "yes"}:  
        return True  
    if s in {"0", "false", "nao", "não"}:  
        return False

    return None

\# \==========================================================  
\# Tipo de modelo (regra de negócio CORRETA)  
\# \==========================================================

def infer\_tipo\_modelo(modelo\_nome: Any) \-\> Optional\[str\]:  
    if not isinstance(modelo\_nome, str):  
        return None

    s \= modelo\_nome.upper()

    if "LGD" in s:  
        return "LGD"  
    if "FCC" in s or "CCF" in s:  
        return "FCC"  
    if re.search(r"\\bPD\\b", s):  
        return "PD"  
    if "BEHAVIOUR" in s or "BEHAVIOR" in s or "RR" in s:  
        return "SCORING"

    return None

def manter\_linha\_parametro(row) \-\> bool:  
    p \= row\["parametro"\]  
    t \= row\["tipo\_modelo"\]

    if not isinstance(p, str):  
        return False

    p \= p.upper()

    if t \== "LGD":  
        return p \== "LGD"  
    if t \== "FCC":  
        return "FCC" in p  
    if t \== "PD":  
        return "PD" in p  
    if t \== "SCORING":  
        return p \== "SCORING"

    return False

\# \==========================================================  
\# Leitura dos arquivos  
\# \==========================================================

EXPECTED\_COLS \= \[  
    "ID", "ID\_Teste\_Original", "Parâmetro", "Essencial",  
    "Fase", "Descrição", "Tipo",  
    "Artigos\_BCB\_303", "Item\_EBA", "CRE36\_Basel", "EBA\_CCF",  
    "pergunta", "resposta\_completa", "resposta\_binaria",  
    "justificativa", "observacoes", "modelo\_utilizado",  
\]

def read\_one\_excel(path: Path) \-\> pd.DataFrame:  
    df \= pd.read\_excel(path)

    for c in EXPECTED\_COLS:  
        if c not in df.columns:  
            df\[c\] \= pd.NA

    meta \= {  
        "ano\_validacao": extract\_year\_from\_path(path.parent) or extract\_year\_from\_path(path),  
        "arquivo": path.name,  
        "arquivo\_stem": path.stem,  
        "mrm\_codes": ",".join(extract\_mrm\_codes(path.name)),  
        "segmento": infer\_segmento(path.name),  
        "modelo\_nome": infer\_modelo\_nome(path.stem),  
    }

    for k, v in meta.items():  
        df\[k\] \= v

    df\["score\_item"\] \= df\["resposta\_binaria"\].apply(to\_binary\_score)  
    df\["essencial\_bool"\] \= df\["Essencial"\].apply(is\_essencial)  
    df\["parametro"\] \= df\["Parâmetro"\].astype("string").str.upper()

    return df

def find\_excels(root: Path) \-\> List\[Path\]:  
    files \= \[\]  
    for folder in \["resultados\_prompts\_2024", "resultados\_prompts\_2025"\]:  
        p \= root / folder  
        if p.exists():  
            files \+= list(p.rglob("\*.xlsx"))  
            files \+= list(p.rglob("\*.xlsm"))  
    return sorted(set(files))

\# \==========================================================  
\# MAIN  
\# \==========================================================

def main() \-\> None:  
    parser \= argparse.ArgumentParser()  
    parser.add\_argument("--root", default=".")  
    parser.add\_argument("--out", default="comparacao\_scores\_validacoes.xlsx")  
    args \= parser.parse\_args()

    root \= Path(args.root).resolve()

    files \= find\_excels(root)  
    if not files:  
        raise SystemExit("Nenhum Excel encontrado.")

    base \= pd.concat(\[read\_one\_excel(f) for f in files\], ignore\_index=True)

    \# \-------------------------  
    \# BASE DINÂMICA  
    \# \-------------------------  
    base\["tipo\_modelo"\] \= base\["modelo\_nome"\].apply(infer\_tipo\_modelo)

    base\_dyn \= base\[  
        base\["tipo\_modelo"\].notna() &  
        base\["score\_item"\].notna()  
    \].copy()

    base\_dyn \= base\_dyn\[base\_dyn.apply(manter\_linha\_parametro, axis=1)\].copy()

    \# \-------------------------  
    \# TABELA DINÂMICA (LONGA – uma linha por ano)  
    \# \-------------------------  
    group\_cols \= \[  
        "ano\_validacao",  
        "modelo\_nome",  
        "segmento",  
        "tipo\_modelo",  
        "modelo\_utilizado",  
        "mrm\_codes",  
    \]

    tabela\_dinamica \= (  
        base\_dyn  
        .groupby(group\_cols, dropna=False)  
        .agg(  
            n\_itens=("score\_item", "size"),  
            n\_ok=("score\_item", "sum"),  
            score=("score\_item", "mean"),  
        )  
        .reset\_index()  
    )

    tabela\_dinamica\["score\_pct"\] \= tabela\_dinamica\["score"\] \* 100

    \# \-------------------------  
    \# OUTPUT  
    \# \-------------------------  
    out\_path \= Path(args.out).resolve()  
    with pd.ExcelWriter(out\_path, engine="openpyxl") as writer:  
        base.to\_excel(writer, index=False, sheet\_name="base\_empilhada")  
        tabela\_dinamica.to\_excel(writer, index=False, sheet\_name="tabela\_dinamica\_modelos")

    print(f"OK\! Arquivo gerado em: {out\_path}")

if \_\_name\_\_ \== "\_\_main\_\_":  
    main()

