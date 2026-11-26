# PAQ Jobs

Projeto de mentoria da ONG Prototipando A Quebrada (PAQ) para ajudar jovens a encontrar vagas de estágio e jovem aprendiz. A interface permite buscar vagas, abrir seus detalhes e conversar com um assistente de carreira. A API consulta as vagas no MongoDB e usa a Groq para gerar as respostas do chat.

O back-end usa Node.js, Express e Mongoose. O front-end usa React, Vite, React Router e Tailwind CSS.

## Rodar o projeto localmente

Os comandos abaixo usam PowerShell. Comece na raiz do repositório, onde fica o `package.json` do back-end. O projeto foi verificado com Node.js 24 e npm 11.

### 1. Preparar o MongoDB

Use uma instância do MongoDB com um usuário autorizado a ler e gravar dados no banco do projeto. Se precisar criar uma instância local, abra o Docker Desktop e execute o comando abaixo uma vez. Ele cria um banco de desenvolvimento acessível pela máquina local e mantém os dados em um volume. Para religar o mesmo contêiner em outra sessão, rode `docker start paqjobs-mongo`. [Imagem oficial do MongoDB](https://hub.docker.com/_/mongo)

```powershell
docker run -d --name paqjobs-mongo -p 127.0.0.1:27017:27017 -e MONGO_INITDB_ROOT_USERNAME=paqjobs -e MONGO_INITDB_ROOT_PASSWORD=paqjobs-local -v paqjobs-mongo-data:/data/db mongo:8.0
```

### 2. Configurar o back-end

Crie um arquivo chamado `.env` na raiz, ao lado do `package.json`. Os valores de banco abaixo correspondem ao contêiner local. Para usar outro banco, preencha com os valores da sua instância. Substitua o valor de `KEY_GROQ` pela sua chave e mantenha o arquivo `.env` apenas no seu ambiente. [Guia de configuração da Groq](https://console.groq.com/docs/quickstart)

```dotenv
PORT=3000
DB_USER=paqjobs
DB_PASS=paqjobs-local
DB_HOSTS=127.0.0.1:27017
DB_NAME=paqjobs
DB_OPTIONS=authSource=admin
KEY_GROQ=COLE_SUA_CHAVE_AQUI
```

O back-end monta uma URI no formato `mongodb://`. Em `DB_HOSTS`, informe o host com a porta ou a lista de hosts do cluster. Em `DB_OPTIONS`, informe as opções que iriam depois do `?`. Para usuários e senhas com caracteres reservados na URI, use codificação percentual. [Opções de conexão do MongoDB](https://www.mongodb.com/docs/manual/reference/connection-string-options/)

| Variável | Quando configurar |
| --- | --- |
| `KEY_GROQ` | O SDK lê essa chave ao iniciar a API. Para o chat real, a chave precisa ter acesso ao modelo `llama-3.3-70b-versatile`, listado pela Groq como Enterprise. [Modelos da Groq](https://console.groq.com/docs/models) |
| `URL_ADZUNA` | URL completa da consulta à Adzuna. Preencha se for usar `GET /vaga/adzuna`. |
| `URL_JOOBLE` | URL completa do endpoint do Jooble. Preencha se for usar `GET /vaga/jooble`. |

### 3. Iniciar a API

No primeiro terminal, rode os comandos abaixo na raiz do repositório.

```powershell
npm ci
npm run dev
```

A API usa `http://127.0.0.1:3000` por padrão. Aguarde a mensagem `Conectado com sucesso ao MongoDB!` no terminal. O comando `npm start` sobe o servidor sem o monitoramento de arquivos do Nodemon.

### 4. Iniciar o front-end

Abra um segundo terminal na raiz do repositório e rode os comandos abaixo.

```powershell
cd frontend
npm ci
npm run dev
```

Abra o endereço mostrado pelo Vite, normalmente `http://localhost:5173`. O navegador chama `/api` no Vite, que encaminha a requisição para o Express e remove o prefixo. Por exemplo, `/api/vaga/` chega ao Express como `/vaga/`. Se alterar `PORT` no back-end, informe a URL correspondente em `API_PROXY_TARGET` no arquivo `frontend/.env.local`. [Exemplo de configuração do front-end](frontend/.env.example)

### 5. Conferir a integração

Em um terceiro terminal, consulte a API. Um banco recém-criado retorna uma lista vazia e a interface informa que nenhuma vaga foi encontrada.

```powershell
(Invoke-WebRequest -UseBasicParsing -Uri 'http://127.0.0.1:3000/vaga/').Content
```

Em um banco local de teste, cadastre esta vaga fictícia para conferir o cartão e a página de detalhes na interface.

```powershell
$vaga = @{
    _id = 1001
    id_vaga_external = 1001
    name = 'Estagio em Desenvolvimento Web'
    title = 'Estagio em Desenvolvimento Web'
    description = 'Vaga ficticia para teste local.'
    company = 'Empresa Exemplo'
    location = 'Florianopolis'
    redirect_url = 'https://example.com/paqjobs-vaga-1001'
} | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:3000/vaga/' -ContentType 'application/json' -Body $vaga
Invoke-RestMethod -Uri 'http://127.0.0.1:3000/vaga/1001'
```

Com uma chave válida da Groq, abra o chat na interface ou teste a resposta pelo PowerShell.

```powershell
$mensagem = @{ input = 'Ola! Como voce pode me ajudar?' } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:3000/chat/converse' -ContentType 'application/json' -Body $mensagem
```

## Rotas da API

| Método e caminho | O que faz |
| --- | --- |
| `GET /vaga/` | Lista vagas do MongoDB. Aceita `?search=termo` ou `?q=termo` para buscar por título, nome ou descrição. |
| `POST /vaga/` | Cria ou atualiza uma vaga pelo campo `id_vaga_external`. |
| `GET /vaga/:id` | Busca uma vaga pelo identificador do banco ou pelo identificador externo. |
| `GET /vaga/adzuna` | Consulta a Adzuna e grava os resultados no MongoDB. |
| `GET /vaga/jooble` | Consulta o Jooble e devolve os resultados na resposta HTTP. |
| `POST /chat/converse` | Envia a mensagem do corpo JSON no campo `input` ou `mensagem`. A API devolve o texto em `resposta`. |
| `GET /chat/converse` | Aciona o mesmo controlador, com uma saudação padrão quando a mensagem é omitida. |
| `GET /user/` e `GET /user/:id` | Listam pessoas ou buscam uma pessoa pelo ID, respectivamente. |
| `POST /user/`, `PUT /user/:id` e `DELETE /user/:id` | Criam, atualizam e removem pessoas, respectivamente. |

A lista exibida na interface vem do MongoDB. A sincronização com a Adzuna ocorre ao chamar a rota correspondente. A tela de login é um protótipo visual. Use dados fictícios nas rotas de pessoas durante o desenvolvimento, pois elas aceitam requisições sem autenticação e o cadastro grava a senha em texto puro.

## Estrutura dos arquivos

### Raiz e back-end

| Arquivo | Responsabilidade |
| --- | --- |
| `README.md` | Guia do projeto completo. |
| `package.json` e `package-lock.json` | Declaram os comandos e as dependências da API e registram as versões usadas pelo `npm ci`. |
| `.gitignore` | Exclui `node_modules/` e `.env` do Git. |
| `src/index.js` | Carrega o ambiente, configura Express e CORS, registra as rotas e inicia a conexão com o MongoDB. |
| `src/routers/index.js` | Agrupa as rotas sob os prefixos `/vaga`, `/user` e `/chat`. |
| `src/routers/vaga.router.js` | Mapeia as rotas de vagas para os controladores. |
| `src/routers/pessoa.router.js` | Mapeia as operações de cadastro e consulta de pessoas. |
| `src/routers/chatbot.router.js` | Mapeia os métodos GET e POST da conversa. |
| `src/db/vaga.schema.js` | Define os campos das vagas no MongoDB. |
| `src/db/pessoa.schema.js` | Define os campos das pessoas no MongoDB. |
| `src/controllers/vagas/buscarVagas.js` | Lista as vagas e aplica o filtro de busca. |
| `src/controllers/vagas/buscarVagasPorId.js` | Busca os detalhes de uma vaga pelo ID. |
| `src/controllers/vagas/criarVagas.js` | Recebe o JSON de uma vaga e chama o serviço de persistência. |
| `src/controllers/vagas/buscarVagasAdzuna.js` | Consulta a Adzuna e adapta seus campos antes de gravar as vagas. |
| `src/controllers/vagas/buscarVagasJooble.js` | Consulta o Jooble com filtros de cargo e localização. |
| `src/controllers/pessoas/getPessoas.js` | Lista as pessoas cadastradas. |
| `src/controllers/pessoas/getPessoaPorId.js` | Busca uma pessoa pelo ID. |
| `src/controllers/pessoas/criarPessoa.js` | Valida os campos obrigatórios e cadastra uma pessoa. |
| `src/controllers/pessoas/atualizarPessoa.js` | Atualiza os campos de uma pessoa existente. |
| `src/controllers/pessoas/deletarPessoa.js` | Remove uma pessoa pelo ID. |
| `src/controllers/chatbot.controller.js` | Envia a conversa à Groq, consulta vagas quando o modelo solicita a ferramenta e formata a resposta. |
| `src/services/vaga.service.js` | Centraliza a criação ou atualização de vagas no MongoDB. |
| `src/services/tools.service.js` | Define a ferramenta de busca de vagas apresentada ao modelo da Groq. |

### Front-end

Os caminhos abaixo são relativos à pasta `frontend/`.

| Arquivo | Responsabilidade |
| --- | --- |
| `README.md` e `LICENSE` | Documentação da interface, referência ao repositório de origem e licença MIT do front-end. |
| `package.json` e `package-lock.json` | Comandos, dependências e versões da interface. |
| `.gitignore` e `.env.example` | Regras para ignorar dependências, builds e configurações locais, além do exemplo das variáveis do Vite. |
| `index.html` | Documento HTML que fornece o elemento onde o React monta a interface. |
| `vite.config.js` | Configura React, Tailwind e o proxy local da API. |
| `eslint.config.js` | Configura as regras de análise estática para JavaScript, JSX e hooks do React. |
| `netlify.toml` | Faz a Netlify servir o HTML da interface ao abrir uma rota interna diretamente. |
| `src/main.jsx` e `src/index.css` | Inicializam o React, o roteador e o CSS do Tailwind. |
| `src/App.jsx` | Compõe o layout compartilhado, com cabeçalho, conteúdo, rodapé e chat. |
| `src/routes/routes.jsx` | Define as páginas `/`, `/vagas/:id` e `/login`. |
| `src/pages/Vagas.jsx` | Monta a página inicial e controla a listagem, a pesquisa e a paginação. |
| `src/pages/Login.jsx` | Implementa o protótipo visual de entrada e cadastro. |
| `src/pages/JobDetail/index.jsx` | Carrega e exibe os dados de uma vaga. |
| `src/pages/JobDetail/JobNaoEncontrada.jsx` | Mostra a tela de vaga indisponível. |
| `src/pages/JobDetail/StatusVaga.jsx` | Renderiza os selos informativos usados nos detalhes da vaga. |
| `src/components/Header.jsx` e `src/components/Footer.jsx` | Exibem a navegação e o rodapé compartilhados. |
| `src/components/JobCard.jsx`, `src/components/SearchInput.jsx` e `src/components/Pagination.jsx` | Renderizam o cartão de uma vaga, o campo de busca e os controles de página, respectivamente. |
| `src/components/home/Hero.jsx` e `src/components/home/HowItWorks.jsx` | Compõem o destaque inicial e a seção sobre o funcionamento do PAQ Jobs. |
| `src/container/ChatSidebar.jsx` e `src/container/ChatSidebar.css` | Controlam o painel do chat, as mensagens e sua formatação. |
| `src/services/vagasApi.js` | Envia as requisições HTTP de vagas e do chat. |
| `src/store/apiRoutes.js` | Centraliza a URL base e os caminhos da API usados pela interface. |
| `public/PaqJobsLogo.png`, `public/favicon.svg` e `public/icons.svg` | Arquivos estáticos de marca e ícones. O PNG é usado no cabeçalho, no login e como ícone da aba. |
| `src/assets/hero.png`, `src/assets/react.svg` e `src/assets/vite.svg` | Imagem e ícones preservados do repositório de origem, atualmente sem referência na interface. |

## Build do front-end

Na pasta `frontend/`, rode `npm run lint` para conferir o código e `npm run build` para gerar `dist/`. Use `npm run preview` para abrir o build localmente. Quando a interface e a API estiverem em domínios diferentes, defina `VITE_API_BASE_URL` com a URL pública da API antes de gerar o build.

## Créditos

Projeto desenvolvido durante a mentoria da ONG Prototipando A Quabrada. [Repositório de origem do front-end](https://github.com/JoabeSCosta/front-end-PAQ) · [Licença do front-end](frontend/LICENSE)
