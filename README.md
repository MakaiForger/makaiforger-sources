# MakaiForger Sources

Banco de fontes de download para jogos. Arquivos JSON contendo URIs de download organizados por fonte (FitGirl, DODI, SteamRip, etc.).

## Estrutura

```
sources.tar.gz
└── *.json  (29 arquivos de fontes de download)
```

Cada JSON contém metadados dos jogos disponíveis naquela fonte, com títulos, URIs, tamanhos e datas de upload.

## Como publicar uma nova versão

1. **Prepare os arquivos** na pasta `data/sources/` do projeto principal
2. **Empacote:**
   ```bash
   tar -czf sources.tar.gz -C data/sources .
   ```
3. **Crie um release no GitHub** com tag incremental (v0.0.2, v0.0.3...)
   - Asset: `sources.tar.gz`
   - Nunca apague versões anteriores
4. **Envie para o GitLab (fallback):**
   ```bash
   curl -X PUT "https://gitlab.com/api/v4/projects/83081384/packages/generic/makaiforger-sources/v0.0.2/sources.tar.gz" \
     -H "PRIVATE-TOKEN: $GL_TOKEN" \
     -H "Content-Type: application/gzip" \
     --data-binary @sources.tar.gz
   ```
5. **Atualize o metadata.json** no repositório `MakaiForger/makaiforger-resources`

## Como o aplicativo atualiza

O Makai Forger baixa automaticamente o novo `sources.tar.gz` ao iniciar, comparando a versão do `metadata.json` remoto com a versão local. SHA256 é verificado antes da extração.

## Fallback

| Prioridade | Origem |
|------------|--------|
| 1ª | GitHub (`MakaiForger/makaiforger-sources`) |
| 2ª | GitLab (Project ID 83081384) |

## Versionamento

| Versão | Data | Descrição |
|--------|------|-----------|
| v0.0.1 | - | Primeira versão com 29 fontes de download |
