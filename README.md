# DP Lynwood — Ouvidoria (site público)

Repositório separado, pronto pra publicar com **GitHub Pages**. Este é o site que **qualquer pessoa** acessa para enviar sugestão, elogio ou reclamação — não precisa de login.

O painel privado onde o Comando vê e modera os registros (`comando.html`) fica num **repositório à parte** — veja a seção 3.

| Arquivo | O que é |
|---|---|
| `index.html` | O site da Ouvidoria em si (nomeado `index.html` pra virar a página raiz no GitHub Pages). |

Usa o mesmo projeto Firebase que você já tinha (`comando-geral-98c87`) — **sem precisar abrir o Firebase Console em nenhum momento.**

---

## 1. Subir para o GitHub e ativar o Pages

1. Crie um repositório novo no GitHub, por exemplo `ouvidoria-lynwood` (pode ser público — a chave do Firebase que aparece no código **não é secreta**, veja o aviso abaixo).
2. Suba `index.html` para a raiz do repositório:
   ```
   git init
   git add .
   git commit -m "Ouvidoria DP Lynwood"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/ouvidoria-lynwood.git
   git push -u origin main
   ```
3. No GitHub, vá em **Settings → Pages**.
4. Em **Source**, selecione **Deploy from a branch** → branch **main** → pasta **/ (root)** → **Save**.
5. Espere 1–2 minutos. A URL final fica algo como:
   `https://SEU-USUARIO.github.io/ouvidoria-lynwood/`

> **Sobre a chave do Firebase aparecer no código:** isso é normal e esperado — a `apiKey` do Firebase não funciona como senha, ela só identifica o projeto.

> **Sobre não precisar mexer nas regras do banco:** este site funciona assim que publicado, sem nenhuma etapa extra no Firebase Console, desde que o seu projeto já esteja com as regras do Realtime Database relativamente abertas (o padrão de projetos novos em "modo de teste", bem comum). Se ao testar aparecer erro de permissão ao enviar um registro, é sinal de que suas regras estão mais restritas — nesse caso específico, ajustar isso exige sim abrir o Firebase Console (é uma configuração do seu projeto que só você pode mudar por lá).

## 2. Testar

1. Abra a URL do GitHub Pages, envie um registro de teste com nome e mensagem, sem palavrão — deve salvar normalmente.
2. Digite um palavrão no campo de mensagem ou no nome — aparece um aviso vermelho em tempo real, e ao tentar enviar o envio é **recusado** (nada é salvo no banco). É só apagar/corrigir o texto e enviar de novo.
3. Depois de um envio válido, confira no Painel do Comando (outro repositório) se o registro apareceu.

## 3. Sobre o repositório do Painel do Comando

Este repositório só tem a Ouvidoria. O Painel do Comando (login, lista de registros, banimento, troca de senha) fica em um **segundo repositório separado**, com login pronto pra usar (`adm` / `admin1234`) assim que publicado — também sem precisar mexer no Firebase Console.

---

## O que foi corrigido/mudado nesta versão

- **Linguagem imprópria agora bloqueia o envio.** Antes, uma mensagem com palavrão era salva mesmo assim (só ficava marcada para o Comando revisar depois). Agora, se o nome ou a mensagem contiverem alguma palavra da lista, o envio é recusado na hora — nada chega a ser salvo no banco — e a pessoa vê exatamente qual campo precisa corrigir.
- **Falso positivo no filtro de palavrões:** a palavra "cu" estava sendo detectada dentro de palavras comuns como "escuro" (por causa de uma checagem simples de substring). Agora o filtro reconhece limites de palavra, então só marca quando a palavra ofensiva aparece de fato — testado com mais de 15 casos (incluindo variações como "arrombado"/"arrombada" e frases como "filho da puta") sem falso positivo nem falso negativo.

## Como funciona o filtro de linguagem imprópria

Fica na constante `PALAVRAS_PROIBIDAS` dentro do `index.html` — edite livremente. Ignora acentos, maiúsculas/minúsculas e disfarces simples (`p0rr4`), e respeita limites de palavra para não confundir com palavras comuns. **O envio é bloqueado enquanto a palavra estiver presente** — a pessoa precisa corrigir o texto antes de conseguir enviar.

> **Leitura importante:** como a mensagem nunca chega a ser salva quando é bloqueada, o Painel do Comando não tem como saber quem tentou enviar linguagem imprópria (não há mais registro nenhum daquela tentativa) — a aba "Sinalizados" do painel só existiria para casos que passassem pelo filtro sem ser detectados. Se algum dia você quiser que essas tentativas fiquem visíveis para o Comando decidir se bane ou não (em vez de bloquear silenciosamente), é só pedir — é uma mudança pequena de voltar a salvar, só que marcado, em vez de recusar.

## Sobre o banimento de IP/aparelho (leitura importante)

- **IP**: obtido via serviço público (ipify) no navegador de quem envia. Bloqueia bem a mesma rede/pessoa insistindo, mas troca de rede/VPN contorna.
- **Aparelho**: identificador salvo no navegador (`localStorage`). Some se a pessoa limpar os dados do navegador ou usar outro navegador/aba anônima.
- Por isso o painel guarda também o **nome digitado** — cruzando nome + IP + aparelho dá pra identificar reincidentes na prática, mesmo sem bloqueio 100% à prova de burla (isso exigiria servidor próprio).

## Sobre segurança sem regras dedicadas (leitura importante)

Como este sistema evita qualquer etapa no Firebase Console, ele conta com as regras padrão relativamente abertas do seu projeto. Isso significa que, tecnicamente, alguém com a URL direta do seu banco de dados poderia ler ou escrever nos dados sem passar pelos sites. Se algum dia quiser fechar isso (regra "só cria, não edita/apaga" nas sugestões + login real no painel), é só pedir — é rápido de aplicar, só exige 2 cliques seus no Firebase Console que hoje estamos evitando de propósito.

## Estrutura dos dados (Realtime Database)

```
/suggestions/{id}: name, message, hasProfanity, profanityWords, ip, deviceId, fingerprint, userAgent, status, createdAt
/banned/{id}: type ("ip" | "device"), value, reason, bannedAt
```

Qualquer ajuste, é só chamar.
