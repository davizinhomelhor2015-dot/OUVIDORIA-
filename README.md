# DP Lynwood — Ouvidoria (site público)

Repositório separado, pronto pra publicar com **GitHub Pages**. Este é o site que **qualquer pessoa** acessa para enviar sugestão, elogio ou reclamação — não precisa de login.

O painel privado onde o Comando vê e modera os registros (`comando.html`) fica num **repositório à parte** — veja a seção 4.

| Arquivo | O que é |
|---|---|
| `index.html` | O site da Ouvidoria em si (nomeado `index.html` pra virar a página raiz no GitHub Pages). |
| `database.rules.json` | Trecho de regras do Realtime Database para colar nas suas regras já existentes (ver seção 2). |

Usa o mesmo projeto Firebase que você já tinha (`comando-geral-98c87`).

---

## 1. Subir para o GitHub e ativar o Pages

1. Crie um repositório novo no GitHub, por exemplo `ouvidoria-lynwood` (pode ser público — a chave do Firebase que aparece no código **não é secreta**, veja o aviso abaixo).
2. Suba `index.html` e `database.rules.json` para a raiz do repositório:
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

> **Sobre a chave do Firebase aparecer no código:** isso é normal e esperado — a `apiKey` do Firebase não funciona como senha, ela só identifica o projeto. Quem realmente protege os dados são as **regras do Realtime Database** (seção 2) e o **login do Comando** (que fica no outro repositório). Sem essas duas coisas configuradas, um repositório público não é seguro; com elas, é.

## 2. Publicar as regras do Realtime Database (importante — leia com atenção)

O arquivo `database.rules.json` traz **só os blocos novos** (`suggestions` e `banned`). Ele **não é para publicar sozinho** — se você substituir todas as suas regras só por esse arquivo, qualquer outro dado que já exista no seu banco fica inacessível. (Essa etapa só precisa ser feita **uma vez** — não repita ao configurar o repositório do Painel do Comando, é a mesma regra pros dois.)

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) → projeto `comando-geral-98c87` → **Compilação → Realtime Database → Regras**.
2. Copie o conteúdo de `database.rules.json` e **cole dentro** do seu JSON de regras atual, dentro de `"rules": { ... }`, ao lado do que já existe — não apague nada que já estava lá. Fica assim, por exemplo:

```json
{
  "rules": {

    /* ↓↓↓ o que você já tinha continua aqui, sem mudar ↓↓↓ */

    /* ↑↑↑ até aqui ↑↑↑ */

    "suggestions": {
      ".read": "auth != null",
      "$id": {
        ".write": "auth != null || !data.exists()",
        ".validate": "newData.hasChildren(['name','message']) && newData.child('name').isString() && newData.child('name').val().length > 0 && newData.child('name').val().length <= 60 && newData.child('message').isString() && newData.child('message').val().length > 0 && newData.child('message').val().length <= 1000"
      }
    },

    "banned": {
      ".read": true,
      ".write": "auth != null",
      ".indexOn": ["value"]
    }
  }
}
```

> (Remova os comentários `/* ... */` — JSON de verdade não aceita comentários; eles estão aqui só pra indicar onde entra o que já existia.)

3. Clique em **Publicar**.

**Por que assim, e não `".write": true` simples:** com essa regra, qualquer pessoa pode *criar* um novo registro (obrigatoriamente com nome e mensagem válidos), mas **não pode alterar nem apagar** um registro que já existe — só o Comando (logado) pode. Isso fecha uma brecha real: sem essa restrição, alguém poderia sobrescrever ou apagar as sugestões dos outros só sabendo o link.

## 3. Testar

1. Abra a URL do GitHub Pages, envie um registro de teste com nome e mensagem.
2. Digite um palavrão no campo de mensagem — deve aparecer o aviso âmbar em tempo real; o envio continua liberado mesmo assim.
3. Depois, confira no Painel do Comando (outro repositório) se o registro apareceu, com o selo **⚠ Linguagem imprópria** se aplicável.

## 4. Sobre o repositório do Painel do Comando

Este repositório só tem a Ouvidoria. O Painel do Comando (`comando.html` → login, lista de registros, banimento, troca de senha) fica em um **segundo repositório separado**, com seu próprio README explicando como configurar o login e o restante. Depois de publicar os dois, você pode linkar um no outro (o painel já tem um espaço reservado pra isso — veja o README dele).

---

## O que foi corrigido nesta versão

- **Falso positivo no filtro de palavrões:** a palavra "cu" estava sendo detectada dentro de palavras comuns como "escuro" (por causa de uma checagem simples de substring). Agora o filtro reconhece limites de palavra, então só marca quando a palavra ofensiva aparece de fato — testado com mais de 15 casos (incluindo variações como "arrombado"/"arrombada" e frases como "filho da puta") sem falso positivo nem falso negativo.
- **Brecha de segurança nas regras do banco:** antes, a regra pública de escrita permitia não só criar, mas também sobrescrever/apagar sugestões de outras pessoas. Agora é "somente criação" pública — editar ou apagar exige login do Comando.

## Como funciona o filtro de linguagem imprópria

Fica na constante `PALAVRAS_PROIBIDAS` dentro do `index.html` — edite livremente. Ignora acentos, maiúsculas/minúsculas e disfarces simples (`p0rr4`), e respeita limites de palavra para não confundir com palavras comuns. **Nada é bloqueado por causa disso** — apenas alerta quem escreve e chega marcado no painel para o Comando avaliar.

## Sobre o banimento de IP/aparelho (leitura importante)

- **IP**: obtido via serviço público (ipify) no navegador de quem envia. Bloqueia bem a mesma rede/pessoa insistindo, mas troca de rede/VPN contorna.
- **Aparelho**: identificador salvo no navegador (`localStorage`). Some se a pessoa limpar os dados do navegador ou usar outro navegador/aba anônima.
- Por isso o painel guarda também o **nome digitado** — cruzando nome + IP + aparelho dá pra identificar reincidentes na prática, mesmo sem bloqueio 100% à prova de burla (isso exigiria servidor próprio).

## Estrutura dos dados (Realtime Database)

```
/suggestions/{id}: name, message, hasProfanity, profanityWords, ip, deviceId, fingerprint, userAgent, status, createdAt
/banned/{id}: type ("ip" | "device"), value, reason, bannedAt
```

Qualquer ajuste, é só chamar.
