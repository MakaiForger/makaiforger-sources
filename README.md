# MakaiForger Sources

Banco de fontes de download para jogos. Arquivos JSON contendo links (magnet URIs) organizados por fonte — repacks, scene releases, GOG, e outros.

## Objetivo

Fornecer ao Makai Forger uma base local de links de download para que o usuário possa baixar jogos diretamente pelo app, sem precisar visitar sites externos. Cada arquivo JSON é uma "fonte" de downloads, contendo dezenas a milhares de títulos com links magnet.

## Estrutura

```
sources.tar.gz
├── fitgirl.json          (FitGirl Repacks   — 6.8 MB)
├── dodi.json             (DODI Repacks      — 3.1 MB)
├── empress.json          (0xEMPRESS         — 27 KB)
├── xatab.json            (Xatab Repacks     — 6.8 MB)
├── kaoskrew.json         (KaosKrew          — 3.5 MB)
├── thepiratebay.json     (The Pirate Bay    — 5.1 MB)
├── lewdzone.json         (LewdZone          — 5.1 MB)
├── crotorrents.json      (Crotorrents       — 2.3 MB)
├── gog.json              (GOG               — 2.3 MB)
├── brasilrespawn.json    (BrasilRespawn     — 2.0 MB)
├── onlinefix.json        (OnlineFix         — 507 KB)
├── lewdninja.json        (LewdNinja         — 298 KB)
├── repack-info.json      (RePack Info       — 236 KB)
├── ertila-repo.json      (Ertila Repo       — 185 KB)
├── ertila-extremo.json   (Ertila Extremo    — 313 KB)
├── steamrip.json         (SteamRip          — 178 KB)
├── atop-games.json       (Atop Games        — 146 KB)
├── linuxrulez.json       (LinuxRuleZ!       — 94 KB)
├── tinyrepacks.json      (TinyRepacks       — 93 KB)
├── limetorrents.json     (LimeTorrents      — 213 KB)
├── ankergames.json       (AnkerGames        — 48 KB)
├── behind-the-dune.json  (Behind The Dune   — 1.4 KB)
├── steamrip-software.json(SteamRip Software — 1.3 KB)
├── rapelay.json          (RapeLay           — 1.7 KB)
├── persona-5-px.json     (Persona 5 PX      — 1.6 KB)
├── genshin-impact.json   (Genshin Impact    — 402 B)
├── arknights-endfield.json(Arknights Endfield— 429 B)
├── duet-night-abyss.json (Duet Night Abyss  — 449 B)
├── nikke.json            (Nikke             — 349 B)
└── ...
```

**Total: 29 fontes, ~3.4 MB comprimido, ~36 MB descomprimido.**

## Schema dos JSONs

Cada arquivo segue esta estrutura:

```typescript
interface SourceFile {
  name: string;                    // Nome legível da fonte (ex: "FitGirl", "DODI")
  downloads: DownloadEntry[];      // Lista de jogos disponíveis
}

interface DownloadEntry {
  title: string;                   // Título do jogo + versão (ex: "Tomb Raider IV-VI Remastered (MULTi11) [DODI Repack]")
  uris: string[];                  // Links de download (geralmente magnet: URIs)
  fileSize?: string;               // Tamanho legível (ex: "4.8 GB", "643 MB")
  uploadDate?: string;             // Data ISO 8601 (ex: "2025-02-15T00:00:00.000Z")
  steamId?: string;                // (opcional) Steam App ID para matching exato
  recommended?: boolean;           // (opcional) Marcado como recomendado
}
```

### Exemplo real

```json
{
  "name": "FitGirl",
  "downloads": [
    {
      "title": "The Legend of Heroes: Trails through Daybreak II – Complete Edition, v1.1.2 + 15 DLCs",
      "uris": [
        "magnet:?xt=urn:btih:4352BEFD677FA564260B4CAE1D3126BFE90DC231&dn=..."
      ],
      "uploadDate": "2025-02-17T14:00:11.000Z",
      "fileSize": "12.9 GB"
    }
  ]
}
```

### Campos opcionais

| Campo | Tipo | Presente em | Descrição |
|-------|------|-------------|-----------|
| `steamId` | string | repack-info.json, xatab.json, etc. | Steam App ID para matching exato com a biblioteca Steam |
| `recommended` | boolean | fontes internas | Marca o download como recomendado |
| `source` | string | repack-info.json | Identificador adicional da fonte |

## Como o app usa estes dados

O Makai Forger usa `local-sources-handler.ts` que faz 3 operações:

### 1. Listar fontes disponíveis

```
handleGetDownloadSources()
  → lê todos os *.json do diretório data/sources/
  → extrai o nome de cada fonte do campo "name" do JSON
  → retorna lista { id, name, downloadCount, ... }
```

Usado na UI para mostrar quais fontes estão disponíveis e quantos downloads cada uma tem.

### 2. Buscar downloads para um jogo específico

```
handleGetGameDownloadSources(shop, objectId, title, embeddedDownloads?)
  → se objectId tem steamId e algum download na fonte tem steamId matching → match exato
  → senão, compara title com cada download.title usando normalização (lowercase, sem acentos/símbolos)
  → se um download.title contém o search title OU vice-versa → match fuzzy
  → retorna todos os matches com URIs, tamanho, data
```

Usado quando o usuário abre um jogo e quer ver opções de download disponíveis.

### 3. Saber quais fontes têm um jogo

```
handleGetSourceNamesForTitle(title)
  → varre todas as fontes
  → retorna array com nomes das fontes que têm matching para o título
```

Usado para mostrar badges como "Disponível em: FitGirl, DODI, Xatab".

### Fluxo completo

```
Usuário pesquisa/navega até um jogo
    │
    ▼
UI chama handleGetGameDownloadSources(título do jogo)
    │
    ▼
Handler varre as 29 fontes:
  ├── Match por steamId (se disponível)
  └── Match fuzzy por título
    │
    ▼
Retorna todos os links encontrados, agrupados por fonte
    │
    ▼
Usuário escolhe um link → app abre no qBittorrent (ou download direto)
```

## Como adicionar uma nova fonte

### 1. Crie o arquivo JSON

```json
{
  "name": "Minha Fonte",
  "downloads": [
    {
      "title": "Meu Jogo (v1.0) [Minha Fonte]",
      "uris": ["magnet:?xt=urn:btih:..."],
      "fileSize": "10 GB",
      "uploadDate": "2026-06-10T00:00:00.000Z"
    }
  ]
}
```

### 2. Coloque em `data/sources/`

Nomenclatura: `minha-fonte.json` (lowercase, hífens).

### 3. Regenerae o tarball

```bash
tar -czf sources.tar.gz -C data/sources .
```

### 4. Publique (seção abaixo)

### Boas práticas

- **Magnet URIs** são preferíveis (funcionam com qBittorrent integrado)
- **steamId** ajuda no matching exato — inclua quando souber
- **Limite de tamanho** — fontes grandes tipo FitGirl (6.8 MB) são aceitáveis, mas tente manter cada fonte abaixo de 10 MB
- **Dados atuais** — fontes desatualizadas (ex: títulos de 2023 sem versões novas) são menos úteis
- **Evite duplicatas** — se duas fontes têm o mesmo jogo, o app mostra ambas (o usuário escolhe)
- **Caracteres especiais** — o matching usa normalização, então acentos e símbolos não quebram a busca

## Como publicar uma nova versão

1. **Atualize os JSONs** na pasta `data/sources/`
2. **Empacote:**
   ```bash
   tar -czf sources.tar.gz -C data/sources .
   ```
3. **Crie um release no GitHub** com tag incremental (`v0.0.2`, `v0.0.3`, etc.)
   - Asset: `sources.tar.gz`
   - Nunca apague versões anteriores
4. **Envie para o GitLab (fallback):**
   ```bash
   curl -X PUT \
     "https://gitlab.com/api/v4/projects/83081384/packages/generic/makaiforger-sources/v0.0.2/sources.tar.gz" \
     -H "PRIVATE-TOKEN: $GL_TOKEN" \
     -H "Content-Type: application/gzip" \
     --data-binary @sources.tar.gz
   ```
5. **Atualize o metadata.json** em `MakaiForger/makaiforger-resources`:
   ```json
   "sources.tar.gz": {
     "version": 2,
     "tag": "v0.0.2",
     "sha256": "<sha256sum>",
     "size": <tamanho_gz>,
     "original_size": <tamanho_descomprimido>,
     "github_repo": "MakaiForger/makaiforger-sources",
     "gitlab_project_id": 83081384
   }
   ```

## Como o app atualiza automaticamente

O `resource-manager.ts` no Makai Forger:

1. Ao iniciar, baixa `metadata.json` de `MakaiForger/makaiforger-resources`
2. Compara versão local vs remota do `sources.tar.gz`
3. Se `localVersion < remoteVersion`, baixa o novo arquivo
4. Verifica SHA256 — se falhar, loga erro e pula
5. Extrai com `tar -xzf` para `resources/data/sources/`
6. Os handlers em `local-sources-handler.ts` leem os JSONs diretamente do diretório

## Sistema de fallback

| Prioridade | Origem | URL |
|------------|--------|-----|
| 1ª | GitHub | `MakaiForger/makaiforger-sources` releases |
| 2ª | GitLab | Project ID 83081384 (generic packages) |

## Versionamento

| Versão | Data | Descrição |
|--------|------|-----------|
| v0.0.1 | - | Primeira versão — 29 fontes de download |

Todas as versões são mantidas nos releases do GitHub e GitLab. Nunca apague versões anteriores.

## Manutenção

- **Periodicidade:** atualizar conforme novas fontes surgirem ou links quebrarem
- **Validação:** verificar se os magnet URIs ainda são válidos (alguns trackers morrem)
- **Tamanho:** cada fonte pode crescer bastante. Remova títulos muito antigos se necessário
- **Formato:** todo JSON deve ter `name` e `downloads` array. Valide antes de publicar
- **Remoção de fonte:** se uma fonte ficar obsoleta, remova o arquivo JSON da pasta antes de empacotar

Projeto mantido por **desenvolvedor** independente.
