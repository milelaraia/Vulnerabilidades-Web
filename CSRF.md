# TryHackMe - CSRF

###### Solved by @milelaraia

Esta é uma sala do TryHackMe que apresenta um mini curso sobre vulnerabilidades CSRF.

## Afinal, o que é CSRF?

CSRF `(Cross‑Site Request Forgery)` é um tipo de ataque web onde um invasor induz um usuário autenticado a realizar requisições indesejadas a um site no qual ele está logado. É como "falsificar" requisições legítimas usando `formulários HTML` e `códigos JavaScript`, fazendo com que o navegador da vítima os envie automaticamente, usando os cookies/sessão já autenticada.

## Tipos de ataque CSRF

Existem maneiras diferentes de acessar vulnerabilidades usando o CSRF, vamos analisar algumas delas.
#### A necessidade de Token CSRF

Uma das primeiras falhas apresentadas é quando a aplicação permite que `ações críticas` (alterar senha, email, privilégios, configurações) sejam executadas por `requisições GET`, que vale ressaltar, não exigem `token de proteção`. 

Como é mostrado no curso, o navegador sempre envia `cookies de sessão` quando carrega imagens, iframes, links, etc., assim é possível enviar o ataque forçando a vítima a acessar um `url` que, sem o uso do `token`, executa uma ação automática.

Além disso é possível injetar `formulários HTML` falsos onde não há token, já que o navegador envia automáticamente os `cookies da sessão` e a aplicação confia exclusivamente em cookies.

#### O problema dos Cookies

Primeiramente, vamos explicar o que é um `SameSite`.

- `SameSite:` Basicamente é um atributo de cookies que controla quando o navegador deve ou não enviar um cookie em requisições vindas de outros sites. Existem três diferentes tipos desse atributo, são eles: `Lax, Strict, None`.

- `Lax:` funcionam como uma proteção moderada. Eles permitem que o navegador envie o cookie quando o usuário acessa o site diretamente ou em requisições seguras como GET, HEAD e OPTIONS. Contudo, eles bloqueiam o envio do cookie em requisições POST vindas de outros domínios, ajudando a reduzir ataques CSRF. Ainda assim, cookies continuam sendo enviados em GETs iniciados por sites externos, o que pode ser arriscado se informações sensíveis estiverem guardadas neles.

- `Strict:` funcionam como uma proteção rígida, pois só permitem que o cookie seja enviado quando a requisição vem do próprio site que o definiu. Isso impede completamente que cookies sejam enviados em contextos externos, bloqueando de forma eficaz ataques de CSRF. Ao manter o isolamento total entre origens diferentes, esse modo oferece o nível mais alto de segurança, sendo especialmente útil quando a aplicação lida com dados sensíveis do usuário.

- `None:` são totalmente permissivos, pois são enviados tanto em requisições do próprio site quanto de outros domínios, sendo úteis quando é necessário compartilhar cookies entre diferentes origens. Contudo, para reduzir os riscos de segurança desse comportamento aberto, eles exigem o atributo Secure em conexões HTTPS, garantindo que o cookie só seja transmitido por canais seguros e diminuindo a chance de interceptação ou alteração maliciosa durante o tráfego.

Com isso, devemos imaginar que a falta do SameSite correto gera um buraco de vulnerabilidades envolvendo Cookies.

#### Falta de validação de Origem

Mesmo requisições `POST` podem ser falsificadas, já que muitas aplicações não fazem verificação de cabeçalhos `Origin ou Referer`, desta maneira, o navegador inclui cookies independentemente do domínio onde o formulário está hospedado. Ao não validar a origem, o servidor não tem como distinguir se a ação foi iniciada pelo próprio site ou por um atacante.

Além disso, é comum que embora o navegador aplique a `Same-Origin Policy (SOP)` ou o `CORS (Cross-Origin Resource Sharing)` para restringir requisições entre origens diferentes, isso não impede que o navegador envie cookies de sessão quando um atacante faz o usuário executar um script malicioso. Assim, um atacante pode criar um código JavaScript que dispara silenciosamente um `XMLHttpRequest` ou `fetch()` para um endpoint onde a vítima está autenticada, fazendo com que ações críticas sejam realizadas sem o usuário perceber.


FALTOU OS METODOS DE DEFESA E A CONCLUSAO MINHA LINDA
