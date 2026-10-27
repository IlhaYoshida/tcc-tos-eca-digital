# Dados do TCC: análise automatizada de Termos de Uso e Políticas de Privacidade de jogos mobile em relação ao ECA Digital

Material de apoio do trabalho **"Análise Automatizada dos Termos de Uso e Políticas de Privacidade de Jogos Mobile em Relação ao ECA Digital"**, de William Haruo Yoshida, com orientação de Tafarel Rocha da Silva e Thiago Adriano Coleti (Ciência da Computação, UENP, Bandeirantes-PR).

O trabalho compara 86 *cases* do ToS;DR, selecionados por sua relação com o ECA Digital (Lei 15.211/2025), com os documentos de 20 jogos mobile, usando um *bi-encoder* e um *cross-encoder*, e atribui a cada jogo uma nota de A a E pela fórmula do ToS;DR. Estes arquivos permitem conferir qual *case* está ligado a qual artigo e recalcular as notas.

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `cases.csv` | Os 86 *cases*: título original (inglês) e tradução, classificação no ToS;DR, artigo(s) do ECA Digital, categoria temática, nível da ligação e justificativa. |
| `resultados_por_jogo.csv` | Resultado dos 20 jogos: nota, balanço, contagem de *cases* por classificação, idioma dos documentos e documentos coletados. |
| `ocorrencias_por_case_e_jogo.csv` | Para cada par (jogo, *case*), as duas pontuações do modelo e se o *case* foi considerado encontrado (1) ou não (0). Tem 1.720 linhas (20 jogos × 86 *cases*). |

Os arquivos são CSV em UTF-8, separados por vírgula.

## Colunas

**`cases.csv`**

- `n`: número do *case* (mesmo número do trabalho).
- `case_original_en`: título do *case* no ToS;DR (idioma original, inglês).
- `case_pt`: tradução automática para o português usada na comparação com documentos em português.
- `classificacao_tosdr`: `bom`, `ruim`, `bloqueador` ou `neutro`, segundo o ToS;DR.
- `artigos_eca_digital`: artigo(s) do ECA Digital associado(s) ao *case* pelo autor (ex.: `Art. 17, §4, VI`).
- `categoria`: uma das 11 categorias temáticas do trabalho.
- `nivel_da_ligacao`: ver abaixo.
- `justificativa_do_nivel`: por que a ligação recebeu esse nível.

**`resultados_por_jogo.csv`**: `jogo`, `classificacao_indicativa` (Google Play), `nota` (A a E), `balanco`, `bons`, `ruins`, `bloqueadores`, `neutros`, `total_encontrados`, `idioma_dos_documentos` (`pt` ou `en`) e `documentos_coletados`.

**`ocorrencias_por_case_e_jogo.csv`**: `jogo`, `case_n`, `encontrado`, `similaridade_biencoder`, `pontuacao_crossencoder` e `documento_de_origem` (o documento em que está o parágrafo escolhido pelo *cross-encoder* para aquele *case*).

## Nível da ligação entre o *case* e o artigo

O nível indica o quanto o *case* trata do que o artigo regula:

- **direta**: o *case* descreve uma prática que o artigo regula, ou o seu oposto;
- **parcial**: o *case* está ligado a mais de um artigo, e só um deles tem ligação direta;
- **indireta**: o tema é próximo, mas o artigo regula outro agente ou mecanismo, ou o *case* é genérico;
- **fraca**: a ligação se dá apenas por palavras em comum.

A associação entre *cases* e artigos e esses níveis são **avaliação do autor, sem validação jurídica**.

## Como a nota é calculada

Um *case* é contado como encontrado em um jogo se a similaridade do *bi-encoder* é maior ou igual a 0,57 **e** a pontuação do *cross-encoder* é maior ou igual a 0,9 (*cases* ruins e bloqueadores) ou a −0,5 (*cases* bons e neutros).

O balanço é `bons − ruins − 3 × bloqueadores`. A nota segue a ordem: sem pontos, N/A; balanço ≤ −10 ou bloqueadores > bons, E; pelo menos 3 bloqueadores ou ruins > bons, D; balanço < 5, C; qualquer ponto ruim, B; caso contrário, A.

Para recalcular as notas a partir dos arquivos (só biblioteca padrão do Python), com os limiares que quiser testar:

```python
import csv
from collections import defaultdict

BI, VETO_RUIM, VETO_BOM = 0.57, 0.9, -0.5

classe = {int(r["n"]): r["classificacao_tosdr"]
          for r in csv.DictReader(open("cases.csv", encoding="utf-8"))}

cont = defaultdict(lambda: {"bom": 0, "ruim": 0, "bloqueador": 0, "neutro": 0})
for r in csv.DictReader(open("ocorrencias_por_case_e_jogo.csv", encoding="utf-8")):
    c = classe[int(r["case_n"])]
    limiar = VETO_RUIM if c in ("ruim", "bloqueador") else VETO_BOM
    if float(r["similaridade_biencoder"]) >= BI and float(r["pontuacao_crossencoder"]) >= limiar:
        cont[r["jogo"]][c] += 1

def nota(b, r, k):
    if b + r + k == 0:
        return "N/A"
    balanco = b - r - 3 * k
    if balanco <= -10 or k > b:
        return "E"
    if k >= 3 or r > b:
        return "D"
    if balanco < 5:
        return "C"
    return "B" if r > 0 else "A"

for jogo, c in sorted(cont.items()):
    print(jogo, c["bom"] - c["ruim"] - 3 * c["bloqueador"], nota(c["bom"], c["ruim"], c["bloqueador"]))
```

## Origem dos dados e o que não está aqui

- Os títulos e as classificações dos *cases* são do ToS;DR (https://tosdr.org). Os dados do ToS;DR têm seus próprios termos de uso; consulte-os antes de reutilizar a lista.
- O texto do ECA Digital usado está em https://www.planalto.gov.br/ccivil_03/_ato2023-2026/2025/lei/l15211.htm.
- As traduções dos *cases* são automáticas (`deep-translator`) e podem conter imprecisões.
- Os Termos de Uso e as Políticas de Privacidade dos jogos são de terceiros e **não** estão neste repositório, nem os trechos extraídos deles.