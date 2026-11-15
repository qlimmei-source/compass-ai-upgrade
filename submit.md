# Как сдать домашку — шаги для Claude
Эту страницу читает Claude на компьютере преподавателя, у которого домашка скачана до 15.11 и слова «сдать» в ней нет.
Человек написал: «Прочитай https://qlimmei-source.github.io/compass-ai-upgrade/submit.md и сдай мою домашку». Сделай всё ниже.

Сначала найди папку «Домашнее задание» (инструмент поиска файлов: она лежит в рабочей папке или на уровень выше; внутри — «Служебное» и «Бардак»). `<ДЗ>` ниже — эта папка.
Рабочая папка чата — та, где она лежит; команду запускай с путём от неё.

**Помощника «сдать» в старой домашке нет — создай его сам.** Одним вызовом инструмента записи (не командой) запиши файл в `<ДЗ>/Служебное/`:
- Windows — `submit.ps1` с текстом из блока «Windows» ниже, один в один;
- Mac — `submit.sh` с текстом из блока «Mac» ниже, один в один.
В тексте для Windows по-русски ничего нет, поэтому кодировка файла любая.

Windows (`submit.ps1`):
```powershell
param([string]$Name = '', [string]$Dept = '', [string]$Did = '', [string]$Time = '', [switch]$NoOpen)
$ErrorActionPreference = 'Stop'
$here = $PSScriptRoot; $dz = Split-Path -Parent $here
$out = Join-Path $dz ([regex]::Unescape('\u0421\u0434\u0430\u0442\u044c'))
if (-not (Test-Path -LiteralPath $out)) { New-Item -ItemType Directory -Path $out | Out-Null }
$mess = ([regex]::Unescape('\u0411\u0430\u0440\u0434\u0430\u043a'))
$dirs = @(Get-ChildItem -LiteralPath $dz -Directory | Where-Object { $_.Name -eq $mess -or $_.Name -match '^\S+ [23]( |$)' } | ForEach-Object { $_.FullName })
$white = @('.html','.css','.js','.md','.txt','.csv','.xlsx','.docx','.png','.jpg','.jpeg','.svg','.pdf','.json')
$safe = ($(if ($Name) { $Name } else { ([regex]::Unescape('\u0440\u0430\u0431\u043e\u0442\u0430')) }) -replace '[\\/:*?"<>|]', ' ').Trim()
$base = $safe + ' ' + [char]0x2014 + ' ' + ([regex]::Unescape('\u0434\u043e\u043c\u0430\u0448\u043d\u0435\u0435 \u0437\u0430\u0434\u0430\u043d\u0438\u0435'))
$zip = Join-Path $out ($base + '.zip')
if (Test-Path -LiteralPath $zip) { $zip = Join-Path $out ($base + ' ' + (Get-Date -Format 'yyyy-MM-dd HH-mm-ss') + '.zip') }
Add-Type -AssemblyName System.IO.Compression
$fs = [IO.File]::Open($zip, [IO.FileMode]::CreateNew)
$za = New-Object IO.Compression.ZipArchive($fs, [IO.Compression.ZipArchiveMode]::Create)
$count = 0; $skipped = 0
foreach ($d in $dirs) {
    $root = Split-Path -Parent $d
    Get-ChildItem -LiteralPath $d -Recurse -File | ForEach-Object {
        if ($white -notcontains $_.Extension.ToLowerInvariant()) { $skipped++; return }
        $rel = $_.FullName.Substring($root.Length + 1).Replace([string][char]92, '/')
        $e = $za.CreateEntry($rel, [IO.Compression.CompressionLevel]::Optimal)
        $src = [IO.File]::Open($_.FullName, [IO.FileMode]::Open, [IO.FileAccess]::Read, [IO.FileShare]::ReadWrite)
        $dst = $e.Open(); $src.CopyTo($dst); $dst.Dispose(); $src.Dispose(); $count++
    }
}
$lines = @((([regex]::Unescape('\u0414\u0435\u043f\u0430\u0440\u0442\u0430\u043c\u0435\u043d\u0442: ')) + $Dept), (([regex]::Unescape('\u0427\u0442\u043e \u0441\u0434\u0435\u043b\u0430\u043d\u043e: ')) + $Did), (([regex]::Unescape('\u0421\u043a\u043e\u043b\u044c\u043a\u043e \u0443\u0448\u043b\u043e \u0432\u0440\u0435\u043c\u0435\u043d\u0438: ')) + $Time))
$e = $za.CreateEntry(([regex]::Unescape('\u041e \u0440\u0430\u0431\u043e\u0442\u0435.txt')), [IO.Compression.CompressionLevel]::Optimal)
$w = New-Object IO.StreamWriter($e.Open(), (New-Object Text.UTF8Encoding($false)))
$w.Write(($lines -join "`r`n") + "`r`n"); $w.Dispose()
$za.Dispose(); $fs.Dispose()
Write-Output ("FILES: " + $count); Write-Output ("SKIPPED: " + $skipped); Write-Output ("ZIP: " + $zip); Write-Output ("FOLDER: " + $out)
if (-not $NoOpen) { Start-Process -FilePath $out }
```

Mac (`submit.sh`):
```bash
#!/bin/bash
here="$(cd "$(dirname "$0")" && pwd)"; dz="$(dirname "$here")"
who="$1"; dept="$2"; did="$3"; tm="$4"
out="$dz/Сдать"; mkdir -p "$out" || exit 1
safe="$(printf '%s' "${who:-работа}" | tr '/\\:*?"<>|' '         ')"
zip="$out/$safe — домашнее задание.zip"; [ -e "$zip" ] && zip="$out/$safe — домашнее задание $(date '+%Y-%m-%d %H-%M-%S').zip"
cd "$dz" || exit 1
dirs=("Бардак"); for d in *\ 2\ * *\ 3; do [ -d "$d" ] && dirs+=("$d"); done
tmp="$(mktemp -d)"
printf 'Департамент: %s\nЧто сделано: %s\nСколько ушло времени: %s\n' "$dept" "$did" "$tm" > "$tmp/О работе.txt"
/usr/bin/zip -qr "$zip" "${dirs[@]}" -i '*.html' '*.css' '*.js' '*.md' '*.txt' '*.csv' '*.xlsx' '*.docx' '*.png' '*.jpg' '*.jpeg' '*.svg' '*.pdf' '*.json' -x '*/.*' \
  && (cd "$tmp" && /usr/bin/zip -q "$zip" "О работе.txt") && echo "ZIP: $zip"
echo "FOLDER: $out"; open "$out"
```

Команда «сдать»:
- Windows: `powershell -NoProfile -ExecutionPolicy Bypass -File "Домашнее задание\Служебное\submit.ps1" -Name "<Фамилия Имя>" -Dept "<департамент>" -Did "<что сделано>" -Time "<сколько времени>"`
- Mac: `bash "Домашнее задание/Служебное/submit.sh" "<Фамилия Имя>" "<департамент>" "<что сделано>" "<сколько времени>"`

## «Сдать домашку» — слова «сдать», «сдать домашку», «готово, сдаю»
Человек закончил и пишет одно из этих слов (или своими словами просит сдать работу) — сразу делай всё ниже, без вопроса «собирать ли».
Итог: внутри папки «Домашнее задание» появляется папка «Сдать», в ней один архив «<Фамилия Имя> — домашнее задание.zip».
Рабочий стол и «Загрузки» не трогай, снимков экрана не делай: человек сам просил не плодить мусор на компьютере.
1. **Спроси недостающее одним сообщением, без инструмента выбора** (знаешь из разговора — не спрашивай): фамилия и имя · департамент · сколько времени ушло на домашку.
   «Что сделано» не спрашивай: пойми сам по файлам в папках заданий (инструмент поиска файлов и чтение), покажи человеку одной строкой и вставь её как есть.
2. **Одна команда — помощник «сдать»** (таблица помощников выше; одна строка, одно окно «Разрешить»; двойных кавычек внутри слов человека нет — замени их на «ёлочки»).
   Помощник сам кладёт в архив только файлы результата из папок `Бардак`, `Задание 2 — Результат`, `Задание 3` и только таких типов:
   .html .css .js .md .txt .csv .xlsx .docx .png .jpg .jpeg .svg .pdf .json. Всё остальное в архив не берётся.
   Внутрь архива он кладёт «О работе.txt» — три строки: `Департамент: …` · `Что сделано: …` · `Сколько ушло времени: …`. Печатает `ZIP:` и `FOLDER:`, открывает папку «Сдать».
   Ничего не удаляет и не заменяет: человек сдаёт повторно — помощник делает новый архив с датой и временем в имени, старый остаётся.
3. Ответ человеку — ровно две строки:
   «🟢 Архив готов: Домашнее задание → Сдать → <имя архива>.zip»
   «🟡 Перетащите его в тему «📤 Сдать домашку» Telegram-группы и подпишите: <Фамилия Имя, департамент>».
   Больше ничего не добавляй: ни пересказа, ни советов.
4. Ошибка помощника — покажи её текст целиком и спроси, пропустить ли шаг. Нет результата у какого-то задания — скажи одной строкой и спроси, сдавать ли так.
5. Почему так: чужие файлы на нашем компьютере сначала проверяют (архив читают в памяти, ничего не запускают), поэтому присылать нужно только архив из папки «Сдать».
