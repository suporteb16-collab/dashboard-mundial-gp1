# Galvanizador Pro 1.0 — Dashboard

Dashboard de performance em tempo real para o lançamento **Galvanizador Pro 1.0**, integrando dados do **Meta Ads** e **Kiwify** via Google Sheets.

Desenvolvido por [B16](https://b16.com.br).

---

## Visão Geral

O dashboard exibe automaticamente:

- **Valor Investido** — gasto total no Meta Ads no período
- **Total de Vendas** — transações aprovadas na Kiwify
- **Faturamento** — receita total das vendas aprovadas
- **CAC** — Custo de Aquisição de Cliente (Investido ÷ Vendas)
- **Melhor CAC por Criativo** — cruzamento entre Ad Name (Meta) e utm_content (Kiwify)
- **Funil de Conversão** — Impressões → Cliques → Page View → Conversas Iniciadas → Vendas
- **Evolução Diária** — Investimento, Faturamento, CAC, Vendas e Conversas por dia
- **Distribuição por Origem** — canais de tráfego (utm_source)
- **Performance por Criativo** — investimento vs. vendas vs. CAC por criativo
- **Conjuntos e Campanhas** — investimento por adset e campanha

---

## Estrutura

```
galvanizador-dashboard/
├── index.html       # Dashboard completo (HTML + CSS + JS em um arquivo)
└── README.md        # Este arquivo
```

---

## Fontes de Dados

| Fonte | Aba na Planilha | Atualização |
|-------|----------------|-------------|
| Meta Ads | `meta_ads` | A cada hora (automático) |
| Kiwify | `kiwify` | Via webhook em tempo real |

**Planilha Google Sheets:** acesso via Service Account GCP, lida pelo Cloudflare Worker.

---

## Arquitetura

```
Meta Ads (automático)    ──▶  Google Sheets (meta_ads)  ──┐
                                                           ├──▶  Cloudflare Worker (GET)  ──▶  Dashboard (index.html)
Kiwify (webhook POST)    ──▶  Google Sheets (kiwify)    ──┘
```

1. **Google Sheets** centraliza os dados brutos
2. **Cloudflare Worker** autentica via JWT (Service Account GCP) e serve os dados como CSV
3. **Dashboard** busca os CSVs a cada hora e renderiza tudo no browser

---

## Configuração do Worker

O Worker precisa das seguintes variáveis de ambiente no Cloudflare:

| Variável | Descrição |
|----------|-----------|
| `GOOGLE_CLIENT_EMAIL` | E-mail da Service Account GCP |
| `GOOGLE_PRIVATE_KEY` | Chave privada RSA da Service Account (com `\n` literal) |
| `SHEET_ID` | ID da planilha Google Sheets |
| `WEBHOOK_SECRET` | Senha para validar webhooks da Kiwify |

### Endpoint GET (leitura para o dashboard)
```
GET https://SEU_WORKER.workers.dev?sheet=meta_ads
GET https://SEU_WORKER.workers.dev?sheet=kiwify
```
Retorna o conteúdo da aba como CSV.

### Endpoint POST (webhook Kiwify)
```
POST https://SEU_WORKER.workers.dev?secret=SUA_SENHA
```
Recebe o payload da Kiwify e grava a transação na aba `kiwify`.
Processa apenas pedidos com `order_status = PAID` ou `APPROVED`.

---

## Configuração do Dashboard

No arquivo `index.html`, ajuste as constantes no início do `<script>`:

```js
const WORKER_URL    = 'https://SEU_WORKER.workers.dev'; // URL do Cloudflare Worker
const SHEET_META    = 'meta_ads';                        // Nome da aba Meta Ads
const SHEET_TAMB    = 'kiwify';                          // Nome da aba Kiwify
const REFRESH_MS    = 60 * 60 * 1000;                   // Atualização: 1 hora
const DEFAULT_START = '2026-06-23';                      // Data início padrão
```

---

## Deploy via GitHub Pages

1. Suba o `index.html` na raiz do repositório
2. Vá em **Settings → Pages**
3. Source: `Deploy from a branch` → `main` → `/ (root)`
4. O dashboard ficará disponível em: `https://SEU_USUARIO.github.io/galvanizador-dashboard`

---

## Filtros

- **Período** — seletor de data no header (default: últimos 6 dias até hoje)
- **Produto** — o dashboard filtra automaticamente por `Product_product_name = 'Galvanizado Pro -1.0'`

---

## Legenda de CAC

| Cor | Faixa | Interpretação |
|-----|-------|---------------|
| 🟢 Verde | abaixo de R$ 100 | Positivo |
| 🟡 Amarelo | R$ 100 – R$ 300 | Atenção |
| 🔴 Vermelho | acima de R$ 300 | Alerta |

---

## Tecnologias

- **HTML / CSS / JavaScript** puro — sem framework, sem dependências de build
- **Chart.js 4.4** — gráficos
- **Google Fonts** — DM Sans + Bebas Neue
- **Cloudflare Workers** — backend serverless
- **Google Sheets API v4** — banco de dados
