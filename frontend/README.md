# Front-end do PAQ Jobs

Interface React do projeto de mentoria da ONG Prototipando A Quabrada (PAQ). [Repositório de origem](https://github.com/JoabeSCosta/front-end-PAQ/tree/2833c4fee245bce1e0e97c6ff5001ab9000ac2dd)

## Rodar localmente

Na raiz do repositório, configure o `.env` do back-end. A conexão com o MongoDB usa `DB_USER`, `DB_PASS`, `DB_HOSTS`, `DB_NAME` e `DB_OPTIONS`. O chatbot usa `KEY_GROQ`. Rode `npm ci` e `npm run dev` para iniciar a API.

Em outro terminal, a partir da raiz, rode os comandos abaixo.

```powershell
cd frontend
npm ci
npm run dev
```

Abra o endereço indicado pelo Vite, normalmente `http://localhost:5173`. O Vite encaminha as chamadas de `/api` para `http://127.0.0.1:3000`. Se a API usar outra porta, defina `API_PROXY_TARGET` em `frontend/.env.local` conforme o [exemplo de configuração](.env.example).

## Build

Rode `npm run build` dentro de `frontend/` para gerar `dist/`. Para hospedar a interface e a API em domínios separados, defina `VITE_API_BASE_URL` com a URL pública da API antes do build.
