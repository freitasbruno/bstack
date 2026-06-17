---
name: update
description: "Harness update skill for BStack projects. Fetches the latest skill files from github.com/freitasbruno/bstack and overwrites only changed files in .claude/skills/. Tracks update history in .claude/memory/harness.md. Never modifies CLAUDE.md, memory files, or project settings."
---

# /update — Update Harness

You are updating the BStack skill files in this project to the latest 
version from the source repository. Only changed files are overwritten. 
CLAUDE.md, memory files, and project settings are never touched.

## On start

1. Check .claude/memory/harness.md — if it exists, report the last 
   update date and commit hash. If not, note this is the first 
   recorded update.

2. Clone the bstack source to a temp directory:

   $ts = Get-Date -Format 'yyyyMMddHHmmss'
   $tmp = "$env:TEMP\bstack-update-$ts"
   git clone --depth 1 https://github.com/freitasbruno/bstack $tmp

   If clone fails: report the error and stop.

3. Get the commit hash:

   Set-Location $tmp
   git rev-parse --short HEAD

4. Compare each skill file in the source against the local copy by 
   content hash. Collect files that are changed or new:

   $changed = @()
   Get-ChildItem -Recurse -File "$tmp\.claude\skills\" | ForEach-Object {
       $rel = $_.FullName.Substring(("$tmp\.claude\skills\").Length)
       $dest = Join-Path ".claude\skills" $rel
       if (Test-Path $dest) {
           $srcHash = (Get-FileHash $_.FullName -Algorithm MD5).Hash
           $dstHash = (Get-FileHash $dest -Algorithm MD5).Hash
           if ($srcHash -ne $dstHash) { $changed += $rel }
       } else {
           $changed += $rel
       }
   }

   Also check memory-viewer.html separately:

   $viewerChanged = $false
   if (Test-Path "$tmp\.claude\memory-viewer.html") {
       if (Test-Path ".claude\memory-viewer.html") {
           $srcHash = (Get-FileHash "$tmp\.claude\memory-viewer.html" -Algorithm MD5).Hash
           $dstHash = (Get-FileHash ".claude\memory-viewer.html" -Algorithm MD5).Hash
           if ($srcHash -ne $dstHash) { $viewerChanged = $true }
       } else {
           $viewerChanged = $true
       }
   }

5. If nothing changed and viewer is unchanged:
   - Clean up: `Remove-Item -Recurse -Force $tmp`
   - Report: "Already up to date. @ [commit-hash]"
   - Stop.

6. List only the changed/new files and ask once:
   "These files will be updated. Proceed?"

## Apply update

On confirmation:

1. Copy only changed skill files:

   foreach ($rel in $changed) {
       $src  = Join-Path "$tmp\.claude\skills" $rel
       $dest = Join-Path ".claude\skills" $rel
       $dir  = Split-Path $dest -Parent
       if (-not (Test-Path $dir)) { New-Item -ItemType Directory -Force $dir | Out-Null }
       Copy-Item -Path $src -Destination $dest -Force
   }

2. If viewer changed, copy it:

   Copy-Item -Path "$tmp\.claude\memory-viewer.html" `
     -Destination ".claude\memory-viewer.html" -Force

3. Clean up:

   Remove-Item -Recurse -Force $tmp

## On completion

1. Run `Get-Date -Format "yyyy-MM-dd HH:mm"` to get the real timestamp.

2. Write or append to .claude/memory/harness.md:

   If creating for the first time:

   # Harness Update Log

   Tracks BStack harness updates for this project instance.
   Source: https://github.com/freitasbruno/bstack

   | Updated | Commit | Changed |
   |---------|--------|---------|

   Then append the row, listing the changed skill names (comma-separated):

   | [timestamp] | [commit-hash] | [skill-a, skill-b, ...] |

3. Report:

   Updated: [N] files — [skill-a, skill-b, ...]
   From:    https://github.com/freitasbruno/bstack @ [commit-hash]
   Logged:  .claude/memory/harness.md
