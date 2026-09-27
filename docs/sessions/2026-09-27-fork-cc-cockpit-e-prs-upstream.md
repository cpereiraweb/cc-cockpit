# Sessão 2026-09-26/27: fork do cc-cockpit, seis melhorias e PRs para o upstream

## Objetivo

Trazer para o cc-cockpit (de wallacemartinss) as ideias já testadas no codex-cockpit — autostart
controlável, pacote .deb e ícone próprio — e, no caminho, corrigir o que incomodava no uso diário:
a aba Plano que pedia para digitar o que o Claude Code já sabe, o rótulo da bandeja cortado e uma
"conta" que não era conta.

## Estado final

| Onde | O quê |
|---|---|
| Fork `cpereiraweb/cc-cockpit` | branch padrão `cpereira/main` (versão pessoal, `0.5.2+cp3`); `main` idêntico ao upstream |
| Upstream `wallacemartinss/cc-cockpit` | PRs #3 a #8 abertos, todos *mergeable* sobre `99f5674` |
| Branch `pr-assets` do fork | capturas usadas nos PRs (conta fictícia Jane Doe), links fixados no commit `8faf4d9` |
| Máquina local | pacote `cc-cockpit_0.5.2+cp3_all.deb` gerado em `~/Code/Ferramentas/cc-cockpit-pessoal/dist/` |

### Worktrees em `~/Code/Ferramentas/`

| Pasta | Branch | PR |
|---|---|---|
| `cc-cockpit` | `feat/autostart-icon` | #4 |
| `cc-cockpit-discover` | `fix/account-discovery` | #3 |
| `cc-cockpit-version` | `feat/settings-version` | #5 |
| `cc-cockpit-plan` | `feat/plan-suggestion` | #6 |
| `cc-cockpit-tray` | `feat/tray-label-formats` | #7 |
| `cc-cockpit-ptax` | `feat/ptax-rate` | #8 |
| `cc-cockpit-pessoal` | `cpereira/main` | — (versão pessoal) |
| `cc-cockpit-assets` | `pr-assets` | — (imagens) |

## O que foi feito, por PR

- **#3 `fix/account-discovery`** — `~/.claude-mem` (pasta do plugin claude-mem) entrava como conta porque um
  `settings.json` bastava. Agora só contam `projects/`, `sessions/`, `.claude.json` ou `.credentials.json`.
- **#4 `feat/autostart-icon`** — `--autostart on|off|status`; desligar grava `Hidden=true` em vez de apagar
  (anula entrada de sistema e sobrevive a reinstalações); `setup` continua ativando por padrão, mas respeita
  um "off" explícito; `TryExec`; caminho absoluto de invocação vence o PATH; switch "Iniciar ao entrar";
  ícone SVG (o anel da bandeja); dependências `python3-gi-cairo` e `gir1.2-gtk-3.0` no deb; primeiros testes e passo no CI.
- **#5 `feat/settings-version`** — título das Configurações mostra `v<versão>`, como o dashboard.
- **#6 `feat/plan-suggestion`** — aba Plano só leitura com os dados da conta (plano, preço, nome, e-mail,
  organização, assinante desde, uso extra, validade do login); custo manual vira ajuste opcional;
  BRL/R$ como padrão em português; mês em moeda local ao lado do retorno; retorno formatado no idioma (11,8×).
- **#7 `feat/tray-label-formats`** — dez formatos de rótulo com prévia ao vivo; botão Aplicar; a bandeja
  passa a receber uma cópia do config.
- **#8 `feat/ptax-rate`** — botão "Buscar no Banco Central" (SGS série 1, PTAX), só no clique, só com BRL;
  endereço editável em "Avançado" com botão "Padrão".

## Decisões e por quê

- **Perfil antes das credenciais para o plano.** O `rateLimitTier` de `.credentials.json` é fixado no login e
  fica velho depois de uma troca de plano (visto: perfil `max_20x`, token `max_5x`). O `oauthAccount` de
  `.claude.json` é rebuscado pelo Claude Code.
- **`tray_label: null` = seguir `tray_show_cost`.** Um valor padrão fixo seria gravado pelo `config.ensure()`
  e mudaria a bandeja de quem já tinha desligado o valor.
- **Previsão do limite só se vier antes do reset.** Com os dados reais, "limite em 11h30" com reset em 3h30
  anunciava um aperto que não aconteceria.
- **PTAX só no clique.** O upstream anuncia "sem chamadas de rede"; a exceção é explícita, opcional e
  documentada no README e no .deb. `rate_url` fica `null` enquanto o padrão é usado, para que uma correção
  futura do endereço chegue a quem nunca o mudou.
- **Branch `cpereira/main` como versão pessoal**, independente do destino dos PRs. Sufixo `+cpN` na versão
  para o `apt` tratar como mais nova e para distinguir no `dpkg -l`. Nunca criar tags `v*` no fork: o
  `release.yml` do upstream publica no PyPI com elas.
- **Capturas dos PRs em ambiente isolado** (HOME temporário, conta fictícia), para não expor dados pessoais em repositório público.

## Enganos no caminho

- **Rótulo cortado:** a primeira explicação (painel cheio) estava certa; foi descartada cedo demais ao
  procurar um limite fixo que não existe na extensão AppIndicator nem no tema Yaru. Confirmado quando a
  área de notificação foi esvaziada e o rótulo apareceu inteiro.
- **`--autostart off` sem entrada prévia** não gravava nada, e um `setup` seguinte reativaria o autostart.
  Corrigido e registrado em `~/Vaults/DevNotes/solved-errors/2026-09-26-autostart-off-sem-entrada-nao-persistia.md`.
- **`apt install` do mesmo número de versão** não troca o pacote. Registrado em
  `~/Vaults/DevNotes/solved-errors/2026-09-27-apt-install-deb-mesma-versao-nao-reinstala.md`.
- **Commit antes da validação:** `feat/settings-version` foi commitada e publicada antes de o usuário validar,
  contrariando o CLAUDE.md. O usuário aprovou em seguida; nos passos restantes o merge ficou sem commit até a validação.

## Pendências

- Acompanhar os PRs #3 a #8. Depois do primeiro merge, os demais precisam de rebase (conflitos pequenos em
  `tests/__init__.py`, `i18n.py`, `preferences.py`).
- A cada release do upstream: merge de `upstream/main` na `cpereira/main`, versão `<nova>+cp1`, gerar e instalar o .deb.
- `dist/` do `cc-cockpit-pessoal` guarda pacotes de teste antigos (`0.5.2`, `+cp1`, `+cp2`) e existe um stash
  `teste-validado-cp2` sem uso — podem ser descartados.
- A entrada `mem` do config do usuário só sai com `cc-cockpit accounts --remove mem`.
