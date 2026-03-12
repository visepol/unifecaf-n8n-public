# Como Importar o Workflow de Triagem de Currículos

Este guia explica como importar e configurar o workflow no n8n.

---

## Pré-requisitos

- n8n rodando (local via Docker ou em servidor)
- Conta Google com Gmail e Google Sheets ativados
- Chave de API do Google Gemini (gratuita em [https://aistudio.google.com](https://aistudio.google.com))

---

## 1. Importar o Workflow

1. Acesse o n8n em `http://localhost:5678` (ou o endereço do seu servidor)
2. No menu lateral, clique em **Overview**
3. Clique em **Create workflow** → **Import from file**
4. Selecione o arquivo `workflow-triagem-curriculos.json`
5. O workflow será importado com todos os nós configurados

---

## 2. Configurar as Credenciais

O workflow utiliza três credenciais que precisam ser configuradas:

### Gmail OAuth2
1. Vá em **Settings → Credentials → New**
2. Selecione **Gmail OAuth2**
3. Adicione seu Client ID e Client Secret do Google Cloud Console
4. A URL de callback é: `http://SEU-HOST:5678/rest/oauth2-credential/callback`
5. Autorize o acesso à sua conta Google

### Google Gemini (PaLM API)
1. Vá em **Settings → Credentials → New**
2. Selecione **Google PaLM API**
3. Cole sua chave de API obtida em [https://aistudio.google.com](https://aistudio.google.com)

### Google Sheets OAuth2
1. Vá em **Settings → Credentials → New**
2. Selecione **Google Sheets OAuth2**
3. Utilize as mesmas credenciais OAuth2 do Gmail (mesmo Client ID/Secret)
4. Autorize o acesso ao Google Sheets

---

## 3. Configurar a Planilha

1. Crie duas planilhas no Google Sheets:
   - **Triagem - Aprovados**: com colunas `Data`, `Nome Candidato`, `Email`, `Status`, `Justificativa`
   - **Triagem - Rejeitados**: com as mesmas colunas

2. Nos nós **Registrar Sheets (Aprovado)** e **Registrar Sheets (Rejeitado)**, atualize o **Spreadsheet ID** com o ID das suas planilhas (visível na URL do Google Sheets)

---

## 4. Ativar o Workflow

1. Clique em **Publish** no canto superior direito
2. O Gmail Trigger começará a verificar novos e-mails a cada minuto

---

## Como Testar

Para testar o sistema ao vivo, envie um e-mail para:

**vinicius.perpetuo.247755@a.fecaf.com.br**

Com um currículo em PDF anexado. O workflow responderá automaticamente em até 1 minuto.

Este repositório inclui dois PDFs de exemplo prontos para uso:

| Arquivo | Resultado esperado |
|---|---|
| `curriculo-aprovado.pdf` | Candidato aprovado: atende todos os requisitos |
| `curriculo-rejeitado.pdf` | Candidato rejeitado: não atende os requisitos |

O workflow irá:
1. Detectar o novo e-mail
2. Verificar se há anexo PDF
3. Extrair o texto do currículo
4. Analisar com o Google Gemini
5. Enviar resposta automática ao candidato
6. Registrar o resultado na planilha correspondente

---
