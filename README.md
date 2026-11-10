# NTFD External 

> Este repositório é mantido apenas como registro histórico/técnico de um projeto
> antigo. Ele foi feito para interagir com o FiveM/GTA V de formas que violam os
> Termos de Serviço da Rockstar/Cfx.re e podem resultar em banimento permanente.
> O autor não oferece suporte, garantia ou autorização de uso.

---

## Status

| Plataforma | Windows 10/11 x64 |
| Loader | C++ / Win32 |
| Payload | C++ / DLL, D3D11 + ImGui |
| Builds citadas | b3258 e b3889 (offsets mudam entre builds) |
| Interface | Overlay próprio + menu Web local |
| Licença | Nenhuma definida (todos os direitos reservados) |

O projeto dependia de um backend remoto para autenticação, perfil, configuração e
entrega do payload. Esse backend não faz parte deste repositório e não é
garantido que continue online.

---

## Visão geral

O NTFD External era dividido em duas partes principais:

1. **Loader** — executável que fazia a autenticação, baixava um payload
   criptografado e o carregava em memória no processo do jogo.
2. **Payload** — DLL com o overlay, o menu, os subsistemas de leitura de
   memória e as funcionalidades do cheat.

Além disso, o repositório contém um Worker de proxy para avatares do Discord e
documentação auxiliar.

```text
Loader
  ├─ instância única
  ├─ autenticação / perfil
  ├─ download do payload
  ├─ descriptografia
  ├─ carregamento em memória (manual map)
  └─ inicialização do payload

Payload
  ├─ anexa ao processo do jogo
  ├─ resolve offsets da build
  ├─ cria overlay D3D11 + ImGui
  ├─ inicia threads de leitura (entidades, nomes, veículos)
  ├─ menu + features
  ├─ Web Menu local
  └─ unload: restaura patches e libera recursos
```

> Para o mapa mental completo, veja [`MAPA_MENTAL_NTFD.md`](MAPA_MENTAL_NTFD.md).

---

## Como funcionava (alto nível)

1. O usuário abria o loader.
2. O loader verificava instância única e registrava um handler de exceções.
3. A autenticação consultava o backend remoto e o serviço de licenças.
4. Após a aprovação, o loader baixava o payload criptografado.
5. O blob era descriptografado e carregado no processo do jogo.
6. O payload resolvia os offsets da build atual e iniciava o overlay.
7. O overlay desenhava ESP, mira, menu e demais elementos via D3D11/ImGui.
8. Threads auxiliares mantinham snapshots de entidades, veículos e nomes.
9. No unload, o payload restaurava patches, liberava hooks e encerrava o overlay.

O loader suportava mais de um modo de carga (memory-only e fallback para
`LoadLibrary` com arquivo temporário). **Memory-only não torna o projeto
indetectável**: o processo continua escrevendo memória, criando threads e
aplicando patches em runtime.

---

## Estrutura do repositório

```text
Ntfd External/
├─ assets/                  Recursos visuais e ícones usados pelo overlay
├─ backup_20261030_230542/  Backup antigo do projeto (não é a versão principal)
├─ DiscordProxy/            Worker/proxy para avatar do Discord
├─ docs/                    Notas de segurança, assinatura e roadmap
├─ loader/                  Executável loader (C++/Win32)
├─ obj/                     Artefatos intermediários de build
├─ src/                     Código do payload
│  ├─ Core/
│  │  ├─ Features/          ESP, aimbot, silent, magic, exploits, Spotify etc.
│  │  ├─ SDK/               Structs, memória, offsets e leitura do jogo
│  │  ├─ Threads/           Entity list, ponteiros, nomes e veículos
│  │  └─ Util/              Utilitários internos
│  ├─ Gui/
│  │  ├─ Login/             Interface de login
│  │  ├─ Overlay/           Janela, D3D11, ImGui e apresentação
│  │  └─ Pages/             Páginas antigas da interface
│  ├─ Includes/             Bibliotecas de terceiros e headers
│  ├─ Security/             Autenticação, anti-crack e proteção interna
│  ├─ Ui/
│  │  └─ pages/             Páginas do menu novo
│  ├─ Globals.hpp           Estado global compartilhado
│  ├─ main.cpp              Entrada do payload
│  └─ src.vcxproj           Projeto Visual Studio do payload
├─ tools/                   Scripts auxiliares
├─ build_payload.ps1        Build + empacotamento/upload do payload
├─ Ntfd External.sln        Solution Visual Studio
└─ MAPA_MENTAL_NTFD.md       Arquitetura detalhada
```

### Diretórios principais

| Caminho | Responsabilidade |
|---|---|
| `loader/loader.cpp` | Autenticação, download, descriptografia, mapeamento e inicialização |
| `src/main.cpp` | Ponto de entrada do payload, attach ao jogo e inicialização |
| `src/Core/SDK/` | Leitura de memória, structs do jogo e resolução de offsets |
| `src/Core/Features/` | Implementação das funcionalidades e do Web Menu |
| `src/Core/Threads/` | Threads de atualização de entidades, nomes e veículos |
| `src/Gui/Overlay/` | Janela transparente, dispositivo D3D11 e render |
| `src/Ui/` | Menu, widgets, tema e páginas |
| `src/Security/` | Login, licença, HWID e verificações internas |
| `DiscordProxy/` | Worker de avatar; não contém o token do bot no cliente |
| `docs/` | Notas de arquitetura, assinatura e segurança |

---

## Subsistemas

### Loader
- Instância única por mutex.
- Log de exceções em `%TEMP%`.
- Autenticação e consulta de perfil.
- Download do payload com versionamento.
- Descriptografia AES.
- Carregamento manual em memória, com fallback opcional.
- Resolução de imports, relocations, `.pdata` e TLS.

### Payload
- Anexo ao processo do jogo e resolução dos offsets da build.
- Overlay D3D11 + ImGui com janela própria.
- Loop de render e ticks de funcionalidades.
- Sistema de configuração e presets.
- Web Menu local para controle pela rede.
- Integração com Spotify via CDP/oEmbed.
- Unload com restauração de patches.

### Leitura do jogo
- Estruturas (`CPed`, `CVehicle`, `CPlayerInfo`, etc.).
- Cache de view matrix/projeção para ESP.
- Lista de entidades e veículos.
- Resolução de nomes por lista local e fontes remotas.
- Detecção de build e offsets por versão.

### Interface
- Menu desenhado em ImGui.
- Tema claro/escuro, accent, presets e configurações.
- Páginas para Aim, Visuals, Chams, World, Player, Settings, Server, Spotify e Crosshair.
- Crosshair com código nativo e presets salvos localmente.
- Web Menu e preview para acesso pelo navegador/celular.

### Spotify
- Leitura de metadados e capa.
- Controle de play/pause, faixa, volume e seek via CDP.
- Extração da cor dominante da capa para o tema da interface.
- Efeito visual de batida no fundo do menu (pulso aproximado em 120 BPM).

---

## Build (histórico)

Requisitos usados na época:

- Visual Studio 2022 (Community ou superior)
- Windows SDK
- DirectX 11
- MSBuild x64
- Bibliotecas já incluídas no projeto: ImGui, Font Awesome, nlohmann/json, libcurl

Build pela solution:

```powershell
msbuild "Ntfd External.sln" /p:Configuration=Release /p:Platform=x64 /m
```

O script `build_payload.ps1` também compilava, criptografava e (opcionalmente)
enviava o payload para o backend. Ele contém **material sensível** (chave de
criptografia e caminho para segredo administrativo). Não versione esse arquivo
sem sanitizá-lo.

Saídas típicas (o caminho exato depende do `OutDir` de cada projeto):

- `src/x64/Release/ntfd_payload.dll` (ou `x64/Release/ntfd_payload.dll`)
- `x64/Release/SpotifySetup.exe` (nome usado pelo loader)
- `payload.enc` (temporário, removido após o upload)

---

## Configuração e persistência

- Presets de mira fixa e algumas preferências ficavam no registro do Windows,
  em `HKCU\Software\NTFD`.
- Configurações do cheat podiam ser salvas/exportadas pelo próprio menu.
- Tema, accent, cores de ESP, binds e opções do Spotify ficavam na configuração
  carregada.
- Nada disso é necessário para entender a arquitetura; são apenas dados locais.

---

## Limitações conhecidas

- Offsets e assinaturas mudam entre builds do jogo; uma atualização pode quebrar
  leitura de entidades, projeção, aimbot e silent aim.
- **Memory-only não é stealth.** O comportamento em runtime (escrita de memória,
  threads remotas, hooks e patches) continua detectável.
- Bancos de dados de nomes podem ser falsificados pelo servidor. Se o servidor
  anonimiza o nome no cliente e na API pública, não há como recuperar o nome real
  apenas pelo lado do cliente.
- A extração de cor da capa depende do formato da imagem e do WIC.
- O efeito de batida é uma aproximação visual; não usa BPM real da faixa.
- O projeto depende de um backend remoto que pode estar offline.

---

## Segurança e segredos

Antes de guardar ou compartilhar qualquer cópia, remova/rotacione:

- `DiscordProxy/admin_secret.txt`
- tokens de bot do Discord
- chaves de criptografia embutidas no loader/script
- URLs privadas de Worker/backend
- HWIDs, logs e configurações pessoais
- DLLs compiladas, `payload.enc`, `.obj`, `.pdb` e caches

Sugestão de `.gitignore`:

```gitignore
# Build
x64/
obj/
*.obj
*.pdb
*.ilk
*.dll
*.exe
payload.enc

# Segredos / dados locais
DiscordProxy/admin_secret.txt
*_token.txt
*.log
*.ini
*.json
!DiscordProxy/package.json
!DiscordProxy/version_meta.json

# IDE
.vs/
*.user
```

> O `DiscordProxy/README.md` original também explica que o token do bot nunca
> deve ir para a máquina do cliente.

---

## Aviso legal

Este projeto **não é afiliado** à Rockstar Games, Take-Two, Cfx.re ou FiveM.
O uso de cheats viola os Termos de Serviço e pode resultar em banimento de
contas e HWID. O código é disponibilizado apenas como registro histórico do
próprio autor, sem garantia de funcionamento, suporte ou autorização de uso.

Se este repositório for publicado no futuro, recomenda-se:

1. manter privado; ou
2. remover toda a funcionalidade de cheat e publicar apenas os componentes
   educacionais/genéricos; ou
3. escolher uma licença adequada e remover segredos e dados pessoais.

---

## Documentação adicional

- [`MAPA_MENTAL_NTFD.md`](MAPA_MENTAL_NTFD.md) — arquitetura detalhada e fluxos.
- [`docs/SECURITY_ROADMAP.md`](docs/SECURITY_ROADMAP.md) — notas de segurança.
- [`docs/SIGNING_AND_FP.md`](docs/SIGNING_AND_FP.md) — assinatura e fingerprint.
- [`DiscordProxy/README.md`](DiscordProxy/README.md) — proxy de avatar.

---

## Bibliotecas e créditos

- Dear ImGui
- Font Awesome
- nlohmann/json
- libcurl
- DirectX 11
- Windows API

Os créditos das bibliotecas de terceiros pertencem aos seus respectivos autores.
