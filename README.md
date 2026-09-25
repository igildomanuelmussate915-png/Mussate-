# WhatsApp Bot

Bot de atendimento automático para WhatsApp usando a WhatsApp Cloud API.

## 1. Estrutura do projeto

```text
whatsapp-bot/
├── .env.example
├── .gitignore
├── package.json
├── README.md
├── server.js
└── src/
    ├── app.js
    ├── controllers/
    │   └── webhookController.js
    ├── routes/
    │   └── webhookRoutes.js
    └── services/
        ├── botService.js
        └── whatsappService.js
```

## 2\. Requisitos

- Node.js
- Conta Meta for Developers
- WhatsApp Business Platform
- Número configurado para a WhatsApp Cloud API
- Servidor público com HTTPS

## 3\. Instalação

Execute:

``` bash
npm install
```

## 4\. Configuração

Crie um arquivo chamado:

``` text
.env
```

Use o `.env.example` como modelo:

``` text
PORT=3000

VERIFY_TOKEN=seu_token_de_verificacao

WHATSAPP_TOKEN=seu_token_da_meta

PHONE_NUMBER_ID=seu_phone_number_id

GRAPH_API_VERSION=v23.0
```

Nunca publique o arquivo `.env` no GitHub.

## 5\. Executar o bot

``` bash
npm start
```

O servidor deverá apresentar:

``` text
Bot WhatsApp iniciado na porta 3000
```

## 6\. Testar o servidor

Abra:

``` text
http://localhost:3000/
```

A resposta esperada é:

``` json
{
  "status": "online",
  "message": "Bot WhatsApp funcionando"
}
```

## 7\. Webhook

O webhook está disponível em:

``` text
/webhook
```

Depois de publicar o servidor, a URL será semelhante a:

``` text
https://seu-dominio.com/webhook
```

Essa URL será configurada na aplicação da Meta.

### 7.1. Requisitos da URL

A URL do webhook deve:

- Ser pública e acessível pela internet.
- Usar HTTPS com um certificado TLS válido e confiável.
- Não usar `localhost`, `127.0.0.1` ou endereços privados.
- Responder às requisições `GET` de verificação da Meta.
- Aceitar requisições `POST` com as notificações recebidas.
- Permanecer disponível para evitar perda de eventos.
- Não exigir autenticação adicional na rota de verificação, pois a Meta precisa acessá-la diretamente.

Para testes locais, publique o servidor com uma ferramenta de túnel HTTPS, como ngrok ou Cloudflare Tunnel. Em produção, use um domínio próprio ou um serviço de hospedagem com HTTPS configurado.

### 7.2. Criar e configurar o webhook no Meta for Developers

1. Acesse [https://developers.facebook.com/](https://developers.facebook.com/) e entre na conta que administra a aplicação da Meta.

2. No painel, abra a aplicação vinculada à WhatsApp Cloud API. Se ainda não existir uma aplicação:
   - Clique em **My Apps**.
   - Selecione **Create App**.
   - Escolha o tipo de aplicação adequado, normalmente **Business**.
   - Informe o nome da aplicação e conclua a criação.
   - Adicione o produto **WhatsApp** à aplicação.

3. No menu lateral da aplicação, acesse **WhatsApp > Configuration**. Em algumas versões do painel, essa área pode aparecer como **WhatsApp > API Setup** ou dentro da configuração do produto WhatsApp.

4. Localize a seção **Webhook** e clique em **Edit**, **Configure** ou **Set up webhooks**.

5. No campo **Callback URL**, informe a URL pública completa do endpoint, incluindo o caminho `/webhook`:

   ``` text
   https://seu-dominio.com/webhook
   ```

   Não inclua espaços, barras adicionais ou o protocolo `http://`.

6. No campo **Verify token**, informe exatamente o mesmo valor definido na variável `VERIFY_TOKEN` do arquivo `.env`:

   ``` text
   VERIFY_TOKEN=seu_token_de_verificacao
   ```

   Esse token é criado por você. Ele não precisa ser igual ao token de acesso da Meta. Use um valor longo, aleatório e difícil de adivinhar, por exemplo:

   ``` text
   VERIFY_TOKEN=whatsapp_webhook_8f3a2c9d_token
   ```

7. Salve ou confirme a configuração. A Meta enviará uma requisição `GET` para a URL informada contendo parâmetros semelhantes a:

   ``` text
   hub.mode=subscribe
   hub.verify_token=seu_token_de_verificacao
   hub.challenge=valor_enviado_pela_meta
   ```

   O servidor deverá:
   - Verificar se `hub.mode` é `subscribe`.
   - Comparar `hub.verify_token` com `VERIFY_TOKEN`.
   - Responder com o valor de `hub.challenge` quando o token estiver correto.
   - Retornar erro, como `403`, quando o token for inválido.

   Se a verificação for concluída, o painel exibirá a confirmação de que o webhook foi conectado.

8. Depois da validação, na mesma área de configuração, localize a seção **Webhook fields**, **Subscribe to fields** ou **Manage webhook fields**.

9. Inscreva a aplicação no evento:

   ``` text
   messages
   ```

   Esse é o evento necessário para receber mensagens recebidas, alterações de status e outras notificações relacionadas às conversas do WhatsApp. Dependendo da versão do painel, outros campos podem aparecer, mas `messages` é o campo essencial para este bot.

10. Verifique se o número de telefone está associado à aplicação e à conta do WhatsApp Business. Em **WhatsApp > API Setup**, confirme o `Phone number ID` e use esse valor na variável:

    ``` text
    PHONE_NUMBER_ID=seu_phone_number_id
    ```

11. Confirme também que o token usado pelo servidor possui as permissões necessárias para operar a WhatsApp Cloud API, especialmente:
    - `whatsapp_business_messaging`
    - `whatsapp_business_management`, quando exigida para administrar recursos da conta

12. Reinicie o servidor após alterar o `.env`:

    ``` bash
    npm start
    ```

### 7.3. Testar a verificação do webhook

Antes de configurar o painel, confirme que o endpoint está acessível por HTTPS. Um teste de verificação pode ser feito com:

``` bash
curl "https://seu-dominio.com/webhook?hub.mode=subscribe&hub.verify_token=seu_token_de_verificacao&hub.challenge=123456"
```

Quando o token estiver correto, a resposta esperada será:

``` text
123456
```

Se a resposta for `403`, verifique se o valor enviado em `hub.verify_token` é exatamente igual ao valor de `VERIFY_TOKEN`. Se houver erro de conexão ou certificado, confirme se o domínio está público e se o HTTPS possui um certificado válido.

### 7.4. Testar o recebimento de mensagens

Depois de salvar a URL e assinar o evento `messages`:

1. Envie uma mensagem, como `oi`, para o número configurado na WhatsApp Cloud API.
2. Verifique os logs do servidor.
3. Confirme se o endpoint recebeu uma requisição `POST`.
4. Confirme se o bot processou a mensagem e enviou a resposta esperada.

A Meta pode enviar eventos de teste ou notificações com estruturas diferentes. O servidor deve responder rapidamente com um status HTTP `200` após receber uma notificação válida. O processamento mais demorado deve ser feito de forma assíncrona para evitar novas tentativas de entrega.

## 8\. Menu do bot

Quando o cliente enviar:

``` text
oi
```

ou:

``` text
menu
```

o bot responderá:

``` text
👋 Olá! Seja bem-vindo ao nosso WhatsApp.

Escolha uma opção:

1️⃣ Informações
2️⃣ Produtos
3️⃣ Atendimento
4️⃣ Contactos

Envie o número da opção desejada.
```

## 9\. Opções

### 1 - Informações

Apresenta informações sobre o negócio ou projeto.

### 2 - Produtos

Apresenta produtos ou serviços.

### 3 - Atendimento

Permite encaminhar o cliente para atendimento.

### 4 - Contactos

Apresenta os contactos definidos no bot.

## 10\. Segurança

Nunca coloque no GitHub:

- WHATSAPP\_TOKEN
- senhas
- chaves privadas
- credenciais da Meta
- arquivo `.env`

Use variáveis de ambiente.

## 11\. Próximas funcionalidades

O projeto poderá receber:

- Botões interativos
- Catálogo de produtos
- Sistema de pedidos
- Atendimento humano
- Banco de dados
- Cadastro de clientes
- Histórico de pedidos
- Painel administrativo
- Respostas automáticas
- Integração com pagamentos
- Sistema de autenticação

## 12\. Licença

Este projeto destina-se ao uso autorizado pelo proprietário da conta WhatsApp Business.

```

Esse conteúdo pode ser colocado diretamente em **`README.md`** no GitHub.
