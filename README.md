# 🤖 IA Qwik: Chatbot de Autoajuda com 3 Modos

Trabalho prático de **Tópicos (2026)**: *"Frameworks: Do Vanilla ao Modernismo"*.
Framework do grupo: **[Qwik](https://qwik.dev/docs/)**.

O usuário digita uma **desculpa real** (ex.: "Tô com preguiça de estudar") e uma IA responde conforme o modo escolhido. Cada modo muda a personalidade da IA **e a interface inteira**.

## 🎭 Os 3 modos

| Modo | Personalidade | Visual |
|---|---|---|
| 😡 **Agressivo** | Bronca curta, ácida e bem-humorada, em CAIXA ALTA (sem palavrões) | Vermelho fogo, fonte Impact, balões que tremem |
| 🧘 **Calmo** | Voz serena, respiração guiada, imagens da natureza | Verde e azul suaves, tudo em câmera lenta |
| 😈 **Sexy** | Flerte e duplo sentido, só insinuação (nada explícito) | Vinho, rosa e dourado, brilho que pulsa |

Cada modo mantém a **sua própria conversa**: trocar de modo não reenvia a saudação, e ao voltar o histórico continua lá.

## 🧰 Tecnologias

- **Qwik + Qwik City** 1.x (resumabilidade, `server$`, signals e stores)
- **Bootstrap 5.3** (cards, badges, btn-group, input-group, spinner e alert) com temas por variáveis CSS
- **Google Gemini API** (chamada `POST` com `async/await`)
- **Vite 7** e **TypeScript**

## ✅ Requisitos

| Item | Detalhe |
|---|---|
| **Node.js** | **20.19 ou mais novo, ou 22.12 ou mais novo** (recomendado: versão LTS 22 ou 24). O Node 18 **não** funciona com o Vite 7 |
| **npm** | Já vem com o Node |
| **Git** | Para clonar o repositório |
| **Navegador** | Chrome, Edge ou Firefox |
| **Chave da API do Gemini** | Grátis em [aistudio.google.com](https://aistudio.google.com) (*Get API key*) |
| **Internet** | Precisa acessar `generativelanguage.googleapis.com` |

> Não precisa instalar WAMP, Apache, PHP nem XAMPP. O Qwik tem o próprio servidor de desenvolvimento.

Confira o Node no terminal:

```bash
node -v
npm -v
```

## 🚀 Instalação passo a passo

**1. Clone o repositório** (de preferência numa pasta sem espaços no caminho):

```bash
git clone https://github.com/danielraitersg/IA-Qwik-Topicos-2026.git
cd IA-Qwik-Topicos-2026
```

**2. Instale as dependências:**

```bash
npm install
```

**3. Crie o arquivo `.env.local`** na raiz do projeto (ao lado do `package.json`), copiando o modelo:

```bash
cp .env.example .env.local
```

No PowerShell use `Copy-Item .env.example .env.local`. Depois abra o `.env.local` e coloque a sua chave, sem aspas e sem espaços:

```
GEMINI_API_KEY=cole_sua_chave_aqui
```

**4. Rode o projeto:**

```bash
npm run dev
```

**5. Abra no navegador:** <http://localhost:5173/>

> ⚠️ O `.env.local` só é lido quando o servidor inicia. Se você mudar a chave, pare o servidor (`Ctrl+C`) e rode `npm run dev` de novo.

## 🏫 Instalação em computador com restrições (laboratório)

Em alguns computadores da escola o `cmd.exe` é bloqueado pelo administrador, e o npm usa o `cmd.exe` para rodar scripts. Se aparecer *"O prompt de comando foi desativado pelo administrador"*:

**1.** Mande o npm usar o PowerShell (só nesta janela):

```powershell
$env:npm_config_script_shell = "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
```

Para gravar a configuração (pode ser perdida se o perfil do computador for limpo):

```powershell
npm config set script-shell "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"
```

**2.** Instale pulando os scripts de instalação (o pacote `sharp` usa `||`, que o PowerShell antigo não entende):

```powershell
npm install --ignore-scripts
```

**3.** Rode normalmente com `npm run dev`.

**Sem permissão para instalar programas?** Baixe o Node em [nodejs.org](https://nodejs.org/en/download) na versão *Windows Binary (.zip)*, extraia numa pasta sua e, em cada PowerShell novo, rode:

```powershell
$env:Path = "C:\caminho\da\pasta\node;" + $env:Path
node -v
```

**Se o `npm run dev` não achar o `vite`:** confira se você está dentro da pasta que tem o `package.json`. Se mesmo assim falhar, rode direto:

```powershell
node .\node_modules\vite\bin\vite.js --mode ssr
```

## 🧠 Como funciona

```
Navegador                      Servidor (Qwik City)                 Google
┌──────────────┐  server$()   ┌─────────────────────┐   POST      ┌────────────┐
│ index.tsx    │ ───────────► │ pedirBronca()       │ ──────────► │ Gemini API │
│ (UI + modos) │ ◄─────────── │ lê GEMINI_API_KEY   │ ◄────────── │            │
└──────────────┘   resposta   └─────────────────────┘   resposta  └────────────┘
```

- **`server$`:** a função `pedirBronca` parece uma função comum, mas **roda só no servidor**. Por isso a chave da API nunca chega ao navegador.
- **Tentativas e modelos reserva:** se um modelo estiver sobrecarregado (`429` ou `5xx`), o código espera e tenta de novo, e depois passa para o próximo modelo da lista `MODELOS`.
- **Bloqueio de segurança do Google:** se a resposta vier bloqueada, ele tenta de novo com temperatura menor. Se tudo falhar, responde com uma **frase reserva** do personagem, e o chat nunca fica mudo.
- **Um prompt por modo:** o modo ativo é enviado junto com a mensagem, e o servidor escolhe o prompt de sistema correspondente.
- **Temas:** o elemento raiz recebe `data-modo="agressivo | calmo | sexy"` e o CSS troca as variáveis de cor, fonte, bordas e animações.
- **Resumabilidade do Qwik:** a página chega do servidor já pronta e o JavaScript só é baixado quando o usuário interage (dá para ver na aba *Network* do navegador).

### Estrutura

```
src/
├── routes/
│   └── index.tsx      # Chatbot completo: modos, temas, server$ e interface
├── root.tsx           # Raiz do app (importa o global.css)
├── global.css         # Importa o Bootstrap 5
└── components/
    └── router-head/   # <head> da página
.env.example           # Modelo do arquivo de variáveis (sem chave real)
```

## 📋 Requisitos do trabalho × onde estão

| Requisito | Onde |
|---|---|
| Consumo de API de IA via `POST` com `async/await` | `pedirBronca` (`server$`) em `src/routes/index.tsx` |
| Balões de mensagem renderizados dinamicamente | `conversa.map(...)` no JSX |
| Auto-scroll | `useVisibleTask$` com `track` |
| Controle visual de loading | `spinner-border` e campos desabilitados |
| Tratamento de exceções | `try/catch/finally`, tentativas, modelos reserva e alerta `alert-danger` |
| Estilização com **Bootstrap 5** | Classes `card`, `badge`, `btn-group`, `input-group`, `spinner-border`, `alert` |

## 🧾 Scripts

| Comando | O que faz |
|---|---|
| `npm run dev` | Servidor de desenvolvimento (<http://localhost:5173/>) |
| `npm run build` | Gera a versão de produção |
| `npm run build.types` | Confere os tipos do TypeScript |
| `npm run lint` | Roda o ESLint |

## 🩺 Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| `Missing script: "dev"` ou `package.json` não encontrado | Terminal fora da pasta do projeto | `cd` até a pasta que contém o `package.json` |
| `GEMINI_API_KEY não encontrada` no terminal | `.env.local` ausente, com nome errado ou servidor não reiniciado | Confira o arquivo na raiz e reinicie o `npm run dev` |
| `Erro da API: 400` ou `403` | Chave inválida | Gere outra em aistudio.google.com |
| `Erro da API: 404` | O Google aposentou o modelo | Atualize a lista `MODELOS` em `src/routes/index.tsx` com os nomes atuais ([lista de modelos](https://ai.google.dev/gemini-api/docs/models)) |
| `Erro da API: 429` ou `503` | Limite gratuito ou sobrecarga temporária | Espere um minuto. O código já tenta de novo sozinho |
| `usando resposta reserva` no terminal | O Google bloqueou a resposta por segurança | Normal de vez em quando; o chat segue com uma frase pronta |
| `fetch failed` | A rede bloqueia o Google | Use outra rede ou o hotspot do celular |
| `EPERM` ou `ENOTEMPTY` no `npm install` | Antivírus ou outro programa travando os arquivos | Mova o projeto para fora de pastas como `C:\wamp64` e tente de novo |
| Página sem estilo | Bootstrap não importado | Confira se `src/global.css` começa com `@import "bootstrap/dist/css/bootstrap.min.css";` |

## 🔐 Segurança

- **Nunca** suba o `.env.local` nem cole a chave da API em mensagens, prints ou commits. O `.gitignore` já ignora arquivos `*.local`.
- Se a chave vazar, apague-a em aistudio.google.com e gere outra.
- O repositório traz só o `.env.example`, com um valor de mentira.

## ⚠️ Aviso

Projeto **educativo**, feito para uma demonstração em sala de aula. Os modos usam humor e exagero de propósito. O modo "Sexy" fica só na insinuação (nada explícito) e corta o flerte se a pessoa disser que é menor de 18 anos. Em todos os modos, se alguém escrever sobre sofrimento real, a IA sai do personagem e sugere procurar alguém de confiança ou ligar para o **CVV (188, gratuito, 24 horas)**.

## 🚢 Publicar na internet (opcional)

1. Adicione o adaptador: `npm run qwik add vercel-edge` (ou `netlify-edge`).
2. Envie o projeto para o GitHub.
3. Na [Vercel](https://vercel.com), importe o repositório e cadastre a variável de ambiente `GEMINI_API_KEY` **antes** do deploy.

## 👥 Integrantes

- *(preencher com os nomes do grupo)*
