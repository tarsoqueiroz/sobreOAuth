# Advanced OAuth Security

> [(Udemy) Advanced OAuth Security](https://www.udemy.com/course/advanced-oauth-security/)

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

## Ataques de injeção no Authorization Code

## Como o PKCE previne a injeção no Authorization Code

- [RFC 7636: Proof Key for Code Exchange](https://oauth.net/2/pkce/)

## Calculate a PKCE code verifier and code challenge

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

## História do PKCE

## Authorization Server Mixup sttacks and prevention

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

- [RFC 6750: OAuth 2.0 Bearer Token Usage](https://oauth.net/2/bearer-tokens/)

MTLS para Sender-Constrained Access Token

DPoP para Sender-Constrained Access Token

## Non-Reputiation

- [FAPI 2.0 Message Signing (Draft)](https://openid.bitbucket.io/fapi/fapi-2_0-message-signing.html)
- [RFC 9101](https://datatracker.ietf.org/doc/html/rfc9101)
- [JWT Secured Authorization Response Mode for OAuth 2.0 (JARM) incorporating errata set 1](https://openid.net/specs/oauth-v2-jarm.html)
- [JWT Response for OAuth Token Introspection](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-jwt-introspection-response)
- [RFC9421: HTTP Message Signatures](https://oauth.net/http-signatures/)

## That's all

...folks!!!