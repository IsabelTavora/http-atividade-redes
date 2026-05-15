# Relatório — Laboratório de Inspeção HTTP/HTTPS — Fluxo B (sem privilégio administrativo)

> **Como usar este template:** substitua os campos `[...]` pelas suas respostas,
> anexe as capturas de tela na pasta `evidencias/` e referencie-as onde indicado.
> Preserve a formatação markdown (tabelas, blocos de código) para facilitar a correção.
>
> **Observação:** toda a análise prática deste relatório é feita sobre tráfego **HTTP em texto claro**. A análise de HTTPS é teórica, baseada na fundamentação do `readme.md` do repositório.

---

## Identificação

| Campo                                       | Valor                                         |
|---------------------------------------------|-----------------------------------------------|
| Nome                                        | Isabel Tavora Nunes                           |
| RA                                          |                                       |
| Disciplina                                  | Redes de Computadores                         |
| Turma                                       | ADS Noturno                                   |
| Data                                        | 15/05/2026                                    |
| Fluxo                                       | **B — Aluno sem privilégio de administrador** |
| SO utilizado                                | Windows 11                                    |
| Ferramenta de proxy                         | Fiddler Classic per-user                      |
| Navegador(es)                               | Chrome 124                                    |
| HTTPS-First Mode / HTTPS-Only desabilitado? | sim                                           |

---

## Atividade 1 — Primeira captura (`http://example.com`)

**Captura de tela:** `evidencias/atv1_sessao.png`

**Request-line enviada:**

```http
GET http://example.com/ HTTP/1.1
```

**Status-line recebida:**

```http
HTTP/1.1 200 
```

### Pergunta 1.1
> Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Resposta:**
8

Cabeçalhos:
- Host: example.com
- Connection: keep-alive
- Upgrade-Insecure-Requests: 1
- User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36
- Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
- Accept-Encoding: gzip, deflate
- Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7
- If-Modified-Since: Sat, 09 May 2026 09:00:11 GMT

### Pergunta 1.2
> Qual foi o `Content-Length` da resposta? Se ele não apareceu, registre `Transfer-Encoding`, versão do protocolo ou outro indício observado. O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

**Resposta:** O cabeçalho Content-Length não veio na resposta. Em vez disso, apareceram:
Transfer-Encoding: chunked
Content-Encoding: gzip
Versão do protocolo: HTTP/1.1
Isso mostra que o corpo foi compactado e enviado em pedaços. O tipo de conteúdo é HTML, indicado pelo cabeçalho Content-Type.

---

## Atividade 2 — Anatomia de um GET (`http://httpbin.org/get?...`)

**Captura de tela:** `evidencias/atv2_raw.png`

**Request-line completa:**

```http
GET /get?aluno=ISABEL_TAVORA_NUNES&curso=redes HTTP/1.1
```

**Cabeçalhos-chave capturados:**

| Cabeçalho    | Valor                    |
|--------------|--------------------------|
| `Host`       | httpbin.org                    |
| `User-Agent` | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36                    |
| `Accept`     | text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.7                    |

**Campos do JSON de resposta:**

```json
{
  "args": {
    "aluno": "ISABEL_TAVORA_NUNES",
    "curso": "redes"
  },
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7",
    "Host": "httpbin.org",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36"
  },
  "origin": "168.205.69.195",
  "url": "http://httpbin.org/get?aluno=ISABEL_TAVORA_NUNES&curso=redes"
}

```

### Pergunta 2.1
> O valor do campo `origin` corresponde a qual elemento da rede? Por que normalmente não é o IP local?

**Resposta:** O campo origin mostra o IP público da máquina na internet. Basicamente, é o endereço de saída que o roteador ou o provedor de internet atribui.

### Pergunta 2.2
> Compare o `User-Agent` enviado com o que aparece no JSON da resposta. Coincidem?

**Resposta:** Sim

### Pergunta 2.3
> Em `http://httpbin.org/headers`, liste até três cabeçalhos que o servidor vê mas **não aparecem** no Raw do request. De onde vêm? Se não encontrar três, explique por que o resultado pode variar.

**Resposta:**

| Cabeçalho visto pelo servidor | Origem provável | Observação |
|-------------------------------|-----------------|------------|
| X-Amzn-Trace-Id               | Infraestrutura da Amazon (proxy reverso do httpbin)| Usado para rastrear requisições na nuvem AWS      |
| Via                         | Proxy ou balanceador de carga           | Indica passagem por intermediário HTTP      |
| X-Forwarded-For                         | Proxy ou NAT do provedor           | 
Registra o IP original do cliente antes do proxy|

---

## Atividade 3 — POST e envio de formulário (`http://httpbin.org/forms/post` → `/post`)

**Captura de tela:** `evidencias/atv3_post_raw.png`

**Request-line do POST:**

```http
POST http://httpbin.org/post HTTP/1.1 
```

**Cabeçalhos do request:**

| Cabeçalho        | Valor |
|------------------|-------|
| `Content-Type`   | application/x-www-form-urlencoded |
| `Content-Length` | 186 |

**Corpo completo do request:**

```
custname=Isabel+Tavora+Nunes&custtel=01300000009&custemail=isabeltavoran%40gmail.com&size=large&topping=bacon&topping=cheese&topping=onion&delivery=11%3A00&comments=Yreviled+Snoitcurtsni

```

**Trecho do JSON de resposta (campo `form`):**

```json
"form": {
  "comments": "Yreviled Snoitcurtsni",
  "custemail": "isabeltavoran@gmail.com",
  "custname": "Isabel Tavora Nunes",
  "custtel": "01300000009",
  "delivery": "11:00",
  "size": "large",
  "topping": ["bacon", "cheese", "onion"]
}
```

### Pergunta 3.1
> Qual o formato do corpo? Como esse formato codifica caracteres especiais (espaço, acentos)?

**Resposta:** foi enviado no formato application/x-www-form-urlencoded, os pares chave=valor são concatenados com &, e caracteres especiais são codificados em % e hexadecimal referente ao caracter.

### Pergunta 3.2
> Comparando **Request → WebForms** e **Request → Raw**: qual das duas corresponde literalmente aos bytes enviados no socket TCP?

**Resposta:** A aba Raw.

### Pergunta 3.3 — Composer
> Envie manualmente via Composer um `POST` para `http://httpbin.org/post` com JSON. Registre a resposta. Qual campo do JSON confirma que o servidor interpretou o JSON?

**Captura de tela:** `evidencias/atv3_composer.png`

**Response JSON (trecho relevante):**

```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Content-Length": "0", 
    "Content-Type": "application/json", 
    "Host": "httpbin.org", 
    "X-Amzn-Trace-Id": "Root=1-6a077958-03fb2d2a4c1358b9595a4366"
  }, 
  "json": null, 
  "origin": "168.205.69.195", 
  "url": "http://httpbin.org/post"
}

```

**Resposta:** Imagino que seria o json: null, mas veio vazio.
---

## Atividade 4 — Catálogo de status codes (`http://httpbin.org/...`)

**Captura de tela (lista do Fiddler com as 7 sessões):** `evidencias/atv4_lista.png`

| # | Método | URL | Status-line | `Content-Length` / `Transfer-Encoding` | Body presente? |
|---|--------|-----|-------------|-----------------------------------------|----------------|
| 1 | GET    | `http://httpbin.org/status/200` |HTTP/1.1 200 OK | [...] | não |
| 2 | GET    | `http://httpbin.org/redirect-to?status_code=301&url=/get` | HTTP/1.1 301 Moved Permanently | [...] | não |
| 3 | GET    | `http://httpbin.org/status/404` | HTTP/1.1 404 Not Found | [...] | não |
| 4 | GET    | `http://httpbin.org/status/418` |HTTP/1.1 418 I'm a teapot| 135 | sim |
| 5 | GET    | `http://httpbin.org/status/500` | HTTP/1.1 500 Internal Server Error | [...] | não |
| 6 | GET    | `http://httpbin.org/status/503` | HTTP/1.1 503 Service Unavailable| [...] | não |
| 7 | GET    | `http://httpbin.org/cache` com `If-Modified-Since` | HTTP/1.1 304 Not Modified | [...] | não |

### Pergunta 4.1
> Em qual dos status o corpo está ausente/tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Resposta:** Os status 200, 301, 404, 500, 503 e 304 apareceram sem corpo ou com tamanho zero.No caso específico dos 304 Not Modified e 204 No Content, essa ausência não é opcional: a RFC 9110 exige que o servidor não envie nada quando não há atualização de conteúdo.

### Pergunta 4.2
> No `301`, qual cabeçalho da resposta informa para onde ir? O que aconteceria se estivesse ausente?

**Resposta:** O cabeçalho que manda no redirecionamento é o Location, que aponta para o novo endereço do recurso. Se esse campo não viesse na resposta, o navegador ficaria perdido e poderia mostrar uma mensagem de erro ou simplesmente continuar na página atual sem sair do lugar.

### Pergunta 4.3
> Diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

**Resposta:** 200 OK: o recurso foi entregue sem problemas. O navegador pode guardar o conteúdo no cache, seguindo as regras dos cabeçalhos Cache-Control, ETag e Last-Modified.
304 Not Modified: o servidor confirma que nada mudou desde a última versão salva. O navegador então reaproveita o que já tem no cache, sem precisar baixar de novo.
404 Not Found: o recurso não existe. Nesse caso, não há nada válido para guardar no cache, já que não tem conteúdo para reutilizar.
---

## Atividade 5 — Identificação de cabeçalhos (`http://httpbin.org/response-headers?...` + `/gzip`)

**Captura de tela (Inspectors → Headers):** `evidencias/atv5_headers.png`

| Cabeçalho                    | Req/Resp | Valor capturado | Função em uma frase |
|------------------------------|----------|------------------|----------------------|
| `Host`                       | Req      | httpbin.org           | Indica o nome do servidor ao qual o cliente quer se conectar                |
| `User-Agent`                 | Req    | [...]            |Identifica o software cliente (navegador, app, biblioteca) que enviou a requisição.               |
| `Accept`                     | Req    | [...]            | É usado na negociação de conteúdo.               |
| `Accept-Encoding`            | Req    | [...]            | Informa os algoritmos de compressão que o cliente suporta               |
| `Cookie`                     | Req    | [...]            | Envia ao servidor dados persistentes armazenados no navegador, como identificadores de sessão ou preferências.                |
| `Server`                     | Resp    | [...]            | Identifica o software ou framework que gerou a resposta               |
| `Content-Type`               | Resp    | [...]            | Define o tipo MIME e o charset do corpo da resposta                |
| `Content-Encoding`           | Resp    | [...]            | Indica que o corpo foi compactado antes do envio                |
| `Set-Cookie`                 | Resp    | [...]            | Instrui o cliente a criar ou atualizar um cookie local               |
| `Cache-Control`              | Resp    | [...]            | Define regras de cache                |
| `Strict-Transport-Security`  | Não esperado em HTTP — ver Pergunta 5.3 | — | — |

### Pergunta 5.1
> `Content-Encoding: gzip`/`br` apareceu? Compare `Content-Length`, quando presente, com o conteúdo visível. O que explica a diferença?

**Resposta:** Sim, o Content-Length mostra o tamanho do corpo compactado que foi transmitido. Já na aba TextView, o que aparece é o texto depois de descompactado.
Essa diferença rola porque o gzip reduz os dados pra otimizar a transferência, mas o Fiddler exibe o corpo já descomprimido, pronto pra leitura.

### Pergunta 5.2
> Cliente envia `Accept: application/json` mas o recurso só existe em `text/html`. Qual status code esperar?

**Resposta:** Sem esse cabeçalho, o navegador não teria como saber qual formato usar, e a resposta acaba sendo rejeitada, então seria o 406 Not Acceptable

### Pergunta 5.3
> `Strict-Transport-Security` apareceu nas respostas HTTP? Por que esse cabeçalho está ausente neste fluxo? (Consulte a RFC 6797.) Qual é seu papel contra downgrades para HTTP puro?

**Resposta:** Não, o teste foi feito em HTTP puro e não HTTPS, o servidor então escondeu esse cabeçalho porque não tem como ser HTTPS obrigatório

---

## Atividade 6 — HTTP vs HTTPS (análise sem decriptação)

**Captura de tela HTTP (`neverssl.com`):** `evidencias/atv6_http.png`
**Captura de tela HTTPS (`https://httpbin.org/get`, apenas CONNECT):** `evidencias/atv6_https.png`

### Pergunta 6.1
> Que método HTTP aparece na sessão do `https://httpbin.org/get`? O que ele faz e por que existe?

**Resposta:** Não apareceu no fiddler a sessão

### Pergunta 6.2
> Tabela comparativa dos campos visíveis ao Fiddler em cada caso:

| Campo                          | Visível em HTTP? | Visível em HTTPS (sem decriptação)? |
|--------------------------------|------------------|-------------------------------------|
| Método                         | sim            | Não                               |
| URL completa (path + query)    | sim            | Não                               |
| Cabeçalhos de request          | sim            | Não                               |
| Corpo de request               | sim            | Não                               |
| Status code                    | sim            | Não                               |
| Cabeçalhos de response         | sim            | Não                               |
| Corpo de response              | sim            | Não                               |
| Host (via SNI, no `CONNECT`)   | sim            | Não                               |
| IP e porta de destino          | sim            | Não                               |

### Pergunta 6.3 (teórica)
> O que você **veria** no Fiddler se tivesse privilégio de administrador e pudesse habilitar *Decrypt HTTPS traffic*? Indique telas/abas e justifique por que essa inspeção exige a instalação de um certificado raiz.

**Resposta:** Tudo que o http já mostra. Como o https é seguro, preciso usar um certificado para garantir que está tudo certo.

### Pergunta 6.4
> Por que a técnica de decriptação dos *debugging proxies* **não** funcionaria contra um usuário se um atacante a tentasse sem instalar o certificado?

**Resposta:** Sem o certificado raiz instalado, o navegador percebe que o proxy está interceptando o TLS e mostra um erro de segurança. A decriptação só acontece se o usuário aceitar instalar o certificado, autorizando o proxy a agir como intermediário confiável.

---

## Atividade 7 — Cookies e sessão (`http://httpbin.org/cookies/...`)

**Captura de tela da sequência:** `evidencias/atv7_cookies.png`

| # | URL | `Set-Cookie` recebido | `Cookie` enviado |
|---|-----|-----------------------|-------------------|
| 1 | `/cookies/set?...`       | [...] | [nenhum / ...] |
| 2 | `/cookies` (1ª visita)   | [...] | [...]          |
| 3 | `/cookies` (reload 1)    | [...] | [...]          |
| 4 | `/cookies` (reload 2)    | [...] | [...]          |

### Pergunta 7.1
> `Set-Cookie` aparece uma vez ou em toda requisição? Justifique.

**Resposta:** [...]

### Pergunta 7.2
> Que atributos o `Set-Cookie` trouxe? Explique cada um presente. Para atributos não observados, registre `não observado`.

> **Nota:** o httpbin define cookies mínimos — apenas o atributo `Path=/` estará presente. Para cada atributo ausente, registre **não observado** e explique o comportamento padrão do navegador na sua ausência (ex.: sem `Expires`/`Max-Age` → cookie de sessão; sem `Secure` → pode ser enviado por HTTP; sem `SameSite` → o navegador aplica a política padrão da versão em uso).

**Resposta:**

| Atributo  | Valor | Função | Observado? |
|-----------|-------|--------|------------|
| `Path`    | `/`   | [...]  | Sim        |
| `Domain`  | —     | [...]  | não observado |
| `Expires` | —     | [...]  | não observado |
| `Max-Age` | —     | [...]  | não observado |
| `Secure`  | —     | [...]  | não observado |
| `HttpOnly`| —     | [...]  | não observado |
| `SameSite`| —     | [...]  | não observado |

### Pergunta 7.3
> O atributo `Secure` pode aparecer num cookie recebido por HTTP puro? Qual seria o comportamento esperado? Relacione com o fato de que todo o tráfego desta atividade é visível em texto claro.

**Resposta:** [...]

### Pergunta 7.4
> Na aba **Inspectors → Cookies**, o cookie armazenado coincide com o campo `cookies` do JSON?

**Resposta:** [...]

---

## Atividade 8 — Manipulação com breakpoints

> Esta atividade usa os breakpoints interativos do Fiddler Classic.

**Captura de tela da edição do User-Agent:** `evidencias/atv8_ua_edit.png`

**JSON de resposta após edição:**

```json
{
  "user-agent": "LaboratorioRedes/1.0 (Aluno Isabel)"
}

```

### Pergunta 8.1
> O servidor pode detectar que o `User-Agent` foi forjado? Discuta.

**Resposta:** Imagino que não, o user agent parece ser uma string simples.

### Pergunta 8.2
> Após editar a status-line de `200 OK` para `404 Not Found`, o que o navegador exibe? Comente o papel do proxy como MITM.

**Captura de tela:** `evidencias/atv8_status_edit.png`

**Resposta:** Após alterar a resposta, o navegador exibiu uma página de erro 404, como se o recurso não existisse. O man-in-the-middle ele pode alterar qualquer parte da comunicação, e o cliente confia no que recebe porque ele acontece antes de chegar no navegador.

### Pergunta 8.3
> Confirme que todos os breakpoints foram desabilitados.

- [ ] Breakpoints desabilitados ao final (Shift+F11)

---

## Atividade 9 — Redirecionamento HTTP → HTTPS

**Captura de tela:** `evidencias/atv9_redir.png`

**Status-line da resposta a `http://httpbin.org/redirect-to?status_code=301&url=https%3A%2F%2Fhttpbin.org%2Fget`:**

```http
HTTP/1.1 301 MOVED PERMANENTLY
```

**Cabeçalho `Location` da resposta:**

```
Location: https://httpbin.org/get
```

### Pergunta 9.1
> Código de status e cabeçalho que direcionaram o navegador para `https://`.

**Resposta:** HTTP/1.1 301 MOVED PERMANENTLY e o cabeçalho foi o Location.

### Pergunta 9.2
> Além do redirecionamento 3xx, qual outro mecanismo/cabeçalho faz o navegador passar a forçar HTTPS em visitas futuras? Cite a RFC.

**Resposta:** O mecanismo é o cabeçalho Strict-Transport-Security, definido pela RFC 6797.
Ele instrui o navegador a lembrar que o domínio deve ser acessado apenas via HTTPS por um período determinado (max-age), evitando futuras conexões inseguras.

### Pergunta 9.3
> Se esse cabeçalho fosse enviado por uma resposta servida via HTTP puro, o navegador deveria obedecer? Justifique com base na RFC.

**Resposta:** O mecanismo é o cabeçalho Strict-Transport-Security, definido pela RFC 6797.
Ele instrui o navegador a lembrar que o domínio deve ser acessado apenas via HTTPS por um período determinado (max-age), evitando futuras conexões inseguras.


## Reflexão final (opcional, até 10 linhas)

> O que você aprendeu que não conhecia antes deste laboratório? Há algum
> cabeçalho, código de status ou comportamento que passou a olhar com
> mais atenção? Alguma dificuldade que recomendaria evitar para a próxima turma?

[reflexão]

---

## Encerramento — justificativa de segurança (Fluxo B)

**Parágrafo: por que a remoção de certificado é dispensável neste fluxo e por que seria obrigatória para o aluno administrador:**

Se o aluno está só mexendo com HTTP puro, não precisa se preocupar em remover certificado: o Fiddler nem instala nem usa certificado raiz nesse modo, já que não tem tráfego criptografado pra decifrar.
Agora, pro aluno administrador, que inspeciona HTTPS, aí sim a remoção é obrigatória no fim do laboratório. O certificado raiz do Fiddler dá acesso a todas as conexões seguras da máquina, e deixar isso instalado depois seria abrir brecha pra interceptação indevida.

- [x] HTTPS-First Mode / HTTPS-Only Mode reabilitado no navegador
- [x] Fiddler fechado (porta de proxy liberada)
- [x] Configuração de proxy removida do navegador (se aplicável)

---

## Checklist de entrega

- [ ] Todos os campos `[...]` substituídos
- [ ] Pasta `evidencias/` com capturas nomeadas por atividade (incluindo Atv. 9)
- [ ] 11 questões de verificação respondidas
- [ ] Atividade 9 (redirecionamento HTTP→HTTPS) documentada
- [ ] Justificativa de encerramento redigida
- [ ] Arquivo compactado como `NOME_RA_LAB_HTTP_FLUXOB.zip`
- [ ] Submetido no Microsoft Teams dentro do prazo
