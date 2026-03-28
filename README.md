# Webinar Pesquisa + Mapa de Desenho Humano — Vida Autoral

Formulário de pesquisa de público para leads do webinar gratuito de Desenho Humano.
Coleta dados pessoais + pesquisa (profissão, objetivos, desafios) + dados de nascimento
para gerar o mapa de Desenho Humano via API Bodygraph.

## Fluxo do Usuário

```
Etapa 1: Nome, Email, WhatsApp
    ↓
Etapa 2: Pesquisa (3 perguntas de múltipla escolha)
  - Área de atuação
  - Motivação principal
  - Maior desafio
    ↓
Etapa 3: Dados de nascimento (data, hora, cidade)
    ↓
Loading animado → Mapa HD na tela + PDF por email
```

## Estrutura

```
webinar-pesquisa/
├── api/
│   └── submit.js          # Serverless function (Vercel)
├── public/
│   └── index.html         # Frontend (formulário + mapa)
├── fonts/
│   └── DejaVuSans.ttf     # Fonte para o PDF (copiar do projeto original)
├── vercel.json            # Config de rotas
├── package.json
├── .env.example
└── README.md
```

## Setup & Deploy

### 1. Clonar e configurar

```bash
git clone <repo-url>
cd webinar-pesquisa
npm install
```

### 2. Copiar a fonte DejaVuSans

Copie o arquivo `fonts/DejaVuSans.ttf` do projeto original do mapa (`vidaautoral.com.br/mapa`)
para a pasta `fonts/` deste projeto.

### 3. Configurar variáveis de ambiente

Copie `.env.example` para `.env.local` e preencha:

```bash
cp .env.example .env.local
```

| Variável | Descrição |
|---|---|
| `BODYGRAPH_API_KEY` | Chave da API bodygraphchart.com |
| `RESEND_API_KEY` | Chave da API Resend (email) |
| `FROM_EMAIL` | Email remetente (domínio verificado no Resend) |
| `FROM_NAME` | Nome do remetente (ex: "Vida Autoral") |
| `REPLY_TO` | Email de resposta |
| `GOOGLE_SHEETS_WEBHOOK` | URL do webhook para salvar pesquisa (opcional) |

### 4. Webhook da Pesquisa (Google Sheets)

O campo `GOOGLE_SHEETS_WEBHOOK` recebe uma URL de webhook (Google Apps Script / Make / Zapier)
que receberá um POST JSON com os dados da pesquisa:

```json
{
  "timestamp": "2026-03-28T15:30:00.000Z",
  "nome": "Maria Silva",
  "email": "maria@email.com",
  "telefone": "(86) 99999-9999",
  "profissao": "Empreendedora / dona de negócio",
  "objetivo": "Quero me conhecer melhor e entender meu propósito",
  "desafio": "Me sinto perdida, sem saber qual caminho seguir",
  "data_nascimento": "1991-04-05",
  "hora_nascimento": "14:30",
  "cidade_nascimento": "Teresina"
}
```

#### Opção A: Google Apps Script (gratuito)

1. Abra [Google Sheets](https://sheets.google.com) → crie uma planilha nova
2. Extensões → Apps Script → cole:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);
  sheet.appendRow([
    data.timestamp,
    data.nome,
    data.email,
    data.telefone,
    data.profissao,
    data.objetivo,
    data.desafio,
    data.data_nascimento,
    data.hora_nascimento,
    data.cidade_nascimento
  ]);
  return ContentService.createTextOutput('ok');
}
```

3. Deploy → Nova implantação → App da Web → Qualquer pessoa → Copiar URL
4. Cole a URL no `GOOGLE_SHEETS_WEBHOOK`

#### Opção B: Make/Zapier

Crie um cenário com trigger "Webhook" e conecte ao Google Sheets.

### 5. Deploy na Vercel

```bash
# Via CLI
vercel --prod

# Ou conectar o repositório Git na dashboard da Vercel
```

Adicione as variáveis de ambiente na dashboard da Vercel:
Settings → Environment Variables → adicione todas do `.env.example`

### 6. Domínio personalizado (opcional)

Na dashboard da Vercel, adicione um domínio customizado:
ex: `webinar.vidaautoral.com.br` ou `pesquisa.vidaautoral.com.br`

## Perguntas da Pesquisa

### 1. Área de atuação
- Empreendedora / dona de negócio
- CLT / empregada
- Autônoma / freelancer
- Em transição de carreira
- Outra (campo livre)

### 2. Motivação
- Quero me conhecer melhor e entender meu propósito
- Estou buscando uma virada financeira / nova fonte de renda
- Quero tomar decisões com mais clareza e menos ansiedade
- Tenho curiosidade sobre Desenho Humano

### 3. Maior desafio
- Me sinto perdida, sem saber qual caminho seguir
- Trabalho muito e ganho pouco — quero mais retorno
- Tenho medo de arriscar e sair da zona de conforto
- Quero alinhar minha vida pessoal e profissional

## Customização

- **Alterar perguntas:** edite as `<label class="radio-card">` na etapa 2 do `public/index.html`
- **Alterar CTA do upsell:** edite o link no `document.getElementById('upsell-btn')` no script
- **Alterar branding:** as cores estão nas CSS variables no `:root`
- **Adicionar tracking (Meta Pixel, GA4):** adicione scripts no `<head>` do HTML
