# MPCM Workspace — Guia de desenvolvimento

## Estado atual

- Versão: `0.5.2`
- Aplicação desktop local-first para conexões `SSH`, `Telnet`, `RDP` e `VNC`
- Frontend: React 19, TypeScript, Zustand, Tailwind CSS e Vite
- Backend: Tauri 2 e Rust
- Persistência: SQLite/SQLCipher; segredos do estado portátil são cifrados com a senha mestra
- O launcher RDP nativo é o caminho recomendado; o viewer interno em `clients/internal-rdp-client` permanece experimental

## Comandos principais

```bash
npm install
npm run dev
npm run tauri dev
npm run build
npm run tauri build
npm run version:sync
```

Para o cliente RDP interno:

```bash
npm run rdp:viewer
npm run rdp:screenshot
```

## Estrutura

- `src/components/`: UI reutilizável e terminal integrado
- `src/pages/`: telas e fluxos de produto
- `src/store/`: stores Zustand persistidas e de sessão
- `src/lib/`: sync, backup, estado portátil, validação, lançadores e integrações
- `src-tauri/src/`: comandos Tauri, banco, criptografia e protocolos remotos
- `clients/internal-rdp-client/`: viewer RDP experimental em Rust
- `.github/workflows/build.yml`: build multiplataforma e publicação por tag `v*`

Entry points: `src/main.tsx`, `src/App.tsx`, `src-tauri/src/main.rs` e `src-tauri/src/lib.rs`.

## Convenções

- Preserve alterações do usuário e evite mudanças oportunistas fora do pedido.
- Use os tipos compartilhados de `src/types/index.ts` e os sanitizadores existentes.
- Todo arquivo externo de sync/backup atravessa `src/lib/portableStateSchema.ts` antes de ser aplicado.
- Nunca relaxe a rejeição de segredos em texto claro para corrigir compatibilidade. Na exportação, use listas positivas de campos públicos; segredos pertencem ao envelope cifrado.
- Mudanças de estado portátil devem manter sync e backup consistentes e preservar os vínculos por ID entre hosts, credenciais e chaves SSH.
- O estado importado é persistido de forma transacional por `src/lib/portableStatePersistence.ts` e pelo backend SQLite.
- Mensagens visíveis devem existir em `pt-BR` e `en-US`.
- Registre falhas operacionais pelo logger compartilhado, sem incluir senhas, chaves ou tokens.

## Versionamento e validação

`package.json` é a fonte da versão. Após alterá-lo, execute `npm run version:sync` para sincronizar `package-lock.json`, `src-tauri/Cargo.toml` e `src-tauri/Cargo.lock`.

Antes de publicar, execute pelo menos:

```bash
npm run build
cargo test --manifest-path src-tauri/Cargo.toml
```

Tags `v*` acionam o workflow que compila Linux, Windows e macOS e cria a GitHub Release com os artefatos.
