# Financeiro Família Sousa

App de gestão financeira e tarefas da família. Front-end estático (`index.html`) + banco Supabase + um lembrete diário por e-mail via função serverless no Vercel.

## Estrutura do projeto

```
index.html            -> o app (React + Recharts + ícones via CDN, sem build)
styles.css             -> utilitários de layout (substitui o Tailwind CDN)
api/notificar.js       -> função de servidor que envia o e-mail diário de tarefas
vercel.json             -> aponta as rotas e agenda o cron do e-mail
schema_supabase.sql     -> cria categorias/despesas/salários/extras no banco
schema_tarefas.sql      -> cria a tabela de tarefas no banco
```

Suba **todos** esses arquivos e pastas para a raiz do repositório no GitHub (mantendo `api/notificar.js` dentro da pasta `api`).

## 1. Banco de dados (Supabase)

No **SQL Editor** do seu projeto Supabase, rode nessa ordem:
1. `schema_supabase.sql` (se ainda não rodou)
2. `schema_tarefas.sql`

A URL do projeto e a chave pública já estão embutidas no `index.html` e no `api/notificar.js` — são chaves seguras para expor (a proteção real são as políticas de RLS).

## 2. Publicar no GitHub + Vercel

```bash
git init
git add .
git commit -m "Financeiro Família Sousa"
git branch -M main
git remote add origin <URL_DO_SEU_REPOSITORIO>
git push -u origin main
```

No Vercel: **Add New → Project** → importe o repositório → Framework Preset **Other** → Build Command e Output Directory em branco → **Deploy**.

## 3. E-mail diário de tarefas

O lembrete é enviado por uma função em `api/notificar.js`, chamada automaticamente uma vez por dia pelo **Vercel Cron** (configurado em `vercel.json` para as 10h UTC = 7h em Brasília). Ela:
- Lista as tarefas com **data marcada para hoje**;
- Lista as tarefas com **prazo em até 5 dias**;
- Lista as **atrasadas**;
- Não envia nada se não houver pendências no dia.

Isso depende de 3 variáveis de ambiente que você precisa cadastrar em **Vercel → seu projeto → Settings → Environment Variables**:

| Nome | Valor | Onde conseguir |
|---|---|---|
| `CRON_SECRET` | uma string aleatória (ex.: gerada em 1password.com/password-generator) | você mesmo inventa |
| `RESEND_API_KEY` | chave da API do [resend.com](https://resend.com) | crie uma conta grátis → **API Keys → Create API Key** |
| `NOTIFY_EMAIL` | o e-mail que vai receber os lembretes | **precisa ser o mesmo e-mail usado para criar a conta no Resend** (veja abaixo) |

⚠️ **Importante sobre o Resend**: sem verificar um domínio próprio, contas novas só conseguem enviar e-mail para o **próprio e-mail de cadastro** (é uma proteção deles contra spam). Ou seja: crie a conta no Resend usando o e-mail da família que vai receber os avisos, e use esse mesmo e-mail em `NOTIFY_EMAIL`. Se um dia quiser mandar para vários e-mails da família, dá pra verificar um domínio próprio no Resend e evoluir isso.

Depois de cadastrar as 3 variáveis, vá em **Deployments → ⋯ → Redeploy** para elas passarem a valer.

### Testar sem esperar o horário do cron

Visite (substituindo pela sua URL e pela sua `CRON_SECRET`):

```
https://SEU-PROJETO.vercel.app/api/notificar?chave=SUA_CRON_SECRET
```

Deve devolver um JSON dizendo se enviou o e-mail ou não (e por quê).

## Observações

- Precisa de internet para funcionar (busca e grava tudo no Supabase). A primeira carga também baixa React/Recharts/ícones da CDN.
- O horário do cron está em UTC. `0 10 * * *` = 7h em Brasília (Fortaleza não tem horário de verão, então esse horário não muda ao longo do ano).
- No plano gratuito (Hobby) do Vercel, o cron pode disparar em qualquer minuto dentro da hora marcada — ou seja, o e-mail pode chegar entre 7h e 8h, não cravado às 7h00.
- Se quiser restringir o acesso ao app (hoje qualquer um com o link acessa), o caminho é adicionar Supabase Auth com uma tela de login simples para a família.
