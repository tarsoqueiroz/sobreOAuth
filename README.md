# sobreOAuth

## Sobre sobreOAuth

Estudo e compilação sobre o protocolo OAuth.

> ***Alguns termos serão mantidos em inglês.***

**O que é OAuth 2.0?**

É um protocolo de autorização (não autenticação) que permite que aplicações terceiras acessem recursos de um usuário em um serviço (como Google, GitHub, etc.) sem compartilhar suas credenciais.

> ***Exemplo:*** Um app de fotos que acessa suas imagens do Google Drive sem saber sua senha.

**Diferença entre OAuth 2.0 e OpenID Connect (OIDC)?**

- OAuth 2.0: Focado em autorização (acessar recursos).
- OIDC: Camada sobre OAuth 2.0 que adiciona autenticação (saber quem é o usuário via id_token JWT).

## Referências

- [(Udemy) The Nuts and Bolts of OAuth 2.0](https://www.udemy.com/course/oauth-2-simplified)
- [(Udemy) Advanced OAuth Security](https://www.udemy.com/course/advanced-oauth-security/)
- [The Little Book of OAuth 2.0 RFCs](https://oauth.net/books/The%20Little%20Book%20of%20OAuth%202.0%20RFCs.pdf)
- [`oauth.net`](https://oauth.net/)
  - [OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified/)
  - [OAuth 2.0 Simplified](https://oauth2simplified.com/)
  - [OAuth 2.0 Servers](https://www.oauth.com/)
- [(Udemy) Master OAuth 2.0: A Practical Guide to API Security](https://www.udemy.com/course/master-oauth-2-api-security-practical-guide)
- [(Udemy) Web Authentication With Golang - Google's Go Language](https://www.udemy.com/course/oauth-authentication/)
- [RFC list](https://www.rfc-editor.org/search/rfc_search_detail.php?title=OAuth&page=All)
- OAuth 2.0 em português claro:
  - [Parte I](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-i-23c5618a4601)
  - [Parte II](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-ii-e8e0d8dba994)
  - [Parte III](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-iii-669a07807f9a)

Ferramentas para Testar:

- Postman: Suporte nativo a fluxos OAuth.
- oauth.tools: Site interativo para simular fluxos.
- Keycloak: Servidor de autorização open-source.

## Mapa das Especificações do OAuth 2.0

> `https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/`

The OAuth 2.0 Core Framework (RFC 6749) defines roles and a base level of functionality, but leaves a lot of implementation details unspecified. Since the publication of the RFC, the OAuth Working Group has published many additional specs built on top of this framework to fill in the missing pieces. Looking at the full list of specs the group is working on can be somewhat overwhelming. This chapter lays out how the various specs relate to each other.

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

### Front Channel (Canal Frontal)

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

### Back Channel (Canal Traseiro)

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

### Exemplo Prático (Fluxo Authorization Code)

- Front Channel:
  - Usuário é redirecionado para o AS via navegador.
  - AS devolve o `code` para o `redirect_uri` (no navegador).
- Back Channel:
  - O Client (back-end) troca o `code` por um `access_token` diretamente com o AS.
  - O Client usa o `access_token` para chamar a API do Resource Server (sem navegador).

### Comparação em Cenários Reais

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

## Segurança no OAuth 2.0

Revisão do fluxo Authorization Code

Front Channel vs Back Channel

Objetivos de segurança

Modelos de ataque

### FAPI 2.0

- [FAPI 2.0](https://oauth.net/fapi/)

FAPI 2.0 is an API security profile based on the OAuth 2.0 framework suitable for protecting APIs in high-value scenarios.

The core FAPI functionality is split into two documents, with a third that describes the attacker model:

- [FAPI 2.0 Security Profile (Final)](https://openid.net/specs/fapi-security-profile-2_0-final.html)
- [FAPI 2.0 Attacker Model (Final)](https://openid.net/specs/fapi-attacker-model-2_0-final.html)
- [FAPI 2.0 Message Signing (Draft)](https://openid.bitbucket.io/fapi/fapi-2_0-message-signing.html)

Protegendo o fluxo OAuth

## Segurança do Front Channel

### Ataques de injeção no Authorization Code

### Como o PKCE previne a injeção no Authorization Code

- [RFC 7636: Proof Key for Code Exchange](https://oauth.net/2/pkce/)

### Calculate a PKCE code verifier and code challenge

Implement a function that calculates the PKCE code challenge (hash) given a code verifier in the input string.

```php
<?php
function calculate_code_challenge($code_verifier) {
    // Verifica se o code_verifier não está vazio
    if (empty($code_verifier)) {
        throw new InvalidArgumentException('Code verifier cannot be empty');
    }
    
    // Calcula o hash SHA-256 do code_verifier
    $hash = hash('sha256', $code_verifier, true);
    
    // Codifica o hash em Base64 URL-safe (sem padding)
    $code_challenge = strtr(rtrim(base64_encode($hash), '='), '+/', '-_');
    
    return $code_challenge;
}
?>
```

Exemplo de uso:

```php
<?php
try {
    $code_verifier = 'dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk'; // Exemplo de code_verifier
    $challenge = calculate_code_challenge($code_verifier);
    
    echo "Code Verifier: " . $code_verifier . "\n";
    echo "Code Challenge: " . $challenge . "\n";
    
} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
?>
```

### História do PKCE

### Authorization Server Mixup sttacks and prevention

- [RFC 9207: OAuth 2.0 Authorization Server Issuer Identification](https://www.rfc-editor.org/rfc/rfc9207)
- [RFC 9126: OAuth 2.0 Pushed Authorization Requests](https://www.rfc-editor.org/rfc/rfc9126.html)

## Segurança do Back Channel

Client Authentication in OAuth

MTLS as Client Authentication

- [RFC 8705: OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens](https://www.rfc-editor.org/rfc/rfc8705)

Private Key JWT as Client Authentication

- [RFC 7523](https://www.rfc-editor.org/rfc/rfc7523)
- [RFC 7521](https://www.rfc-editor.org/rfc/rfc7521)

## Security Access Tokens

Requisitos adicionais para validação de Access Token

- [FAPI 2.0 Security Profile](https://openid.bitbucket.io/fapi/fapi-security-profile-2_0.html)

Problemas com Bearer Tokens

MTLS para Sender-Constrained Access Token

DPoP para Sender-Constrained Access Token

## Non-Reputiation

## OAuth School

Tarefas de [OAuth School](https://oauth.school/).

This website is a companion to the course The Nuts and Bolts of OAuth 2.0 and Hands-on Introduction to OAuth 2.0 by Aaron Parecki. You should enroll in the course if you'd like to use this website!

**What You'll Learn:**

OAuth 2.0, OpenID, PKCE, deprecated flows, JWTs, API Gateways, and scopes. No programming knowledge needed.

- The basics of OAuth 2.0 and OpenID Connect
- Best practices for developing OAuth applications (server-side, native, and SPAs)
- How to implement an OAuth client from scratch
- How to protect an API with JWT access tokens

### Tarefa 1: Getting Started

> `https://oauth.school/exercise/introduction/`

In this exercise you'll sign up for an Auth0 developer account to follow along with the exercises. Once you have an account, you'll be able to create applications and configure your OAuth server to be able to follow along with the assignments.

**Create an API Resource:**

- Applications --> APIs --> `+ Create API`
- Name: `API`
- Identifier: `https://api.example.com`
- JSON Web Token (JWT) Profile: `Auth0`
- JSON Web Token (JWT) Singing Algorithm: `RS256`
- `Create`

**Set the Default Audience:**

- `Settings`
- API Authorization Settings
  - Default Audience: `https://api.example.com`
  - `Save`

**Find the Issuer URI:**

- Applications --> Applications --> `+ Create Application`
- Name: `My App`
- Choose an application type: `Native`
- `Create`
- `Settings` tab
- Expand `Advanced Settings`
- `Endpoints` tab
- OpenID Configuration URL: `https://tarsoqueiroz.ca.auth0.com/.well-known/openid-configuration`

```json
{
	
  "issuer":"https://tarsoqueiroz.ca.auth0.com/",
  "authorization_endpoint":"https://tarsoqueiroz.ca.auth0.com/authorize",
  "token_endpoint":"https://tarsoqueiroz.ca.auth0.com/oauth/token",
  "device_authorization_endpoint":"https://tarsoqueiroz.ca.auth0.com/oauth/device/code",
  "userinfo_endpoint":"https://tarsoqueiroz.ca.auth0.com/userinfo",
  "mfa_challenge_endpoint":"https://tarsoqueiroz.ca.auth0.com/mfa/challenge",
  "jwks_uri":"https://tarsoqueiroz.ca.auth0.com/.well-known/jwks.json",
  "registration_endpoint":"https://tarsoqueiroz.ca.auth0.com/oidc/register",
  "revocation_endpoint":"https://tarsoqueiroz.ca.auth0.com/oauth/revoke",
  "scopes_supported":,
 ,"picture","created_at","identities","phone","address"],
  "response_types_supported":["code","token","id_token","code token","code id_token","token id_token","code token id_token"],
  "code_challenge_methods_supported":["S256","plain"],
  "response_modes_supported":["query","fragment","form_post"],
  "subject_types_supported":["public"],
  "token_endpoint_auth_methods_supported":["client_secret_basic","client_secret_post","private_key_jwt","tls_client_auth","self_signed_tls_client_auth"],
  "token_endpoint_auth_signing_alg_values_supported":["RS256","RS384","PS256"],
  "claims_supported":["aud","auth_time","created_at","email","email_verified","exp","family_name","given_name","iat","identities","iss","name","nickname","phone_number","picture","sub"],
  "request_uri_parameter_supported":false,
  "request_parameter_supported":true,
  "id_token_signing_alg_values_supported":["HS256","RS256","PS256"],
  "tls_client_certificate_bound_access_tokens":true,
  "request_object_signing_alg_values_supported":["RS256","RS384","PS256"],
  "backchannel_logout_supported":true,
  "backchannel_logout_session_supported":true,
  "end_session_endpoint":"https://tarsoqueiroz.ca.auth0.com/oidc/logout",
  "backchannel_authentication_endpoint":"https://tarsoqueiroz.ca.auth0.com/bc-authorize",
  "backchannel_token_delivery_modes_supported":["poll"],
  "global_token_revocation_endpoint":"https://tarsoqueiroz.ca.auth0.com/oauth/global-token-revocation/connection/{connectionName}",
  "global_token_revocation_endpoint_auth_methods_supported":["global-token-revocation+jwt"]
}
```

- **Issuer URL:** `https://tarsoqueiroz.ca.auth0.com/`
- **Authorization Endpoint:** `https://tarsoqueiroz.ca.auth0.com/authorize`
- **Token Endpoint:** `https://tarsoqueiroz.ca.auth0.com/oauth/token`

### Tarefa 2: OAuth for Web Server Applications

> `https://oauth.school/exercise/web/`

This exercise will walk you through the OAuth flow you would use if you are building a web server based application. The goal of this exercise is to use the authorization code flow with PKCE to get an access token.

**Endpoints:**

- Issuer URL: `https://tarsoqueiroz.ca.auth0.com/`
  - We'll save the issuer URL to use it when checking your work in the following exercises
- Authorization Endpoint: `https://tarsoqueiroz.ca.auth0.com/authorize`
  - Find your server's authorization endpoint and enter it here
- Token Endpoint: `https://tarsoqueiroz.ca.auth0.com/oauth/token`
  - Find your server's token endpoint and enter it here

The goal of this exercise is to get an access token using the authorization code flow and PKCE as a confidential client. This exercise will walk you through the flow manually without writing any code. You are of course free to write code to do this instead if you’d like, but the instructions here will show you the step by step process of what’s happening under the hood.

- Applications --> Applications --> `+ Create Application`
- Name: `Web App`
- Choose an application type: `Regular Web Applications`
- `Create`
- `Settings` tab

**Regular Web Application:**

- Name: `Web App`
- Domain: `tarsoqueiroz.ca.auth0.com`
- Client ID: `k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam`
- Client Secret: `DMp0TVbRvNvfek2KMK8xpOAwsRcRq1MtaxflHVZIiq4fJvVTUNWk9rIfSSSKBwtJ`
- Application URIs --> Allowed Callback URLs: `https://example-app.com/redirect`
- `Save`

**PKCE Code Verifier:**

- Code Verifier: `ec495c76a175e0b415842da6ea8c05c5e7c9b4518910654bb8db74e3`
- Code Challenge: `CRDdYeX28-A6Tg3Ep83mZm1a8b-0g5e_mJEZxFqHs8g`

**Authorization Request:**

```sh
https://tarsoqueiroz.ca.auth0.com/authorize?
  response_type=code&
  client_id={YOUR_CLIENT_ID}&
  state={RANDOM_STRING}&
  redirect_uri=https://example-app.com/redirect&
  code_challenge={YOUR_CODE_CHALLENGE}&
  code_challenge_method=S256

https://tarsoqueiroz.ca.auth0.com/authorize?
  response_type=code&
  client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam&
  state=ec495c76a175e0b415842da6ea8c05c5e7c9b4518910654bb8db74e3&
  redirect_uri=https://example-app.com/redirect&
  code_challenge=CRDdYeX28-A6Tg3Ep83mZm1a8b-0g5e_mJEZxFqHs8g&
  code_challenge_method=S256

https://tarsoqueiroz.ca.auth0.com/authorize?response_type=code&client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam&state=ec495c76a175e0b415842da6ea8c05c5e7c9b4518910654bb8db74e3&redirect_uri=https://example-app.com/redirect&code_challenge=CRDdYeX28-A6Tg3Ep83mZm1a8b-0g5e_mJEZxFqHs8g&code_challenge_method=S256
```

- `Log In`

```text
Congrats!

The authorization server redirected you back to the app and issued an authorization code!

You can exchange this authorization code for an access token now!

Your app can read the authorization code and state from the URL, and they are printed below for your convenience as well.

code=bxck7YurGPS4lyofMSf6C-xGfUDedCDWdJjvTj2AXRw_2

state=ec495c76a175e0b415842da6ea8c05c5e7c9b4518910654bb8db74e3

You should verify that the state parameter here matches the one you set at the beginning. Otherwise it's possible someone is trying to trick your app!
```

Exchange that temporary authorization code for an access token:

- Code: `J24GE1H56mlb4Xg0s-8-glO6IBAUdVGhG83PMSZ44gMRF`
- State: `ec495c76a175e0b415842da6ea8c05c5e7c9b4518910654bb8db74e3`

```sh
curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id={YOUR_CLIENT_ID} \
  -d client_secret={YOUR_CLIENT_SECRET} \
  -d code_verifier={YOUR_CODE_VERIFIER} \
  -d code={YOUR_AUTHORIZATION_CODE}

curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam \
  -d client_secret=DMp0TVbRvNvfek2KMK8xpOAwsRcRq1MtaxflHVZIiq4fJvVTUNWk9rIfSSSKBwtJ \
  -d code_verifier=ec495c76a175e0b415842da6ea8c05c5e7c9b4518910654bb8db74e3 \
  -d code=J24GE1H56mlb4Xg0s-8-glO6IBAUdVGhG83PMSZ44gMRF
```

**Token Response:**

```json
{
  "access_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJpc3MiOiJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMDEyMzQwODk5OTU1Nzk0MjY0NjkiLCJhdWQiOiJodHRwczovL2FwaS5leGFtcGxlLmNvbSIsImlhdCI6MTc1NTIwMTk4NSwiZXhwIjoxNzU1Mjg4Mzg1LCJhenAiOiJrNWlBc1NpSWprR0pFVDFmZjJXcTh4UHV5cVVRTFdhbSJ9.4KPV1XvgMIsfjaMit-aA_7wBh0fl4KZat2LzjyD8J8q4gIdZ-GN3G46POhsb-S0N1xqLzMlUJki09RFdVgGdW5AIxT-hHzWO629-JByru5ipgsSgSDEm3iz-9FE60cTp8Ml9BP0Ft3C1WwgIpz333DI4fPWXnwg4zymJKFZJS-Lq1ji2S31SQ2cjNZEZtKk6CHzdO2QsU6UTLGJEKZVgHHIU9ZMVWCDxqp6P_33j_TmHjoGVcKp4_fHJsnVp5FgAvnMVSJrPaa8Fkjxc-NqOJwRYSgXWxJxuFhvr4ECI3Bg1V_rVywx3ucBlWFj7UNar3KVLYcZpXSOsHQgQQYSsJw",
  "expires_in":86400,
  "token_type":"Bearer"
}
```

### Tarefa 3: Refresh Tokens

> `https://oauth.school/exercise/refresh/`

In this exercise you’ll learn how to obtain a refresh token and use it to get new access tokens.

The goal of this exercise is to get a refresh token and use the refresh token to get a new access token. We will be building on the previous exercise where you used the authorization code flow to get an access token.

- Applications --> APIs --> `API` --> Access Settings
- Allow Offline Access: `ON`
- `Save`

Start new OAuth flow and request a refresh token:

- Code Verifier: `a4bb1a9220ea44fb32a8b0a37f380f63289af06e99827290f50b091e`
- Code Challenger: `w9VxQZVluYaMd6a71GExsmOBHCK-b6LiCMGX_opByw8`
- Client Id: `k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam`
- Authorization Request:

```sh
https://tarsoqueiroz.ca.auth0.com/authorize?
    response_type=code&
    client_id={YOUR_CLIENT_ID}&
    state={RANDOM_STRING}&
    scope=offline_access&
    redirect_uri=https://example-app.com/redirect&
    code_challenge={YOUR_CODE_CHALLENGE}&
    code_challenge_method=S256

https://tarsoqueiroz.ca.auth0.com/authorize?
    response_type=code&
    client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam&
    state=a4bb1a9220ea44fb32a8b0a37f380f63289af06e99827290f50b091e&
    scope=offline_access&
    redirect_uri=https://example-app.com/redirect&
    code_challenge=w9VxQZVluYaMd6a71GExsmOBHCK-b6LiCMGX_opByw8&
    code_challenge_method=S256

https://tarsoqueiroz.ca.auth0.com/authorize?response_type=code&client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam&state=a4bb1a9220ea44fb32a8b0a37f380f63289af06e99827290f50b091e&scope=offline_access&redirect_uri=https://example-app.com/redirect&code_challenge=w9VxQZVluYaMd6a71GExsmOBHCK-b6LiCMGX_opByw8&code_challenge_method=S256
```

- `Log In`

```text
Congrats!

The authorization server redirected you back to the app and issued an authorization code!

You can exchange this authorization code for an access token now!

Your app can read the authorization code and state from the URL, and they are printed below for your convenience as well.

code=59rEDEAehNYRKbTRqNK509l3YtUWx3n4L4wuL_Z3FXRNM

state=a4bb1a9220ea44fb32a8b0a37f380f63289af06e99827290f50b091e

You should verify that the state parameter here matches the one you set at the beginning. Otherwise it's possible someone is trying to trick your app!
```

**Getting Token:**

```sh
curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id={YOUR_CLIENT_ID} \
  -d client_secret={YOUR_CLIENT_SECRET} \
  -d code_verifier={YOUR_CODE_VERIFIER} \
  -d code={YOUR_AUTHORIZATION_CODE}

curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam \
  -d client_secret=DMp0TVbRvNvfek2KMK8xpOAwsRcRq1MtaxflHVZIiq4fJvVTUNWk9rIfSSSKBwtJ \
  -d code_verifier=a4bb1a9220ea44fb32a8b0a37f380f63289af06e99827290f50b091e \
  -d code=59rEDEAehNYRKbTRqNK509l3YtUWx3n4L4wuL_Z3FXRNM
```

**Token Response:**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJpc3MiOiJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMDEyMzQwODk5OTU1Nzk0MjY0NjkiLCJhdWQiOiJodHRwczovL2FwaS5leGFtcGxlLmNvbSIsImlhdCI6MTc1NTI4NjU1OCwiZXhwIjoxNzU1MzcyOTU4LCJzY29wZSI6Im9mZmxpbmVfYWNjZXNzIiwiYXpwIjoiazVpQXNTaUlqa0dKRVQxZmYyV3E4eFB1eXFVUUxXYW0ifQ.td1DcdCpESCgnM5YBsZS6_TUKVTF_uVYSXy73ownDUBjY-s3CE_PecQs5bWInrVcbYn8isk_Ko7ap4ml6qzAN9O3GxHwyspccq9BG7lHENKtsXXrH5vPK24AlwSmrH1KTISiKSKy6jda9KtdvPLf_5fvpktura-hUWj_aDlEtvW4mdBVfIU2nxEfPiYq0dSBoCaxCsYq9-0d8sJOy20QAg2xpRwStBRtYryk1YT-JmFqI3-k7YGkLP_6YofB1j_C0HrP3stjIMLzkFvWU4SMzIpJVISPuOf0CgyMc9XsItyGayKHZ5GZWJqMbcqq0lhtARh8GRB8plinZUN1-2AQJw",
  "refresh_token": "825yrd71kYbFropBM84DS1tQfVVEmGfVkjBBy7YyX9gnX",
  "scope": "offline_access",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

**Refresh Tokens:**

```sh
curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=refresh_token \
  -d client_id={YOUR_CLIENT_ID} \ 
  -d client_secret={YOUR_CLIENT_SECRET} \
  -d refresh_token={REFRESH_TOKEN}

curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=refresh_token \
  -d client_id=k5iAsSiIjkGJET1ff2Wq8xPuyqUQLWam \
  -d client_secret=DMp0TVbRvNvfek2KMK8xpOAwsRcRq1MtaxflHVZIiq4fJvVTUNWk9rIfSSSKBwtJ \
  -d refresh_token=825yrd71kYbFropBM84DS1tQfVVEmGfVkjBBy7YyX9gnX

```

Refresh Token Response:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJpc3MiOiJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMDEyMzQwODk5OTU1Nzk0MjY0NjkiLCJhdWQiOiJodHRwczovL2FwaS5leGFtcGxlLmNvbSIsImlhdCI6MTc1NTI4Njg5NSwiZXhwIjoxNzU1MzczMjk1LCJzY29wZSI6Im9mZmxpbmVfYWNjZXNzIiwiYXpwIjoiazVpQXNTaUlqa0dKRVQxZmYyV3E4eFB1eXFVUUxXYW0ifQ.nQTJZ8-11gzDfzgn8Oo17EC6sW5dulK9OQPgYsUUecn5pKBKgRToFAXu6M_L9Tzl36BDmu4TKjBH5PDdyEpygNFT5yCf4gCDLzqxB7_gTo_MY9XKYPUBMftNM1VxAdcz7D3sSeu-fApv4R9L32urNg8OJW3JCwA7B3dlxL2g95Ou8qb2LbQqxjspKDxU2ncvfFaJdLmsd2NOspGwAGgj1LWw1whtVDF0dKq98LcDSXYtLGINtvgaAZzwyviES0UWdz02ZmThfzeSkthKaK3KwVQgOGWe5mJY8lsRgz9fXw2wOiDWFLSZEoazP5bF6ka0ONN67jYaPRJQ_VgvRnaR6g",
  "scope": "offline_access",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

### Tarefa 4: OAuth for Single-Page Applications

> `https://oauth.school/exercise/spa/`

This exercise will walk you through the OAuth flow you would use if you are building a single-page application. The goal of this exercise is to use the authorization code flow with PKCE to get an access token.

- Applications --> Applications --> `+ Create Application`
- Name: `My SPA`
- Choose an application type: `Single Page Web Applications`
- `Create`
- `Settings` tab
- Application URIs --> Allowed Callback URLs: `https://example-app.com/redirect`

**Basic Information:**

- Name: `My App`
- Domain: `tarsoqueiroz.ca.auth0.com`
- Client ID: `0JGyioXaoTnSAhgUaSc1rD64GMnFjbma`
- Client Secret: `_9UtfgLNKWUnlVmWVfW4puYhULLe7JK6bQWfRe10xfsyFQsdNugtLJrtTAWsvjUy`

URLs from Getting Started:

- Issuer URL: `https://tarsoqueiroz.ca.auth0.com/`
- Authorization Endpoint: `https://tarsoqueiroz.ca.auth0.com/authorize`
- Token Endpoint: `https://tarsoqueiroz.ca.auth0.com/oauth/token`

**PKCE Code Verifier:**

- Code Verifier (random string): `61529d11e81e75df55f63cbf3a4457226821cb0aafdabd2b4f41cfbb`
- Code Challenge (calculated hash): `leX4MNXUwGDaVqbsFvG6t8Rah9f51jIgPX6Mj9ktKU8`

**Authorization Request:**

```sh
https://tarsoqueiroz.ca.auth0.com/authorize?
  response_type=code&
  client_id={YOUR_CLIENT_ID}&
  state={RANDOM_STRING}&
  redirect_uri=https://example-app.com/redirect&
  code_challenge={YOUR_CODE_CHALLENGE}&
  code_challenge_method=S256

https://tarsoqueiroz.ca.auth0.com/authorize?
  response_type=code&
  client_id=0JGyioXaoTnSAhgUaSc1rD64GMnFjbma&
  state=61529d11e81e75df55f63cbf3a4457226821cb0aafdabd2b4f41cfbb&
  redirect_uri=https://example-app.com/redirect&
  code_challenge=leX4MNXUwGDaVqbsFvG6t8Rah9f51jIgPX6Mj9ktKU8&
  code_challenge_method=S256
```

- `Log In`

```text
Congrats!

The authorization server redirected you back to the app and issued an authorization code!

You can exchange this authorization code for an access token now!

Your app can read the authorization code and state from the URL, and they are printed below for your convenience as well.

code=raNkF_ZbLHXlxSzxJErZIP3zdBgSfgxoTqmmhbwhtvJHJ

state=61529d11e81e75df55f63cbf3a4457226821cb0aafdabd2b4f41cfbb

You should verify that the state parameter here matches the one you set at the beginning. Otherwise it's possible someone is trying to trick your app!
```

- Code: `raNkF_ZbLHXlxSzxJErZIP3zdBgSfgxoTqmmhbwhtvJHJ`
- State (code verifier): `61529d11e81e75df55f63cbf3a4457226821cb0aafdabd2b4f41cfbb`

**Getting Token:**

```sh
curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id={YOUR_CLIENT_ID} \
  -d code_verifier={YOUR_CODE_VERIFIER} \
  -d code={YOUR_AUTHORIZATION_CODE}

curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id=0JGyioXaoTnSAhgUaSc1rD64GMnFjbma \
  -d code_verifier=61529d11e81e75df55f63cbf3a4457226821cb0aafdabd2b4f41cfbb \
  -d code=raNkF_ZbLHXlxSzxJErZIP3zdBgSfgxoTqmmhbwhtvJHJ
```

**Token Response:**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJpc3MiOiJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMDEyMzQwODk5OTU1Nzk0MjY0NjkiLCJhdWQiOiJodHRwczovL2FwaS5leGFtcGxlLmNvbSIsImlhdCI6MTc1NTUyMTM2MSwiZXhwIjoxNzU1NjA3NzYxLCJhenAiOiIwSkd5aW9YYW9UblNBaGdVYVNjMXJENjRHTW5GamJtYSJ9.J9JJsUm1Vua7X9KgeAKVg3La1DaAciySi0QeXbqyiZz5iLJhjrKwYzK3JeYkISo23iLVna0Hp9dUteVaBDR3Z8Pbc5YVb5AUaVZfG9Y2s8VubU4kNYvz60dJZkrOAAfgU4URTodDcm2Cc0q4496lMLHGPP8uKKpq-YCU4G4itLEv5SFK7RW0i1Zkuch1bcXV41ivYRoyicwIWszJc1dfdhuqrPHqfobSO0YCrYlaH1nQFYRQb8T09owV2x3ZuOFRFUochnVk8NPQqwUohRWOFUQPqtzNdiQck85Xc_VFcvmSMj54SlrzRYdDmm7iFUprhkOx7oaRdgqr-h_QbXgGtA",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

### Tarefa 5: Getting an Access Token with the Client Credentials Grant

> `https://oauth.school/exercise/service/`

This exercise will walk you through the process of using the OAuth Client Credentials grant to get an access token.

- Applications --> Applications --> `+ Create Application`
- Name: `My M2M`
- Choose an application type: `Machine to Machine Applications`
- `Create`
- Select API: `API`
- `Authorize`
- `Settings` tab

**Basic Information:**

- Name: `My M2M`
- Domain: `tarsoqueiroz.ca.auth0.com`
- Client ID: `kOiK7T0zrgyVRYMBpBBEI5idRp6F0n85`
- Client Secret: `n-mj8z_xDB9CSn-l60nDy_ZmdXqreHnvEpN1WtC2ScluOF_ioBPwevy5kCjsMwoy`

**Request to the authorization server’s token endpoint:**

```sh
curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=client_credentials \
  -d client_id={YOUR_CLIENT_ID} \
  -d client_secret={YOUR_CLIENT_SECRET}

curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=client_credentials \
  -d client_id=kOiK7T0zrgyVRYMBpBBEI5idRp6F0n85 \
  -d client_secret=n-mj8z_xDB9CSn-l60nDy_ZmdXqreHnvEpN1WtC2ScluOF_ioBPwevy5kCjsMwoy
```

**Token Response:**

```json
{
  "access_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJpc3MiOiJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vIiwic3ViIjoia09pSzdUMHpyZ3lWUllNQnBCQkVJNWlkUnA2RjBuODVAY2xpZW50cyIsImF1ZCI6Imh0dHBzOi8vYXBpLmV4YW1wbGUuY29tIiwiaWF0IjoxNzU1NTQxMzY2LCJleHAiOjE3NTU2Mjc3NjYsImd0eSI6ImNsaWVudC1jcmVkZW50aWFscyIsImF6cCI6ImtPaUs3VDB6cmd5VlJZTUJwQkJFSTVpZFJwNkYwbjg1In0.lN_EFKYUF1FAFdt_m7LA305HYmbOaPt9DlLY7iHvoHWWv7Y725lsP026GIvxAqNvp6ZTWSZS5kPRj17ogDQexyqRdXG_7Sie2YFwLs6jR6DJA1wcwHpAvGR1D9LVoXTvnhXUzmIDYzt_8A4ttLpLIhQAR_FCATfQJ8PipF8Ty9sCWjp2RptWfbnAK57RssKSyPV2QEKgdntT9Cn6TpJxWBP6GQU5RhRJRtGLSvJtYQtihUniELK6RMPoZo23rbfYxrss0R5-wP8tuDvUQBKbgur5VrmUPvPhdVn89iszU-n7b4S7Ir3R85bcF0HjogQYCBLaEs3MmMi_5TMxAwoNJw",
  "expires_in":86400,
  "token_type":"Bearer"
}
```

### Tarefa 6: Getting the User's Name and Email Address using OpenID Connect

> `https://oauth.school/exercise/openid/`

**OpenID Connect Flow**

- Code verifier (state): `e4f680d67c3f310d0c058e4cbd2c1105dfa7f696fb3b2fecc48ca034`
- Calculate hash (code_challenge): `dYWzRoYfy1Sp_w8vZR7_oN9uydO24WwgWRp6FQ8d480`

**Authorization Request:**

```sh
https://tarsoqueiroz.ca.auth0.com/authorize?
  response_type=code&
  client_id={YOUR_CLIENT_ID}&
  state={RANDOM_STRING}&
  scope={SCOPE}&
  redirect_uri=https://example-app.com/redirect&
  code_challenge={YOUR_CODE_CHALLENGE}&
  code_challenge_method=S256

https://tarsoqueiroz.ca.auth0.com/authorize?
  response_type=code&
  client_id=0JGyioXaoTnSAhgUaSc1rD64GMnFjbma&
  state=e4f680d67c3f310d0c058e4cbd2c1105dfa7f696fb3b2fecc48ca034&
  scope=openid+profile+email&
  redirect_uri=https://example-app.com/redirect&
  code_challenge=dYWzRoYfy1Sp_w8vZR7_oN9uydO24WwgWRp6FQ8d480&
  code_challenge_method=S256
```

- `Log In`

```text
Congrats!

The authorization server redirected you back to the app and issued an authorization code!

You can exchange this authorization code for an access token now!

Your app can read the authorization code and state from the URL, and they are printed below for your convenience as well.

code=4Azoa_RSSm6gy4XIGRH-wBgmuqLhzPadoCSxja1UZ6wuM

state=e4f680d67c3f310d0c058e4cbd2c1105dfa7f696fb3b2fecc48ca034

You should verify that the state parameter here matches the one you set at the beginning. Otherwise it's possible someone is trying to trick your app!
```

- Code: `4Azoa_RSSm6gy4XIGRH-wBgmuqLhzPadoCSxja1UZ6wuM`
- State: `e4f680d67c3f310d0c058e4cbd2c1105dfa7f696fb3b2fecc48ca034`

**Get access token and ID token:**

```sh
curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id={YOUR_CLIENT_ID} \
  -d client_secret={YOUR_CLIENT_SECRET} \
  -d code_verifier={YOUR_CODE_VERIFIER} \
  -d code={YOUR_AUTHORIZATION_CODE}

curl -X POST https://tarsoqueiroz.ca.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id=0JGyioXaoTnSAhgUaSc1rD64GMnFjbma \
  -d client_secret=_9UtfgLNKWUnlVmWVfW4puYhULLe7JK6bQWfRe10xfsyFQsdNugtLJrtTAWsvjUy \
  -d code_verifier=e4f680d67c3f310d0c058e4cbd2c1105dfa7f696fb3b2fecc48ca034 \
  -d code=4Azoa_RSSm6gy4XIGRH-wBgmuqLhzPadoCSxja1UZ6wuM
```

**Token Response:**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJpc3MiOiJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMDEyMzQwODk5OTU1Nzk0MjY0NjkiLCJhdWQiOlsiaHR0cHM6Ly9hcGkuZXhhbXBsZS5jb20iLCJodHRwczovL3RhcnNvcXVlaXJvei5jYS5hdXRoMC5jb20vdXNlcmluZm8iXSwiaWF0IjoxNzU1NTQ3NDcxLCJleHAiOjE3NTU2MzM4NzEsInNjb3BlIjoib3BlbmlkIHByb2ZpbGUgZW1haWwiLCJhenAiOiIwSkd5aW9YYW9UblNBaGdVYVNjMXJENjRHTW5GamJtYSJ9.PEXF6WVH4_3hitOBGdXFTFEvg6mqYwTw-j05-f_6Fr3AUgCvJciF3GF1LrAz2RXZrBlhYIGZvn5NCNmG0IgkfR_SgcaITO45m7h3TANOhJ5VtmBd9hHtSvTqrGcbtbHdBYIKVrL1Sj0qgLFbelGEGTu-6I-w2Nnz3pwr76vVz5JsXaKes-HQt9OX07ANC3E30DQsyvwzn5lUTtTCVmkBOnpiYsIdoxc5Oz-WKj1ns5fQE4GWvz8c1Jd7_DWbwg7c9gJHv2170g_Mg3k0LoHbH4o-omUStM3u2fZEJ8AOyrkZwtwlSDnufkClPGrxjTg8kXcriXQPD2xWuGsFl5__lw",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImJoWVdZVHoyaDJ0RUl2cU5WSlRWWiJ9.eyJnaXZlbl9uYW1lIjoiVGFyc28iLCJmYW1pbHlfbmFtZSI6IlF1ZWlyb3oiLCJuaWNrbmFtZSI6InRhcnNvcXVlaXJveiIsIm5hbWUiOiJUYXJzbyBRdWVpcm96IiwicGljdHVyZSI6Imh0dHBzOi8vbGgzLmdvb2dsZXVzZXJjb250ZW50LmNvbS9hL0FDZzhvY0lBb0U3T2F6bVd2WFJQczZNRXZfNDBxZDJIbFhJSmI5V0ppZERDUy11UHozYUtSQmxHMGc9czk2LWMiLCJ1cGRhdGVkX2F0IjoiMjAyNS0wOC0xNFQxNjoyOTozNC40NDhaIiwiZW1haWwiOiJ0YXJzb3F1ZWlyb3pAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImlzcyI6Imh0dHBzOi8vdGFyc29xdWVpcm96LmNhLmF1dGgwLmNvbS8iLCJhdWQiOiIwSkd5aW9YYW9UblNBaGdVYVNjMXJENjRHTW5GamJtYSIsInN1YiI6Imdvb2dsZS1vYXV0aDJ8MTAxMjM0MDg5OTk1NTc5NDI2NDY5IiwiaWF0IjoxNzU1NTQ3NDcxLCJleHAiOjE3NTU1ODM0NzEsInNpZCI6IlZrNlgxb3Frc0cwTVY4T1FNaEpia0JKODBpOHY1QXl0In0.bHY-Za1djQqZiZoJLfYupn3oRQPdtiTpdOww3rMM_0gbxbG_ncocGxReXaaTDuL_kv2nAfIYZhvmCifqRV6H8i2NINGMJf_DjkrGFqZ3_rGHqox9_zV1HYGYDjV_V5j5jxC_SgOzdv9JTKrReiXnsnlgRLs9IT7gAyA-CWA_iYtz8m_UW2gEvQadxdDuvxDDlVQbco5oS4i6C13Lf-E8SZ6scv5KJmyFQdV8BI02tq7INja-RqZk70T888h7AbiHk_8D85yqqoI-o5VUt2a-DhEYcOCLr7eNSzSaxVJwxcuDhU9ZrUH4hASr_nkHwAnTbLfJUO8kzezNsMadHHFKPg",
  "scope": "openid profile email",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

- Base64 decoder: `https://example-app.com/base64`
- Subject (sub): `google-oauth2|101234089995579426469`
- Email address: `tarsoqueiroz@gmail.com`
- Name: `Tarso Queiroz`

### Tarefa 7: Protecting an API endpoint with Access Tokens

This exercise will demonstrate how an API can validate access tokens and allow only authorized requests through.

The goal of this exercise is to demonstrate how an API can validate access tokens issued by the authorization server. We won’t be writing any code, but everything we do in this exercise are steps you can translate to code when you’re ready.

- Applications --> APIs --> `+ Create API`
- Name: `Contacts`
- Identifier: `https://contacts.example.com`
- JSON Web Token (JWT) Signing Algorithm: `RS256`
- `Create`

OAuth client available at:

- `https://example-app.com/client`

Params:

- Issuer URL: `https://tarsoqueiroz.ca.auth0.com/`
- Authorization Endpoint: `https://tarsoqueiroz.ca.auth0.com/authorize`
- Token Endpoint: `https://tarsoqueiroz.ca.auth0.com/oauth/token`
- Client ID: `1qLZlngOd7BGHipIPAzFESTCW9KD9NgS`
- Client Secret (optional): `DOVRcfzHSG6oL6wJfmtDpP_Io20vD4qCUE8MZIqTm3cI7dHvoQGsN-sx-lcJEeGp`
- Scope to request
- Additional Fields
  - idp
  - audience --> `Tick`
    - audience: `https://contacts.example.com`
  - acr_values (okta)


To validate the access token signature:

- `https://www.jwt.io/`

### Tarefa 8: Handling revoked tokens

In this exercise we’re going to demonstrate how to revoke an access token and show how the API can find out it’s been revoked.

This exercise builds on the previous exercises Protect an API with Access Tokens, and Refresh Tokens. Make sure you’ve completed those exercises first since you’ll need to be familiar with the activities in the exercises.

Here are the steps you’ll need to have completed in the previous exercises before continuing:

- Configure an API and at least one custom scope
- Get an access token and refresh using a previously created OAuth client
- Use a refresh token to get a new access token
- Validate an access token using [jwt.io](https://jwt.io/) or your own code.

Let’s double check that you have a valid access token at this stage by pasting the token into jwt.io or validating it with a JWT library.

The token signature should validate, and the token expiration date should be in the future.

Now the idea is to demonstrate what happens when a refresh token is revoked. To demonstrate this, we’ll simulate what the client application would do when the user clicks “log out”, which is to make a request to the OAuth server’s token revocation endpoint telling it to revoke the refresh token.

Make sure you’ve obtained a refresh token by following the instructions in the **Refresh Token** exercise. Note that this exercise will work with applications both with and without a client secret.

Check that the refresh token works by using it to get a new access token.

```sh
curl https://xxxxxxx.us.auth0.com/oauth/token \
  -d grant_type=refresh_token \
  -d client_id={YOUR_CLIENT_ID} \
  -d client_secret={YOUR_CLIENT_SECRET} \
  -d refresh_token={REFRESH_TOKEN}
```

If you’re seeing that the refresh token is still valid, it means the revocation request failed. The response from the revocation endpoint will always be HTTP 200, even if the token is invalid or has already been revoked. Additionally, a client can only revoke tokens that were issued to that same client, so if you’re having trouble here double check that the credentials you’re using in the revocation request belong to the same client that you used to get the refresh token.

Auth0’s access tokens can’t be revoked, so you’ll need to consider the access token lifetime when designing your system.

### Tarefa 9: Enforce scopes in your API

In this exercise you’ll create some additional custom scopes for your API and request access tokens with those scopes.

This exercise will walk you through creating some additional custom scopes for your API, as well as requesting and validating access tokens with those scopes.

You’ll need to be familiar with the earlier exercises, so make sure you have already:

- Set up your Auth0 account and created an API resource
- Set up a client application to be able to request access tokens
- Familiarized yourself with using jwt.io to validate access tokens, or have built an API that validates JWT access tokens

From your API’s settings page, click on the Permissions tab to show your list of current scopes for the API. This is where we’ll add two new scopes, representing finer grained access to the API.

Create two new scopes to represent create and delete access to this resource. The actual scope name is your choice, but we’ll follow a convention of prepending “create:photos” and “delete:photos” to keep things simple.

Create a new scope called “create:photos”, and add a display phrase such as “Upload photos to your account”.

Add another scope called “delete:photos” with a similar description.

Back on your API’s Settings page, uncheck the setting “Allow skipping user consent”. This will make it so that you are always asked for consent when an application requests these scopes, even if it is a first-party app.

Now that these scopes are created, your application can request them. Build the authorization URL, but this time request all of your custom scopes.

```sh
https://xxxxxx.us.auth0.com/authorize?
  response_type=code&
  scope=create:photos+delete:photos&
  client_id={YOUR_CLIENT_ID}&
  state={RANDOM_STRING}&
  redirect_uri=https://example-app.com/redirect&
  code_challenge={YOUR_CODE_CHALLENGE}&
  code_challenge_method=S256
```

Visit that URL in your browser, but this time instead of being redirected back immediately, you will see a prompt asking for permission.

Click Accept and you’ll be redirected back to the redirect URL. Go ahead and exchange that authorization code for an access token just like in the earlier exercises, the sample curl command is repeated below.

```sh
curl -X POST https://xxxxxx.us.auth0.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id={YOUR_CLIENT_ID} \
  -d client_secret={YOUR_CLIENT_SECRET} \
  -d code_verifier={YOUR_CODE_VERIFIER} \
  -d code={YOUR_AUTHORIZATION_CODE}
```

This time when you get back the token response, you should see that it also includes your new custom scopes!

```sh
… "scope":"photos create:photos delete:photos" …
```

Now that the application has an access token, it would make an API request to the API. The API then needs to validate the access token, which it can do the same way as in the previous exercise “Protecting an API with Access Tokens”.

Paste the JWT token into jwt.io and it will validate the signature of the token and show you the contents.

This time, in addition to just checking the signature and audience, your API now also needs to verify which scopes are contained in the token.

The “scope” property will be a space-separated string of scopes that were issued in this token. Your API would look at that string, and depending on whether you’re running the “create” or “delete” API operation, confirm that the corresponding scope is present in the token.

For extra credit, try this again but request only one of your custom scopes and see how that changes the contents of the token.

## That's all

...folks!!!
