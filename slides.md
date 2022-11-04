# git-under-the-hood

- Hva er egentlig en commit?
- Hvorfor må en kjøre både git add OG git commit?
- Hva er egentlig en branch?
- Hvordan holder git styr på filer og endringer?

Utgangspunktet her er at en har brukt git før, vet basically hva de mest vanlige git-kommandoene gjør, men skjønner ikke helt hvorfor det fungerer som det gjør, eller hvordan det funker “under the hood”.

```bash
rm -rf .git index.js
```

---

## Directed Acyclic Graph

Det første man må vite er at git er en Directed Acyclic Graph

---

## Start med et enkelt repository

```bash
git init
touch index.js
echo 'console.log("Hei!")' >> index.js
git add -A
git commit -m "Initial commit"
```

---

# Vi skal grave i .git-mappa

```bash
ls -p .git
```

---

## Hva er egentlig en commit?

Som konsept er en commit et innslag i historikken.

Men i realiteten er en commit er egentlig bare ei fil.

### Sammenligne git-historikken:

```bash
git log
```

### Med det som ligger i `objects`-mappen under `.git`

```bash
tree .git/objects/
```

---

## Git er bare et fancy filsystem

Det er ikke en database.

Det er bare masse filer som peker på hverandre.

---

## Hva er innholdet i en commit fil?

En `commit`-fil under `objects` er komprimert med zlib.

Kan bruke `pigz -dcz` for å decompress.

### Eller enda bedre

Bruk `git cat-file -p` for å decompress og print innhold av en object.

---

## Hæ, hva er et tree?

Et `tree` er på samme måte som en `commit` bare en fil under `.git/objects`

Den kan sees nesten på som ei mappe som peker på andre filer og mapper.

---

## En blob er en fil

Og ja, så klart er den også en `.git/objects`

Men kanskje mer korrekt å si, et snapshot av en fil i et gitt øyeblikk.

---

## En snapshot av en blob er hele filen!

Selv om en er vant til å tenke på "patches" i git-verden, så er det ingen diffs mellom filer som mellomlagres.

```bash
echo "return;" >> index.js
git add index.js
git commit -m "fix: Remember to return"
```

---

## Commit peker på en snapshot

```
+--------+     +------+     +------+
| Commit | --> | Tree | --> | Blob |
+--------+     +------+     +------+
```

---

## Commit peker på en annen commit

```
+--------+     +------+     +------+
| Commit | --> | Tree | --> | Blob |
+--------+     +------+     +------+
    |
    V
+--------+     +------+     +------+
| Commit | --> | Tree | --> | Blob |
+--------+     +------+     +------+
```

---

## Hva er poenget med å lære dette?

Fordi noen mener at git er så skummelt og mystisk at hvis du skal gjøre noe som helst utenom det vanlige må du ta en zip-backup av repoet.

```bash
git reset --hard HEAD~1
```

---

## Oops nå er du screwed!

Eller kanskje ikke? `git reflog` to the rescue.

---

## Men hvorfor er den borte fra commit historikken?

Hvis den fortsatt eksisterer i `.git/objects` - hvorfor er den borte fra `main`-branchen sin historikk?

---

## Hva er egentlig en branch?

Da ser vi i `.git/refs`

---

## Hvordan vet git hvilken branch vi er på?

Da ser vi i `.git/HEAD`

---

## Hvordan holder git styr på endringer?

Si at vi skriver noe inn i `index.js`:

```bash
echo "// unecessary return tbh" >> index.js
```

Hvordan vet git at filen har endret seg fra før?
Sjekker den dato på oppdatering kanskje?

---

## Dette er jobben til stage / index

Ligger under `.git/index`:

```bash
cat .git/index
```

Bare tull og fjas!

---

## Index i klartekst

Heldigvis har vi `git ls-files`

```bash
git ls-files -s
```

---

## Sammenligner fil-hashes

Git vet at en fil har endret seg med å sammenligne fil-hash med index

---

## Staged filer blir med til ny branch

```bash
git checkout -b fix-branch
```

Fordi index-filen deles på tvers av branches.

Filer blir med så lenge det ikke er noen conflicts.

---

## Hvordan computes hashes?

SHA1 av git-object type + file length + NULL-byte + content.

Eller bare kjøre `git hash-object` da..

---

## Kult.. I guess?

Ja det er veldig kult! Fordi det viser hvordan git sikrer historikken.

```bash
git log
```

---

## Passer på ved commit amends eller rebases

```bash
echo "Før commit amend:"
git log --oneline
echo "woops!" >> index.js
git add index.js
git commit --amend --no-edit -q
echo "Etter commit amend:"
git log --oneline
```

---

## Merge vs squash vs rebase

Hvorfor oppfører de seg forskjellig?

---

## Merge commits har to parents

....oooog snapshot conflict fixen mellom de.

Så begge historier sammenføyes på dato.

Hva skjer da om du resetter merge-commiten?

---

## Squash er som en merge

Bare at en sammenføyer ikke historien

---

## Hva som er vanskelig med rebase

Og hvorfor det kan være OK å lære seg

## Bonus 1: lokal backup

Uten å zippe folderen! `git clone --bare`
