# Guia da Sparkles App Store (UmbrelOS)

Guia completo para usar, manter e atualizar esta App Store personalizada no
UmbrelOS, e para publicar novas versões do aplicativo **TODOapp**.

- **ID da store:** `sparkles`
- **Nome na UI:** *Sparkles App Store*
- **URL da store (GitHub):** `https://github.com/And5reas/my-umbrel-app-store`
- **Apps publicados:** `sparkles-9router`, `sparkles-balanca-backend`, `sparkles-todoapp`

> Regra do template: todo app desta store deve ter o ID iniciado com `sparkles-`.

---

## 1. Estrutura da App Store

```text
my-umbrel-app-store/
├── umbrel-app-store.yml        # ID e nome da store (não alterar depois de publicada)
├── README.md                   # Template original da store
├── docs/
│   └── GUIA-UMBREL.md          # Este guia
├── sparkles-9router/           # App existente (preservado)
├── sparkles-balanca-backend/   # App existente (preservado)
└── sparkles-todoapp/           # App novo
    ├── umbrel-app.yml          # Manifesto (metadados exibidos na loja)
    ├── docker-compose.yml      # Serviços (frontend, backend, mongodb, app_proxy)
    ├── icon.png                # Ícone 512×512
    ├── gallery/                # Screenshots exibidos na loja
    │   ├── 1.png
    │   ├── 2.png
    │   └── 3.png
    └── data/db/.gitkeep        # Garante que o diretório de dados exista na instalação
```

O UmbrelOS ignora qualquer diretório que **não** tenha `umbrel-app.yml`
(por isso `docs/` não aparece como app na loja).

---

## 2. Como adicionar a App Store personalizada ao UmbrelOS

Na primeira vez, informe a URL do repositório GitHub no UmbrelOS:

### umbrelOS 2.0 (atual)

1. Abra a **loja de aplicativos (App Store)**.
2. No cabeçalho, toque em **Community App Stores** → **Manage**.
3. Cole a URL: `https://github.com/And5reas/my-umbrel-app-store`
4. Clique em **Add**.

### umbrelOS 0.5.x

1. Abra **Settings** (Configurações).
2. No menu, escolha **Community App Stores**.
3. Cole a URL: `https://github.com/And5reas/my-umbrel-app-store`
4. Confirme a adição.

> ⚠️ Apps de stores comunitárias não são verificados pela equipe do Umbrel.
> Adicione apenas stores de desenvolvedores de confiança (esta é a sua própria store).

---

## 3. Como atualizar a App Store

O UmbrelOS mantém uma cópia local da store em
`~/umbrel/app-stores/<slug-da-url>` e verifica novos commits automaticamente
(a cada poucos minutos, ou ao sincronizar manualmente pela UI).

Fluxo para disponibilizar alterações:

```bash
cd /mnt/Projetos/my-umbrel-app-store
# ... edite/adicione arquivos ...
git add <arquivos-novos>
git commit -m "feat(sparkles-todoapp): descrição"
git push origin master
```

Após o push, o Umbrel passa a exibir a nova versão na loja. Não é preciso
reiniciar o dispositivo. **Nunca** faça force-push ou reescreva a história da
branch `master` — o dispositivo faz pull incremental.

---

## 4. Como localizar o aplicativo na loja

1. Abra a loja de aplicativos do UmbrelOS.
2. Na barra de navegação de categorias, escolha **Developer Tools**
   (categoria `developer`) — ou use a busca por **TODOapp**.
3. O card **TODOapp** mostra ícone, tagline e as screenshots da galeria.

Ficha técnica do app:

| Campo | Valor |
|---|---|
| ID | `sparkles-todoapp` |
| Nome | TODOapp |
| Versão | 1.0.0 |
| Porta (app_proxy) | 3210 |
| Desenvolvedor | And5reas |
| Repositório | https://github.com/And5reas/TODOapp_unisanta |

---

## 5. Como instalar, abrir e desinstalar

### Instalar

1. Na loja, abra o **TODOapp**.
2. Clique em **Install**. O UmbrelOS baixa as imagens do Docker Hub e sobe os
   três containers (mongodb → backend → frontend), aguardando os healthchecks.
3. Quando ficar pronto, clique em **Open**.

### Acessar

| Recurso | Endereço |
|---|---|
| Pela UI do Umbrel | Ícone do TODOapp na tela inicial |
| Direto no navegador | `http://umbrel.local:3210` (ou `http://<ip-do-umbrel>:3210`) |
| API de saúde | `http://umbrel.local:3210/api/health` |

O acesso é feito exclusivamente pelo **app_proxy** do Umbrel ( porta 3210 ),
com autenticação do Umbrel habilitada por padrão. Nenhuma porta é publicada
diretamente pelos containers.

> Para desativar o login do Umbrel neste app, adicione em
> `sparkles-todoapp/docker-compose.yml`, dentro de `app_proxy.environment`:
> `PROXY_AUTH_ADD: "false"` (e faça push).

### Comunicação Frontend ↔ Backend

- O Nginx do frontend recebe as chamadas `/api/*` e encaminha para
  `http://backend:3000` usando o **nome de serviço do Compose** na rede interna
  do Umbrel — nunca `localhost`.
- `APP_HOST: sparkles-todoapp_frontend_1` é o nome de container que o próprio
  UmbrelOS injeta (padrão `<app-id>_<serviço>_1`); não altere.

### Desinstalar

Na página do app, clique em **Remove/Uninstall**.

> ⚠️ **A desinstalação apaga os dados do aplicativo** (as tarefas do banco
> MongoDB em `~/umbrel/app-data/sparkles-todoapp/data/db`). Faça backup antes
> se quiser conservar os dados (Umbrel → Configurações → Backups, ou copie o
> diretório manualmente via SSH).

---

## 6. Como atualizar a versão do aplicativo

Sempre que publicar uma nova versão do app:

1. **Atualize as imagens no Docker Hub** (seção 7).
2. **Atualize os digests** em `sparkles-todoapp/docker-compose.yml`:
   ```bash
   docker buildx imagetools inspect and5reas/todoapp-frontend:latest | grep Digest
   docker buildx imagetools inspect and5reas/todoapp-backend:latest  | grep Digest
   # cole os valores em image: and5reas/...:latest@sha256:<digest>
   ```
3. **Atualize o manifesto** `sparkles-todoapp/umbrel-app.yml`:
   - `version:` — incremente (ex.: `1.1.0`);
   - `releaseNotes:` — descreva o que mudou.
4. **Commit + push** para `master`.
5. No UmbrelOS, o app passa a exibir o botão **Update** (ou atualiza
   automaticamente conforme a versão do umbrelOS). Ao atualizar, os dados do
>   banco são preservados (ficam em `app-data`).

> O `version` do manifesto é o que o Umbrel compara para detectar atualização.
> Alterar apenas a imagem no Docker Hub, sem bump de versão, **não** gera
> atualização para os usuários.

---

## 7. Como publicar novas imagens no Docker Hub

As imagens usadas pelo app são:

| Serviço | Imagem | Plataformas atuais |
|---|---|---|
| frontend | `and5reas/todoapp-frontend` | linux/amd64 |
| backend | `and5reas/todoapp-backend` | linux/amd64 |
| mongodb | `mongo:8` (oficial, multi-arch) | linux/amd64 + linux/arm64 |

### Publicar uma nova versão (somente x86_64)

```bash
cd /mnt/Projetos/TODOapp_unisanta
docker login                    # use token de acesso, não a senha da conta

docker compose build
docker compose push             # publica :latest das duas imagens
```

Depois atualize digests + versão no repositório da store (seção 6) e faça push.

### Publicar imagens multi-arquitetura (amd64 + arm64, ex.: Raspberry Pi)

As imagens atuais só rodam em **x86_64**. Para dispositivos ARM, publique
imagens multi-arquitetura com Buildx:

```bash
cd /mnt/Projetos/TODOapp_unisanta
docker login

# Prepara o builder (uma única vez)
docker buildx create --name umbrel-builder --use

# Build + push de ambas as plataformas de uma vez
docker buildx build --platform linux/amd64,linux/arm64 \
  -t and5reas/todoapp-frontend:latest --push ./frontend

docker buildx build --platform linux/amd64,linux/arm64 \
  -t and5reas/todoapp-backend:latest  --push ./backend

# Confira o índice multi-arch publicado
docker buildx imagetools inspect and5reas/todoapp-frontend:latest
```

Em seguida **atualize os digests** no compose da store (os digests mudam) e
faça push do repositório da store. O linter oficial
(`npm run lint:apps -- sparkles-todoapp --check-images`) passa a aceitar as
imagens sem erros de arquitetura.

---

## 8. Validações executadas neste pacote

- `umbrel-app.yml` validado (YAML + campos obrigatórios + prefixo `sparkles-`
  + `storage.dataRoot: data` + `manifestVersion: 1` compatível com
  umbrelOS 0.5.x e 2.0.x).
- `docker compose config` validado (com o stub `app_proxy` completado, exatamente
  como o UmbrelOS faz na instalação).
- Linter oficial do repositório `getumbrel/umbrel-apps`:
  **0 erros** (apenas avisos intencionais: tag `latest` pinada por digest — os
  repositórios só publicam `latest` — e assets de galeria/ícone commitados, o
  que é o correto para uma *community store*).
- Teste de execução completo: instalação simulada em `app-data`, subida em
  cadeia (mongo healthy → backend healthy → frontend healthy), CRUD de tarefas
  passando pelo proxy do Nginx, persistência após restart, logs sem erros,
  autenticação MongoDB (`SCRAM-SHA-256` com `APP_PASSWORD`) confirmada.
- Teste de integração real: instalação da store + app dentro de um container
  **umbrelOS 2.0.1** oficial, com acesso pelo app_proxy.

### Portas

| Uso | Porta |
|---|---|
| UmbrelOS dashboard | 80 / 443 (reservados, não usar) |
| Auth do Umbrel | 2000 (reservado) |
| **TODOapp (app_proxy)** | **3210** (escolhida por estar livre na loja oficial e nesta store) |
| Portas dos containers | somente internas (8080, 3000, 27017) — nada publicado no host |

---

## 9. Solução de problemas

| Problema | Verificação |
|---|---|
| App não inicia | Página do app no Umbrel → **Logs** (ou SSH: `docker logs sparkles-todoapp_frontend_1`) |
| Frontend sem resposta do backend | Verifique se `BACKEND_UPSTREAM=http://backend:3000` (sem `localhost`) |
| Erro de autenticação no Mongo | O `APP_PASSWORD` do Umbrel precisa estar definido; os dados em `data/db` foram criados com uma senha anterior? Nesse caso remova o diretório (perde dados) ou use o backup |
| App não aparece na loja | Confirme o push no GitHub, o prefixo `sparkles-` no ID e que `umbrel-app.yml` está válido |
| Imagem não roda no dispositivo | As imagens do app são somente `linux/amd64`; dispositivos ARM precisam das imagens multi-arch (seção 7) |
| Conflito de porta 3210 | Outro app da loja está usando a porta; escolha outra no manifesto e no `app_proxy` |

Dados do app no dispositivo:

```text
~/umbrel/app-data/sparkles-todoapp/
├── umbrel-app.yml        # cópia do manifesto instalado
├── docker-compose.yml    # cópia do compose instalado
└── data/db/              # dados do MongoDB (persistente, incluído em backups)
```
