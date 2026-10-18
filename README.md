# Protein Design: IDR–core hotspots e desenho de binders

Repositório da análise computacional de contatos entre uma região intrinsecamente
desordenada (IDR) e o core proteico, seguida da priorização de resíduos candidatos
para o desenho de proteínas ligantes (*binders*). O projeto foi desenvolvido no
contexto de iniciação científica e reúne o notebook de análise, as tabelas
processadas e estruturas representativas em formato PDB.

> **Status:** análise computacional e geração de candidatos estruturais. Os
> resultados deste repositório ainda não representam validação experimental.

## Objetivo

Identificar resíduos do core que apresentam alterações relevantes na frequência de
contato com a IDR em diferentes variantes (`P36R`, `G38R`, `D40G` e `D40Y`).
Esses resíduos são usados como hotspots para orientar o desenho de binders por
modelagem estrutural e desenho de sequência.

## Fluxo de análise

1. Quantificação dos contatos IDR–core a partir das réplicas disponíveis.
2. Seleção e ranqueamento dos hotspots pela soma da variação absoluta da
   frequência de contato.
3. Exportação de uma estrutura representativa do core para cada variante.
4. Geração de backbones com **RFdiffusion3**, condicionados pelos hotspots.
5. Projeto de sequências com **ProteinMPNN**.
6. Predição e ranqueamento estrutural com **ColabFold/AlphaFold2-Multimer**.
7. Refinamento iterativo dos melhores candidatos.

O fluxo completo está documentado em
[`notebooks/binder_core.ipynb`](notebooks/binder_core.ipynb). As etapas de
geração de binders dependem de GPU, checkpoints e ferramentas externas; por isso,
os resultados intermediários do pipeline generativo não são versionados neste
repositório.

## Principais parâmetros da análise

| Parâmetro | Valor |
|---|---:|
| Região IDR | resíduos 1–198 |
| Região do core | resíduos 199–505 |
| Cutoff de contato | 0,800 nm |
| Frequência mínima | 0,010 |
| Réplicas analisadas | 1 e 2 |
| Hotspots por variante | 10 |

Os hotspots são ordenados pela soma da variação absoluta da frequência de
contato. As colunas `sum_gain_frequency` e `sum_loss_frequency` permitem
separar ganhos e perdas de contato na condição mutante.

## Resultados disponíveis

| Variante | Hotspot mais bem ranqueado | Frequência mutante máxima |
|---|---|---:|
| P36R | GLU317 | 0,0917 |
| G38R | LYS430 | 0,3988 |
| D40G | ASP468 | 0,9972 |
| D40Y | MET401 | 0,2510 |

Os valores completos estão em
[`data/processed/summary/idr_core_hotspots_summary.md`](data/processed/summary/idr_core_hotspots_summary.md),
com versões tabular e estruturada em CSV e JSON.

## Organização do repositório

```text
.
├── README.md
├── notebooks/
│   └── binder_core.ipynb
├── data/
│   └── processed/
│       ├── hotspots/          # hotspots do core por variante
│       ├── contact_pairs/     # pares de contato core–IDR
│       └── summary/           # consolidação em Markdown, CSV e JSON
└── structures/
    └── representative_cores/  # estruturas PDB usadas como referência
```

### Convenção dos nomes

- `p36r`, `g38r`, `d40g`, `d40y`: variante analisada;
- `core_hotspots.csv`: hotspots ranqueados no core;
- `idr_core_contact_pairs.csv`: pares de resíduos do core e da IDR;
- `representative_core.pdb`: estrutura representativa exportada para a
  variante.

## Como reproduzir

### Análise e desenho de binders

O notebook foi preparado para execução em ambiente Kaggle. Ele instala as
dependências e ferramentas necessárias, incluindo Biopython, RFdiffusion3,
ProteinMPNN e ColabFold. Para reproduzir:

1. Abra `notebooks/binder_core.ipynb` em um ambiente com Python 3 e GPU.
2. Disponibilize o dataset de entrada no caminho esperado pelo notebook:
   `/kaggle/input/datasets/calistu/idr-core-hotspots`.
3. Configure `VARIANT` na seção de configurações (`p36r`, `g38r`, `d40g` ou
   `d40y`).
4. Execute as células em ordem.

O notebook grava os resultados gerados em
`/kaggle/working/results/<variante>/`, organizados por rodada (`round_0`,
`round_1`, etc.) e por ferramenta (`rfd3`, `mpnn`, `af2`).

### Dependências principais

- Python 3
- `pandas` e `biopython`
- RFdiffusion3 via `rc-foundry`
- ProteinMPNN
- ColabFold/AlphaFold2
- checkpoint do RFdiffusion3 e GPU compatível

As versões específicas e os comandos de instalação usados no experimento estão
registrados na primeira célula do notebook. O tempo de execução e o consumo de
memória dependem do hardware e do número de designs configurado.

## Interpretação e limitações

- Um hotspot indica alteração de contato na análise de dinâmica molecular; não
  implica, isoladamente, aumento de afinidade.
- As estruturas PDB deste repositório são estruturas representativas do core,
  não modelos finais validados dos binders.
- O ranking estrutural depende dos parâmetros de RFdiffusion3, ProteinMPNN e
  AlphaFold2 utilizados no notebook.
- A confirmação das hipóteses requer análises adicionais e validação
  experimental.

## Reprodutibilidade e rastreabilidade

As tabelas processadas e os PDBs versionados permitem verificar os hotspots sem
reexecutar a etapa generativa. O notebook preserva os parâmetros do pipeline e
os caminhos de saída por rodada, facilitando a auditoria dos candidatos gerados.

