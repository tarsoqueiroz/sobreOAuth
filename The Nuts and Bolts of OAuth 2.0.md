# The Nuts and Bolts of OAuth 2.0

> [(Udemy) The Nuts and Bolts of OAuth 2.0](https://www.udemy.com/course/oauth-2-simplified)

## Terminologia

**Regras:**

- **Resource owner**: o usuário dono dos dados (ex.: sua conta do Google).
- **User agent**: o dispositivo
- **Client**: a aplicação que quer acessar os dados (ex.: um app terceiro).
  - **Confidential clients (no servidor)**: tem as credenciais
  - **Public clients (browser web ou aplicativos móveis)**: não tem as credenciais
- **Resource server**: servidor que armazena os dados protegidos (ex.: API do Google Drive).
- **Authorization server**: servidor que valida o usuário e emite tokens (ex.: login do Google).

**Tokens:**

- **Authorization code**: `client_secret` armazenado por um back-end seguro.
- **Access token**: "chave" temporária que o Client usa para acessar o Resource Server.
- **Refresh token**: token usado para obter novos Access Tokens sem pedir permissão novamente ao usuário.

**Grant types (a.k.a. Grant flows / Authorization flows / Fluxos):**

- **Authorization Code** (mais comum)
  - Client redireciona o usuário para o AS (ex.: tela de login do Google).
  - Usuário autoriza e o AS envia um código de autorização para o Client.
  - Client troca esse código por um Access Token (requisição back-end).
    - Vantagem: Seguro (token nunca exposto no front-end).
- **PKCE** (Authorization Code + segurança extra)
  - Similar ao anterior, mas com um code challenge para evitar ataques em apps móveis/SPA.
  - Obrigatório para clientes públicos (como apps JavaScript).
- **Client Credentials** (máquina-a-maquina, service-to-service)
  - Usado para comunicação entre serviços (ex.: microserviços).
  - O Client se autentica diretamente no AS com seu client_id e client_secret.
- **Device Code** (browseless ou input constraint devices)
  - dispositivos sem teclado/tela (ex.: Smart TVs, IoT) ou onde a entrada de credenciais é inviável.
- **Refresh Token**
  - Permite obter novos Access Tokens após a expiração sem interação do usuário (usuário ter que efetuar novo login).
- **Implicit Flow** (depreciado, legacy)
  - Antigo fluxo para SPAs, agora substituído pelo PKCE.
  - Obtinha o Access Token diretamente sem o Authorization Code.
- **Password Grant** (depreciado, legacy)
  - Pede ao usuário a senha.
  - Aplicações legadas ou de alta confiança (ex.: app oficial do próprio provedor de serviço).
  - Não recomendado para clientes públicos (vazamento de senhas).

**Scopes e Permissões:**

- Scopes definem quais permissões o Client pede (ex.: read:contacts, write:files).
- O usuário deve consentir explicitamente com esses scopes durante a autorização.
- Define o conjunto de informações do cliente que serão expostas pelo token.

## Front Channel vs. Back Channel

Estes termos referem-se aos **canais de comunicação** usados durante os fluxos do OAuth 2.0, diferenciados por segurança e participantes:

| Característica   | Front Channel | Back Channel |
| :--------------- | :------------ | :----------- |
| **Local**            | Navegador do usuário (User Agent) | Comunicação direta entre servidores |
| **Segurança**        | Menos seguro (dados expostos)     | Mais seguro (dados criptografados)  |
| **Exemplo**          | Redirecionamentos HTTP            | Chamadas API server-to-server       |
| **Dados sensíveis?** | Nunca tokens completos            | Tokens, client_secret, credenciais  |

## Front Channel (Canal Frontal)

**O que é?**

- Comunicação que passa pelo **navegador do usuário** (User Agent).
- **Não é seguro** para dados sensíveis, pois URLs/parâmetros podem ser vazados (histórico do navegador, logs de rede).

**Quando ocorre?**

Durante a fase de autorização do OAuth:

- O Client (ex.: app web) redireciona o usuário para o Authorization Server (AS) via URL:

```sh
<https://auth-server.com/authorize>?
  response_type=code&
  client_id=123&
  redirect_uri=<https://client.com/callback&>
  scope=email
```

- O AS retorna o código de autorização (ou erro) para o `redirect_uri` via navegador:

```sh
https://client.com/callback?code=ABC123
```

**Riscos e Mitigações:**

- **Exposição de dados:** O código de autorização (`code`) pode ser interceptado.
  - **Solução:** Use PKCE (`code_challenge`) para vincular o código à requisição original.
- **Ataques CSRF:** O parâmetro `state` prevê falsificação de redirecionamento.

## Back Channel (Canal Traseiro)

**O que é?**

- Comunicação **direta entre servidores** (Client → Authorization Server ou Client → Resource Server).
- **Seguro:** Usa HTTPS com autenticação mútua (ex.: `client_secret`).

**Quando ocorre?**

Durante a **troca de tokens**:

- O Client envia o `code` (obtido no Front Channel) + `client_secret` para o AS:

```sh
POST /token HTTP/1.1
Host: auth-server.com
Body:
  grant_type=authorization_code&
  code=ABC123&
  client_id=123&
  client_secret=SEGREDO&
  redirect_uri=https://client.com/callback
```

- O AS responde com o `access_token` diretamente ao Client (sem passar pelo navegador).

**Por que é seguro?**

- O `client_secret` nunca é exposto no Front Channel.
- Tokens são transmitidos via HTTPS com autenticação.

## Exemplo Prático (Fluxo Authorization Code)

- Front Channel:
  - Usuário é redirecionado para o AS via navegador.
  - AS devolve o `code` para o `redirect_uri` (no navegador).
- Back Channel:
  - O Client (back-end) troca o `code` por um `access_token` diretamente com o AS.
  - O Client usa o `access_token` para chamar a API do Resource Server (sem navegador).

## Comparação em Cenários Reais

| Fluxo OAuth | Front Channel | Back Channel |
| :---------- | :------------ | :----------- |
| Authorization Code | Troca de `code` via navegador      | Troca `code` → `token` (server-side)   |
| Implicit           | Token retornado via URL (inseguro) | Não usa Back Channel                   |
| Client Credentials | Não usa Front Channel              | Autenticação direta (server-to-server) |

## Boas Práticas e Segurança

- **Use HTTPS** em todas as etapas.
- **Nunca envie ou armazene** tokens sensíveis (`access_token`, `refresh_token`, `client_secret`) no Front Channel (localStorage/sessionStorage).
- **Use PKCE** para fluxos com clientes públicos (SPAs/mobile).
- **Valide sempre** os scopes no Resource Server.
- **Valide sempre** `redirect_uri` no AS para evitar ataques de redirecionamento.
- **Short-lived Access Tokens + Refresh Tokens** são a melhor prática.

## OAuth Clients

Tarefa 1

## OAuth 2.0 para Server-Side Applications (Web Apps)

Fluxo Authorization Code, este é o fluxo mais seguro para aplicações web com back-end, onde o client_secret pode ser armazenado com segurança.

Segue um passo a passo detalhado:

### Personagens do Fluxo Authorization Code

- **User (Resource Owner)**: Dono dos dados (ex.: usuário do GitHub).
- **Web App (Client)**: Sua aplicação com back-end (ex.: app em Node.js/Django).
- **Authorization Server (AS)**: Serviço que gerencia autorização (ex.: GitHub, Google).
- **Resource Server (RS)**: API que guarda os dados (ex.: API do GitHub).

### Registro do Client para o Fluxo Authorization Code

Pré-requisito para começar, registre sua aplicação no AS (ex.: [GitHub OAuth Apps](https://github.com/settings/applications/new)) para obter:

- `client_id` → Identificador público.
- `client_secret` → Chave secreta (só back-end conhece).
- `redirect_uri` → URL de callback (ex.: `https://seusite.com/oauth/callback`).

### Fluxo de Autorização (Front Channel)

**Passo 1:** Iniciar o fluxo

Sua aplicação redireciona o usuário para o AS com um link contendo:

```bash
https://auth-server.com/authorize?
  response_type=code&
  client_id=SEU_CLIENT_ID&
  redirect_uri=https://seusite.com/callback&
  scope=user:email&
  state=ABC123xyz
```

Parâmetros Chave:

- `response_type=code`: Solicita um código de autorização.
- `state`: Token CSRF para prevenir ataques (gerado aleatoriamente).
- `scope`: Permissões solicitadas (ex.: acessar email do usuário).

**Passo 2:** Usuário autoriza

O AS exibe uma tela de login e pede consentimento ("Aplicação X quer acessar seus dados").

Se o usuário aprovar, o AS redireciona para o redirect_uri com um código temporário:

```bash
https://seusite.com/callback?
  code=XYZ789abc&
  state=ABC123xyz
```

### Troca do Código por Token (Back Channel)

**Passo 3:** Back-end solicita o Access Token

Seu servidor faz uma chamada **diretamente ao AS** (sem passar pelo navegador):

```bash
POST https://auth-server.com/token
Headers:
  Content-Type: application/x-www-form-urlencoded
Body:
  client_id=SEU_CLIENT_ID&
  client_secret=SEU_CLIENT_SECRET&
  code=XYZ789abc&
  grant_type=authorization_code&
  redirect_uri=https://seusite.com/callback
```

**Passo 4:** AS retorna os Tokens

Resposta do AS (`JSON`):

```json
{
  "access_token": "abc123xyz",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "def456uvw",
  "scope": "user:email"
}
```

### Acesso ao Resource Server (API)

**Passo 5:** Usar o Access Token

Seu back-end pode agora acessar os dados protegidos:

```bash
GET https://api.resource-server.com/user
Headers:
  Authorization: Bearer abc123xyz
```

### Diagrama

![Fluxo Authorization Code](./imagens/Fluxo-Authorization_Code.png)

### Por que este Fluxo é Seguro?

- O `client_secret` nunca é exposto no navegador (só no back-end).
- O `access_token` é obtido via Back Channel (HTTPS direto entre servidores).
- O código (`code`) é de uso único e curta duração (mesmo se interceptado, sem `client_secret` é inútil).

### Boas Práticas

- **Sempre valide** o `state` no callback para **evitar CSRF**.
- **Use HTTPS** em todas as etapas.
- **Armazene** `client_secret` **com segurança** (ex.: variáveis de ambiente, serviços como AWS Secrets Manager).
- **Para SPAs**, substitua `client_secret` por PKCE (mas ainda use Authorization Code).

### Tarefa 2

## OAuth 2.0 para Client-Side Applications (SPAs/Mobile)

Fluxo Authorization Code com PKCE tem como objetivo a segurança reforçada para clientes públicos (sem `client_secret`), como SPAs (React, Angular) ou apps móveis (Android/iOS).

Passo a Passo do Authorization Code + PKCE:

### Personagens do Fluxo Authorization Code com PKCE

- **User (Resource Owner)**: Dono dos dados.
- **Client-Side App (Client)**: Aplicação sem back-end (ex.: React app ou app mobile).
- **Authorization Server (AS)**: Serviço como GitHub, Google.
- **Resource Server (RS)**: API que guarda os dados (ex.: API do GitHub).

### Registro do Client para o Fluxo Authorization Code com PKCE

- Registre sua aplicação no AS (ex.: [GitHub OAuth Apps](https://github.com/settings/applications/new)) para obter:
  - `client_id` (público).
  - `redirect_uri` (ex.: `http://localhost:3000/callback` para desenvolvimento).
- Não há client_secret (cliente público não consegue guardar segredos).

### Fluxo com PKCE (Front Channel + Back Channel)

**Passo 1:** Gerar Code Verifier e Challenge

Seu app gera dois códigos antes de iniciar o fluxo:

- `code_verifier`: String aleatória (ex.: 43 caracteres).

```js
// Exemplo em JavaScript:
const codeVerifier = generateRandomString(43); // Usar crypto.getRandomValues()
```

- `code_challenge`: Hash do code_verifier (SHA-256 + Base64URL).

```js
    const codeChallenge = base64urlEncode(sha256(codeVerifier));
```

**Passo 2:** Iniciar autorização (Front Channel)

Redirecione o usuário para o AS com:

```bash
https://auth-server.com/authorize?
  response_type=code&
  client_id=SEU_CLIENT_ID&
  redirect_uri=https://seusite.com/callback&
  scope=user:email&
  state=ABC123xyz&
  code_challenge=CODIGO_GERADO&
  code_challenge_method=S256
```

PKCE adiciona:

- `code_challenge`: Hash do code_verifier.
- `code_challenge_method`: S256 (SHA-256).

**Passo 3:** Usuário autoriza

O AS mostra a tela de login e pede consentimento.

Se aprovado, redireciona para redirect_uri com:

```sh
https://seusite.com/callback?
  code=XYZ789abc&
  state=ABC123xyz
```

**Passo 4:** Trocar Code por Token (Back Channel)

Seu app envia o `code` + `code_verifier` (não o `challenge`) ao AS:

```sh
POST https://auth-server.com/token
Headers:
  Content-Type: application/x-www-form-urlencoded
Body:
  client_id=SEU_CLIENT_ID&
  code=XYZ789abc&
  grant_type=authorization_code&
  redirect_uri=https://seusite.com/callback&
  code_verifier=CODE_VERIFIER_ORIGINAL
```

**Passo 5:** AS valida e retorna Tokens

O AS recalculá o code_challenge a partir do code_verifier.

Se coincidir com o valor inicial, retorna:

```json
{
  "access_token": "abc123xyz",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "def456uvw"
}
```

### Acesso à API (Resource Server)

```sh
GET https://api.resource-server.com/user
Headers:
  Authorization: Bearer abc123xyz
```

### Diagrama do Fluxo PKCE

![Fluxo Authorization Code with PKCE](./imagens/Fluxo-Authorization_Code_with_PKCE.png)

### Por que PKCE é Necessário?

- **Evita Ataques de Interceptação:**
  - Sem PKCE, um invasor poderia roubar o `code` do callback e usá-lo.
  - Com PKCE, o invasor precisaria do `code_verifier` original (não acessível).
- **Substitui o `client_secret`:**
  - Clientes públicos (SPAs/mobile) não podem guardar segredos.
  - O `code_verifier` atua como uma "senha descartável".

### Comparação: Authorization Code vs. Authorization Code + PKCE

| Característica | Authorization Code (Back-end) | Authorization Code + PKCE (SPA/Mobile) |
| :------------- | :---------------------------: | :------------------------------------: |
| Cliente        | Aplicações com back-end       | SPAs, apps móveis                      |
| Segredo        | Usa `client_secret`           | Usa `code_verifier`                    |
| Segurança      | Alta (HTTPS + segredo)        | Alta (HTTPS + PKCE)                    |

### Boas Práticas com PKCE

- Sempre use `S256` (SHA-256) para o `code_challenge_method` (evite `plain`).
- Armazene o `code_verifier` temporariamente (ex.: `sessionStorage`).
- Valide `state` para prevenir CSRF (igual ao fluxo tradicional).

## OAuth 2.0 para Native Applications

Deep Linking

Fluxo do protocolo

Tarefa 3

## OAuth 2.0 para Sigle-Page Applications

Dynamic Backend Server

Fluxo do protocolo

Tarefa 4

## OAuth for the Internet of Things

Para dispositivos com dificuldade para se executar o processo de login (usuário e senha).

- [RFC8628](./recursos/rfc8628.txt.pdf)
- [OAuth 2.0 Device Flow Proxy Server](https://github.com/aaronpk/Device-Flow-Proxy-Server)
  - A demonstration of the OAuth 2.0 Device Code flow for devices without a browser or with limited keyboard entry.
  - This service acts as an OAuth server that implements the device code flow, proxying to a real OAuth server behind the scenes.

Fluxo do protocolo

## Client Credentials Flow

Fluxo do protocolo

Tarefa 5

## Introduction to OpenID Connect

Explicação sobre o que é o OpenID Connect.

Diferença de um OpenID Token de um Access Token.

Sobre o Hybrid OpenID Connect Flows

Validando e usando um Token ID

Tarefa 6

## Access token types and their tradeoffs

Types of Access tokens

Pros and Cons of:

- Reference Tokens
- Self-encoded Tokens

Tarefa 7

## JWT Access Token

Estrutura de um JWT Access Token

JSON Web Token Profile for OAuth 2.0 Access Token

Remote Token Introspection

- [RFC7662 - OAuth 2.0 Token Introspection](./recursos/rfc7662.txt.pdf)

Local Token verification (the fast way)

The best of both worlds: using an API gateway

## Choosing Token Lifetimes

Increasing security with short token lifetimes

Improving user experience with long token lifetimes

Contextually choosing token lifetimes

## Handling revoked or invalidated access token

Reasons why an access token may become invalidated

The problem with local validation

Token lifetime considerations

How apps can revoke access tokens

## OAuth Scopes

The purpose of OAuth scopes

Defining scopes for your API

Prompting the user for consent

## Futuro do OAuth

O futuro do OAuth: OAuth 2.1

- [RFC6749 OAuth Core](./recursos/rfc6749.txt.pdf)
  - Authorization Code +PKCE
  - Client Credentials
- [RFC6750 Bearer Tokens](./recursos/rfc6750.txt.pdf)
  - Tokens in HTTP Header
  - Tokens in POST Form Body

Recursos e leitura adicionais:

- [Advanced OAuth Security](https://www.udemy.com/course/advanced-oauth-security)
- [OAuth 2.0 Simplified](https://oauth2simplified.com/)
- [OAuth 2.0 Playground](https://www.oauth.com/playground/)
- [OAuth.net](https://oauth.net/)

## That's all

...folks!!!
