# Porsche Sales Intelligence

Dashboard executivo de vendas Porsche construído em um único arquivo HTML, com gráficos desenhados em **HTML5 Canvas** e dados sanitizados a partir de uma base Excel.

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=111)
![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=flat-square&logo=python&logoColor=white)
![openpyxl](https://img.shields.io/badge/openpyxl-3.1.5-1D6F42?style=flat-square)
![Sem dependências no front-end](https://img.shields.io/badge/front--end-zero%20depend%C3%AAncias-d5001c?style=flat-square)

---

## Sobre o projeto

Este projeto demonstra um fluxo ponta a ponta de dados de vendas:

1. **Saneamento** da base bruta (`porsche_database.xlsx`) com regras estritas definidas em [`schema.md.txt`](schema.md.txt).
2. **Validação** e geração da base tratada (`porsche_database_sanitized.xlsx`) pelo script [`tools/sanitize_porsche.py`](tools/sanitize_porsche.py).
3. **Visualização** analítica em um dashboard interativo, autocontido, em [`porsche_dashboard.html`](porsche_dashboard.html).

O painel foi desenhado para transmitir a estética da Porsche: fundo grafite profundo, tipografia clara e espaçada, detalhes minimalistas e o vermelho oficial `#d5001c` como único acento.

> A base contém **100 registros de vendas** em USD. O dashboard abre diretamente no navegador, sem servidor, build ou bibliotecas externas.

---

## Funcionalidades

### Filtros globais

Os quatro filtros atualizam simultaneamente KPIs, tabela, cards de negócio e todos os gráficos:

- **Modelo** — 40 modelos Porsche distintos.
- **Ano-modelo** — 2020 a 2026.
- **Cidade** — 79 cidades.
- **Pagamento** — Cartão, transferência, financiamento, leasing, cripto etc.

Há também **busca textual** na tabela e botão **Limpar filtros**.

### KPIs

- Receita total (com o valor exato no selo do card).
- Unidades vendidas, modelos e UFs atendidos.
- Preço médio.
- Modelo líder em unidades (com desempate por receita).
- Ano-modelo líder.
- Taxa de entrega.

### Perguntas de negócio respondidas no painel

1. **Quais os principais modelos vendidos por cidade?**
   Card com as 6 principais praças, o modelo líder de cada uma e sua participação nas vendas locais.

2. **Qual ano-modelo mais saiu em um período?**
   Seletor de período (todo o histórico ou ano de venda) com o ano-modelo campeão, volume e distribuição dos demais anos.

3. **Quais carros são populares em cada cidade?**
   Deep dive por cidade com os 3 modelos mais vendidos, participação, preço médio, milhagem média e taxa de entrega. O seletor do card é sincronizado com o filtro global de cidade.

### Visualizações em Canvas

- Receita por modelo (top 10).
- Receita por trimestre (somente datas válidas).
- Métodos de pagamento (donut).
- Status de entrega.
- Vendas por estado (top 10 UFs).
- Distribuição por ano-modelo.

Todos os gráficos são renderizados com a **Canvas API nativa**, com tooltips interativos, suporte a alta densidade de pixels (`devicePixelRatio`) e redesenho responsivo — sem Chart.js, D3 ou qualquer CDN.

### Base de dados lateral

Tabela completa com as 100 vendas, rolagem própria, cabeçalho fixo, chips semânticos de status e status/tradução em português. Datas inválidas aparecem destacadas como `INVALID`.

---

## Estrutura do repositório

```text
.
├── .opencode/
│   └── agent/
│       └── porsche-sanitizer.md      # Configuração do agente de saneamento
├── tools/
│   └── sanitize_porsche.py           # Pipeline de higienização e validação
├── index.html                        # Página inicial do GitHub Pages (redireciona ao dashboard)
├── porsche_dashboard.html            # Dashboard interativo (single-file)
├── porsche_database.xlsx             # Base bruta de vendas
├── porsche_database_sanitized.xlsx   # Base tratada, com colunas normalizadas
├── schema.md.txt                     # Contrato de dados e regras de negócio
└── README.md
```

---

## Como executar

### Dashboard

Não há instalação. Basta abrir o arquivo no navegador:

```text
porsche_dashboard.html
```

Ou servir localmente:

```bash
python -m http.server 8000
```

Acesse `http://localhost:8000/porsche_dashboard.html`.

### Publicação no GitHub Pages

1. Envie o repositório para o GitHub.
2. Em **Settings → Pages**, selecione **Deploy from a branch**.
3. Escolha a branch `main` e a pasta `/ (root)`.
4. Acesse `https://<usuario>.github.io/<repositorio>/`.

O arquivo [`index.html`](index.html) é a página inicial e redireciona automaticamente para o dashboard. O arquivo `.nojekyll` desativa o processamento do Jekyll, garantindo que os arquivos sejam servidos como estão.

O dashboard também pode ser acessado diretamente em `https://<usuario>.github.io/<repositorio>/porsche_dashboard.html`.

---

## Pipeline de saneamento

### Requisitos

- Python 3.10+ (testado com 3.13)
- `openpyxl`

```bash
python -m pip install openpyxl
```

### Executar

```bash
python tools/sanitize_porsche.py \
  --input porsche_database.xlsx \
  --schema schema.md.txt \
  --output porsche_database_sanitized.xlsx \
  --strict
```

O modo `--strict` interrompe a execução em caso de divergência de valores esperados ou falha nas verificações de qualidade.

### Principais regras aplicadas

| Coluna de origem | Tratamento | Coluna gerada |
| :--- | :--- | :--- |
| `sale_date` | Normalização para ISO `YYYY-MM-DD`; datas impossíveis viram `INVALID` | `SaleDateSanitized` |
| `porsche_model` | Title case canônico, preservando trims (GTS, GT3 RS, Turbo S, Dakar etc.) | `PorscheModelSanitized` |
| `model_year` | Ano de 4 dígitos entre 1990 e 2035; texto e formatos quebrados são convertidos | `ModelYearSanitized` |
| `sale_price` | Remoção de símbolos/texto; conversão de `k` e valores por extenso para decimal | `SalesPriceSanitized` |
| `vehicle_mileage` | Milhas inteiras; conversão de KM com `1 km = 0.621371 mi`; "new" vira 0 | `VehicleMileageSanitized` |
| `payment_method` | Vocabulário controlado de meios de pagamento | `PayMethodSanitized` |
| `city` | Title case com pontuação preservada | `CitySanitized` |
| `state` | Código USPS de duas letras ou `INVALID` | `StateSanitized` |
| `delivery_status` | Status logísticos padronizados, incluindo correção de `DELIVERD` | `DeliveryStatusSanitized` |

O arquivo final preserva todas as colunas originais e insere cada coluna sanitizada imediatamente após sua origem. Nenhuma célula sanitizada fica vazia: valores não normalizáveis recebem `INVALID`.

### Observações sobre os dados

- **24 registros** possuem data de venda inválida. Eles continuam contando em KPIs e análises não temporais, mas são excluídos da linha do tempo trimestral — o dashboard informa essa quantidade nos cards de análise temporal.
- A base é fictícia e tem finalidade educacional.
- Os dados do dashboard estão embutidos no HTML (constante `DATA`). Ao regerar a planilha sanitizada, é necessário atualizar esse JSON no arquivo HTML.

---

## Stack

- **HTML5 + CSS3 + JavaScript puro** no front-end.
- **Canvas 2D** para os gráficos.
- **Python + openpyxl** no saneamento e na validação do Excel.
- Sem frameworks, sem bundler, sem requisições de rede.

---

## Contexto

Projeto desenvolvido durante a formação **Excel e Inteligência Artificial** da [DIO](https://www.dio.me/), combinando um agente especializado em saneamento de dados com um dashboard executivo gerado e refinado com apoio de IA.

---

## Aviso

Porsche, 911, 718, Cayenne, Macan, Panamera, Taycan e demais denominações são marcas registradas da Dr. Ing. h.c. F. Porsche AG. Este é um projeto educacional, sem vínculo oficial com a Porsche.
