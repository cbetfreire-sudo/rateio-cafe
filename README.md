# ☕ Rateio do Café — Extrato da Equipe

Site estático de prestação de contas do café compartilhado do escritório.
Sem backend, sem banco de dados: tudo é lido de arquivos `.json` na pasta `data/`.

## 📁 Estrutura de pastas

```
rateio-cafe/
├── index.html          ← página única (HTML + CSS + JS, usa Tailwind/Alpine/Chart.js via CDN)
├── README.md            ← este guia
└── data/
    ├── index.json        ← lista de meses disponíveis (alimenta o seletor de mês)
    ├── 2026-08.json      ← dados do mês corrente
    └── 2026-07.json      ← dados de um mês anterior (histórico)
```

Cada mês novo = **um arquivo `.json` novo** dentro de `data/`, mais uma linha em `data/index.json`.
Nada de editar HTML/JS todo mês — só os dados.

---

## 🧾 Como funciona o `data/index.json`

Ele é o "índice" que o site lê primeiro, para saber quais meses existem e montar o seletor:

```json
{
  "meses": [
    { "ref": "2026-08", "label": "Agosto/2026", "arquivo": "data/2026-08.json" },
    { "ref": "2026-07", "label": "Julho/2026",   "arquivo": "data/2026-07.json" }
  ]
}
```

- **O primeiro item da lista é o mês exibido por padrão** ao abrir o site — sempre coloque o mês mais recente no topo.
- `ref`: identificador único (recomendo `AAAA-MM`).
- `label`: o que aparece no seletor.
- `arquivo`: caminho para o JSON daquele mês.

## 🧾 Como funciona o `data/AAAA-MM.json` (dados de um mês)

```json
{
  "mes_referencia": "2026-08",
  "label": "Agosto/2026",
  "status": "aberto",            // "aberto" ou "fechado"
  "responsavel_caixa": "Marina Alves",
  "saldo_anterior": 42.30,       // saldo que sobrou do mês anterior

  "contribuintes": [
    {
      "nome": "Marina Alves",
      "cota_sugerida": 20.00,     // valor recomendado por pessoa
      "pago": true,
      "valor_pago": 20.00,        // pode ser diferente da cota (alguém dá uma "ajuda extra")
      "data_pagamento": "2026-08-03"
    },
    {
      "nome": "Carlos Nunes",
      "cota_sugerida": 20.00,
      "pago": false,
      "valor_pago": 0,
      "data_pagamento": null
    }
  ],

  "gastos": [
    {
      "data": "2026-08-02",
      "categoria": "Café/Açúcar",   // uma das 4: "Café/Açúcar", "Leite/Biscoitos", "Utensílios/Descartáveis", "Outros"
      "item": "Café torrado e moído 1kg (x2)",
      "local": "Supermercado Bretas",
      "quantidade": 2,
      "valor_total": 42.80,
      "responsavel": "Marina Alves"
    }
  ]
}
```

O site calcula tudo sozinho a partir daí:
- **Total arrecadado** = soma de `valor_pago` de quem tem `pago: true`
- **Total gasto** = soma de `valor_total` de todos os gastos
- **Saldo do mês** = arrecadado − gasto
- **Saldo acumulado** = `saldo_anterior` + saldo do mês

---

## 🔄 Guia: atualizando os dados no mês seguinte

1. **Copie** o arquivo do mês atual, por exemplo `data/2026-08.json`, e renomeie para o mês novo: `data/2026-09.json`.
2. Dentro dele, atualize:
   - `mes_referencia` e `label` para o novo mês.
   - `saldo_anterior` → copie o **saldo acumulado** que o site mostrou no mês anterior.
   - `status` → deixe `"aberto"` até fechar as contas do mês; mude para `"fechado"` no fim.
   - `contribuintes` → zere os `pago` para `false` e `valor_pago` para `0` (ninguém pagou ainda o mês novo), a menos que já tenha algum adiantamento.
   - `gastos` → apague os gastos antigos e vá adicionando as compras conforme acontecem.
3. Abra `data/index.json` e **adicione uma nova entrada no topo da lista** apontando pro arquivo novo:
   ```json
   { "ref": "2026-09", "label": "Setembro/2026", "arquivo": "data/2026-09.json" }
   ```
4. Ao longo do mês, sempre que alguém pagar a cota ou uma compra for feita, edite o `.json` do mês, salve e suba (`commit` + `push`) — o site atualiza sozinho.

Não é necessário mexer em `index.html` em nenhum momento desse fluxo mensal.

---

## 🖥️ Testando localmente antes de publicar

Como o site usa `fetch()` para ler os `.json`, **abrir o `index.html` direto com duplo-clique não funciona** (o navegador bloqueia leitura de arquivos locais por segurança/CORS). Rode um servidor local simples na pasta do projeto:

```bash
# Python 3 (já vem instalado na maioria dos sistemas)
python3 -m http.server 8000
```

Depois acesse **http://localhost:8000** no navegador.

Alternativas: extensão "Live Server" do VS Code, ou `npx serve`.

---

## 🚀 Publicando no GitHub Pages (grátis)

1. Crie um repositório novo no GitHub (ex: `rateio-cafe`).
2. Suba os arquivos mantendo a estrutura de pastas acima:
   ```bash
   git init
   git add .
   git commit -m "Site do rateio do café"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/rateio-cafe.git
   git push -u origin main
   ```
3. No GitHub, vá em **Settings → Pages**.
4. Em **Build and deployment → Source**, selecione **"Deploy from a branch"**.
5. Em **Branch**, escolha `main` e a pasta `/ (root)` → clique **Save**.
6. Aguarde 1–2 minutos. O GitHub mostrará o link final, algo como:
   ```
   https://SEU-USUARIO.github.io/rateio-cafe/
   ```
7. **Compartilhe esse link com a equipe.** Toda vez que alguém der `push` com um `.json` atualizado, o site publicado se atualiza automaticamente em 1–2 minutos.

### Dica de manutenção contínua
Se quiser deixar ainda mais simples para quem não mexe com git, edite os arquivos `.json` direto pela interface web do GitHub (abra o arquivo no repositório → ícone de lápis ✏️ → edite → "Commit changes"). Não precisa de terminal nem de clonar nada.
