# Master OAuth 2.0: A Practical Guide to API Security

## About

> [`https://www.udemy.com/course/master-oauth-2-api-security-practical-guide`](https://www.udemy.com/course/master-oauth-2-api-security-practical-guide)

OAuth 2.0 with practical flows, implementations, real-world use cases, and decision-making for robust API architectures.

### O que se aprenderá

- Learn OAuth 2.0 Token Types and Formats: Explore access tokens, refresh tokens, JWT, and opaque tokens to secure modern APIs.
- Understand Token Validation Methods: Discover when to use local validation or introspection for efficient and secure token verification.
- Choose the Right OAuth Client Type: Learn when to use public or confidential clients.
- Define and Structure OAuth Scopes: Learn to name and structure OAuth scopes for effective, granular API access control.
- Gain Basics of OIDC: Understand how OpenID Connect extends OAuth for user authentication and single sign-on.
- Master User-Initiated Flows: Learn Implicit, Authorization Code, and PKCE flows for secure user authentication.
- Explore Flow Challenges: Analyze vulnerabilities with real-world hacker scenarios and address them effectively.
- Choose the Best Flow: Use decision trees to identify the ideal OAuth flow for your project needs.
- Discover Advanced Flow Mechanisms: Learn JWT Secured Authorization Request (JAR), JWE, and Pushed Authorization Request (PAR) to enhance OAuth 2.0 security.
- Implement Machine-to-Machine Flows: Learn the Client Credentials Flow for secure backend service communication.
- Understand ROPC and Device Code Flows: Discover flows like Resource Owner Password Credentials and Device Code for resource owner and devices with limited input
- Master Advanced Advanced Client Authentication Methods: Use JWT, SAML assertions, and X.509 mTLS for robust API security.
- Learn mTLS X.509 Basics: Build foundational knowledge of mutual TLS, X.509 certificates, and Public Key Infrastructure (PKI).
- Secure OAuth 2.0 Access Tokens: Protect your tokens with advanced FAPI-compliant mechanisms like mutual TLS and Demonstration of Proof-of-Possession (DPoP).
- Integrate External Identity Providers: Connect with partners, JWT providers, and external systems for scalable identity solutions.
- Connect with Legacy and SAML Systems: Integrate with legacy infrastructures and SAML for phased migrations.
- Simulate Real-World Scenarios: Analyze attacker scenarios and explore diverse project architectures.
- Make Informed Decisions: Use decision trees to select the best OAuth flows and mechanisms for secure architectures.

## OAuth 2.0 Basics

### Authorization Code Grant Flow

Fluxo Padrão do OAuth 2.0 (Authorization Code Grant)

- **Solicitação de Autorização:**
  - O cliente redireciona o usuário ao servidor de autorização com parâmetros como `client_id`, `redirect_uri`, `scope` e `state`.
- **Autenticação e Consentimento:**
  - O usuário autentica-se e concede permissão ao cliente para acessar os recursos solicitados.
- **Código de Autorização:**
  - O servidor de autorização redireciona o usuário de volta ao cliente com um **código de autorização** (`code`) no parâmetro da URL.
- **Solicitação de Token:**
  - O cliente troca o código de autorização por um **token de acesso** (`access token`), enviando uma requisição direta (`back-channel`) ao servidor de autorização com `client_id`, `client_secret`, `code` e `redirect_uri`.
- **Token de Acesso:**
  - O servidor de autorização valida as credenciais e emite um `access token` (e opcionalmente um `refresh token`).
- **Acesso ao Recurso:**
  - O cliente usa o token de acesso para acessar recursos protegidos no servidor de recursos via cabeçalho `Authorization: Bearer <token>`.
- **`Refresh Token` (Opcional):**
  - Se o token expirar, o cliente pode usar o `refresh token` para obter um novo `access token` sem interação do usuário.

![Authorization Code Grant](./imagens/Authorization_Code_Grant.png)

**Pontos-Chave**:

- **Fluxo mais seguro** para aplicações web com back-end.
- Usa **código de autorização** (nunca tokens no front-channel).
- Exige **client_secret** (apenas para clientes confidenciais).
- **PKCE** é recomendado para clientes públicos (ex.: mobile/SPA).

### Roles (Papéis) do OAuth 2.0

1. **Resource Owner** (Proprietário do Recurso)
  - **O que é:** O usuário final que possui os dados protegidos e pode conceder acesso a eles.
  - **Exemplo:** Você, ao autorizar um app a acessar suas fotos no Google Fotos.
2. **Client** (Cliente)
  - **O que é:** A aplicação que solicita acesso aos recursos do usuário em seu nome.
  - **Exemplo:** Um app de impressão de fotos que quer acessar suas imagens.
3. **Resource Server** (Servidor de Recurso)
  - **O que é:** O servidor que hospeda os recursos protegidos do usuário e aceita tokens de acesso para permitir o acesso.
  - **Exemplo:** API do Google Drive que armazena seus arquivos.
4. **Authorization Server** (Servidor de Autorização)
  - **O que é:** O servidor que autentica o usuário, obtém seu consentimento e emite tokens de acesso aos clientes.
  - **Exemplo:** Serviço de login do Google que emite tokens após a autenticação.
5. **User Agent** (Agente do Usuário)
  - **O que é:** O navegador ou aplicativo móvel que atua como intermediário nas interações front-channel (redirecionamentos).
  - **Exemplo:** Chrome, Safari ou o app nativo do celular que redireciona o usuário durante o fluxo OAuth.

![OAuth 2.0 Roles](./imagens/OAuth20_Roles.png)

**Observações**:

- O **Authorization Server** e **Resource Server** podem ser da mesma organização (ex.: Google) ou separados.
- O **Client** pode ser **público** (não guarda segredos, como SPAs) ou **confidencial** (aplicações com back-end).

### OAuth 2.0: Public vs. Confidential Clients

**Confidential Clients**:

- **Capacidade de Armazenar Segredos**: Podem manter credenciais confidenciais (ex.: `client_secret`) de forma segura.
- **Ambiente Controlado**: Executam em servidores seguros (back-end), onde o código e os segredos não são expostos ao usuário final.
  - **Exemplos**:
      - Aplicações web tradicionais (ex.: Java Spring, Node.js com servidor).
      - Serviços backend (ex.: microserviços).
- **Autenticação**: Usam `client_secret` ou métodos avançados (ex.: Private Key JWT, mTLS).
- **Segurança**: Considerados mais seguros porque o `client_secret` não é acessível ao público.

**Public Clients**:

- **Incapacidade de Armazenar Segredos**: Não podem manter credenciais confidenciais porque o código é executado em ambientes não confiáveis.
- **Ambiente Exposto**: Executam no dispositivo do usuário (front-end), onde segredos podem ser extraídos.
  - **Exemplos**:
    - Single-Page Applications (SPAs) (ex.: React, Angular).
    - Aplicativos móveis nativos (ex.: Android, iOS).
    - Aplicações desktop.
- **Autenticação**: Usam PKCE (Proof Key for Code Exchange) para substituir o client_secret e prevenir ataques.
- **Segurança**: Dependem de mecanismos como PKCE e redirecionamentos seguros.

**Diferenças Chave**:

| Aspecto       | Confidential Clients                   | Public Clients                        |
| :------------ | :------------------------------------- | :------------------------------------ |
| Armazenamento | Ambientes seguros (servidor)           | Dispositivos do usuário (inseguros)   |
| Autenticação  | `client_secret`, Private Key JWT, mTLS | PKCE (sem `client_secret`)            |
| Exemplos      | Aplicações web com back-end            | SPAs, apps móveis, apps desktop       |
| Complexidade  | Maior (gerenciamento de segredos)      | Menor (sem segredos)                  |
| Recomendações | Usar fluxo Authorization Code          | Usar Authorization Code + PKCE        |

**Por que a Distinção é Importante**?

- **Segurança**: Confidential clients podem usar métodos de autenticação mais fortes.
- **Usabilidade**: Public clients precisam de fluxos adaptados (ex.: PKCE) para compensar a falta de segredos.
- **Especificações**: OAuth 2.0 e OAuth 2.1 exigem tratamentos diferentes para cada tipo.

**Exemplo de Fluxo com PKCE (Public Clients)**:

1. Cliente gera `code_verifier` e `code_challenge`.
1. Redireciona o usuário com `code_challenge`.
1. Servidor de autorização emite um código de autorização.
1. Cliente troca o código por um token usando `code_verifier` (sem `client_secret`).

## OAuth 2.0 User-Initiated Flows (day 2)

Secure authorization for Web and Mobile applications.

## Advanced Security for User-Initiated OAuth 2.0 Flows (day 3)

## Understanding OAuth 2.0 ROPC Flow: Risks and When to Use It

## OAuth 2.0 Machine-to-Machine (M2M)

Mastering the Client Credentials Flow

## Mastering OAuth 2.0 Device Code Flow for Limited-Input-Devices

## Integrating External Identity Providers with OAuth 2.0 using JWT and SAML (day 4)

## OAuth 2.0 Advanced Client Authentication Methods

## OAuth 2.0 Advanced Token Security Mechanisms

X.509 mTLS and DPoP.

## That's all

...folks!!!
