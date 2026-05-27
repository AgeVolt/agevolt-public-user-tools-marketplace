---
name: agevolt-charging-history
description: Pouzi ked pouzivatel chce svoje AgeVolt nabijacie transakcie, historiu nabijania, charging transactions, nabijania za posledny mesiac alebo obdobny read-only prehlad z my.agevolt.com.
---

# AgeVolt Charging History

Pouzivaj iba priame MCP tools zo servera `agevolt-portal`.

## Auth

Pri poziadavke na nabijacie transakcie najprv pouzi dostupny MCP tool. Ak MCP
server vyziada OAuth, nechaj Codex spustit standardny MCP auth flow a otvorit
prihlasenie v browseri. Neposielaj pouzivatela do terminalu a neziadaj ho, aby
spustal terminalovy MCP login prikaz.

Ak MCP tool call vrati `Auth required` alebo transport error s textom
`Auth required`, a shell je dostupny, spusti standardny MCP OAuth login ty a
nechaj Codex CLI otvorit browser s authorization URL. Neotvaraj authorization
URL druhykrat cez vlastny `Start-Process`, aby nevznikli duplicitne login taby.
V jednom user requeste spusti tento fallback najviac raz; po uspesnom login-e
vzdy zopakuj povodny MCP tool call v tom istom chate. Nepytaj pouzivatela na
novy chat len preto, ze prebehol OAuth login.

```powershell
$codex = if ($env:CODEX_CLI_PATH) { $env:CODEX_CLI_PATH } else { (Get-ChildItem "$env:LOCALAPPDATA\OpenAI\Codex\bin" -Recurse -Filter codex.exe | Sort-Object LastWriteTime -Descending | Select-Object -First 1).FullName }
$env:CODEX_HOME = Join-Path $env:USERPROFILE ".codex"
$out = Join-Path $env:TEMP "agevolt-portal-mcp-login.out.log"
$err = Join-Path $env:TEMP "agevolt-portal-mcp-login.err.log"
$lock = Join-Path $env:TEMP "agevolt-portal-mcp-login.lock"
if ((Test-Path $lock) -and ((Get-Date) - (Get-Item $lock).LastWriteTime).TotalMinutes -lt 5) {
  throw "AgeVolt MCP login is already in progress; wait for the existing browser login to finish"
}
"$PID" | Set-Content $lock
Remove-Item $out,$err -ErrorAction SilentlyContinue
try {
  $cmd = '""' + $codex + '" mcp login agevolt-portal --scopes MCP.Access > "' + $out + '" 2> "' + $err + '""'
  $p = Start-Process -FilePath "$env:SystemRoot\System32\cmd.exe" -ArgumentList @('/d','/s','/c',$cmd) -PassThru -WindowStyle Hidden
  $deadline = (Get-Date).AddMinutes(5)
  while (-not $p.HasExited -and (Get-Date) -lt $deadline) {
    Start-Sleep -Milliseconds 500
  }
  if (-not $p.HasExited) {
    Stop-Process -Id $p.Id -Force -ErrorAction SilentlyContinue
    throw "AgeVolt MCP login did not complete before timeout"
  }
  Get-Content $out,$err -ErrorAction SilentlyContinue
} finally {
  Remove-Item $lock -ErrorAction SilentlyContinue
}
```

Potom zopakuj povodny MCP tool call. Pouzivatela neziadaj, aby tento prikaz
spustal rucne ani aby sam otvaral authorization URL.

Ak pouzivatel chce logout a prihlasenie pod inym uctom, najprv pouzi
`agevolt_logout`, potom spusti `codex mcp logout agevolt-portal` rovnakym
sposobom cez shell s nastavenym `CODEX_HOME` na `%USERPROFILE%\.codex`.
Nasledne spusti `codex mcp login agevolt-portal --scopes MCP.Access` a po
dokonceni loginu zopakuj povodnu MCP poziadavku.

Ak MCP tools v aktualnom chate stale nie su viditelne, povedz strucne, ze treba
refreshnut plugin session po instalacii/deployi; novy chat spominaj iba ako
poslednu moznost, ked MCP server vobec neposkytuje tools/list.

## Tools

Pouzi `agevolt_charging_transactions_list` pre read-only nacitanie ukoncenych
nabijacich transakcii prihlaseneho pouzivatela.

Vstupy:

- `period`: `last_30_days`, `previous_calendar_month`, alebo `current_month`
- `from`: volitelny ISO datum/datetime
- `to`: volitelny ISO datum/datetime
- `limit`: volitelne, default `50`, maximum `200`

Ak pouzivatel povie "posledny mesiac" bez dalsieho spresnenia, pouzi
`period = last_30_days`.

Kazdy vystup z nabijacich transakcii obsahuje `context` s userId, spaceId,
spaceName a rolami. V odpovedi pouzivatelovi vzdy uveď, pre ktoreho pouzivatela
a ktory space sa data zobrazili.

Pouzi `agevolt_current_context`, ked sa pouzivatel pyta, pod akym AgeVolt
uctom/space je MCP prihlasene.

Pouzi `agevolt_spaces_list`, ked pouzivatel chce vidiet dostupne spaces alebo
chce prepnut priestor.

Pouzi `agevolt_space_switch` so `spaceId` zo zoznamu spaces, ked chce pouzivatel
prepnut MCP session na iny space. Po prepnutí pokracuj dalsim volanim MCP toolu,
ak pouzivatel povodne chcel zobrazit data v novom space.

Pouzi `agevolt_logout`, ked pouzivatel chce odhlasit aktualny AgeVolt MCP ucet
alebo sa prihlasit pod inym. Po logoute nepokracuj v citani dat; povedz, ze dalsi
AgeVolt request musi prejst novym OAuth loginom.

## Safety

- Nevolaj `https://api.my.agevolt.com` ani `https://api1.my.agevolt.com` cez shell, curl, browser alebo raw HTTP fallback.
- Necitaj `.codex/.credentials.json`, cookies, localStorage ani ine token storage.
- Nepytaj si JWT alebo refresh token od pouzivatela.
- Charging history a context/list spaces su read-only. Jedine povolene stavove
  MCP operacie su `agevolt_space_switch` a `agevolt_logout`, ak si ich pouzivatel
  vyziada alebo su potrebne na splnenie jeho poziadavky.
