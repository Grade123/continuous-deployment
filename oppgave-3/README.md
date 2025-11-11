# 3 - Continuous deployment

Hva skal vi innom:
- Sette opp Github Pages
- Bruke Github Actions til å deploye en applikasjon til Github Pages

I denne delen skal vi jobbe med å få web-applikasjonen vår deployet ut til Github Pages.

GitHub Pages er en gratis webhosting-tjeneste fra GitHub som lar deg publisere statiske nettsider direkte fra et GitHub-repository.

## 3.1 - Oppsett av Github Pages

I repoet, gå inn i "Settings" i topp-fanen, og velg "Pages". 

Du vil få opp denne siden: 
![Enable github pages](pages.png)


Under "Source", endre fra "Deploy from branch" til "Github Actions"
![Enable Github Action](pages_enable_gha.png)

Vi har nå mulighet til å deploye kode til Github Pages fra Github Actions :tada:

## 3.2 - Deploy av web-applikasjon til Pages

I YAML-filen vi har jobbet i før, legg til en ny jobb som heter `deploy`.

```diff
name: Build and deploy

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./code
    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
      - run: npm run lint
      - run: npm run test

      - name: Archive artifacts
        uses: actions/upload-artifact@v4
        with:
          name: artifact
          path: ./code/dist

+  deploy:
+    needs: build
+    permissions:
+      pages: write     
+      id-token: write   
+
+    runs-on: ubuntu-latest
+    steps:
+      - name: Deploy to GitHub Pages
+        id: deployment
+        artifact_name: artifact
+        uses: actions/deploy-pages@v4 
```

:pencil2: Sjekk at koden din er tilgjengelig på internett. Du finner lenke til Github Pages her:
TOdo: Sett inn bilde. 

### Hva gjør deploy-jobben?

Deploy-jobben kjører kun etter at build-jobben er fullført (`needs: build`). Dette sikrer at vi kun deployer når bygget har gått bra.

Jobben må ha spesielle tillatelser for å kunne skrive til GitHub Pages og bruke ID-token for autentisering. Dette settes med `permissions`-seksjonen (Dvs. vi gir Action-kjøringen vår tilgang til å skrive til Github Pages). 

Selve deploy-steget bruker `actions/deploy-pages@v4` som henter artifaktet vi lastet opp i build-jobben (ved å referere til `artifact_name: artifact`) og deployer dette til GitHub Pages.

## 3.3 - Splitt CI / CD pipeline og bruk av Github Releases

Nå har vi en felles jobb from kjører både bygg og deploy. Som regel ønsker vi å skille disse to, der vi har kontinuerlig kjøring av CI-prosesser (bygg, linting og testing), mens vi har et aktivt forhold til når vi deployer koden. 

GitHub Releases lar deg markere viktige milepæler i prosjektet ditt ved å tagge spesifikke versjoner av koden (f.eks. `v1.0.0`, `v2.1.3`). Når du lager en release, kan du legge ved en changelog som beskriver hva som er nytt, endret eller fikset i denne versjonen. Dette gjør det enkelt for brukere og utviklere å følge med på utviklingen av prosjektet og forstå hva som har endret seg mellom ulike versjoner.

Vi kan også trigge actions ved å lage en release. Dette fungerer bra, med at vi kan generere en changelog over koden vår, tagge commit vi deployer og deretter deploye koden vår ut til et miljø. 

### 3.3.1 - Trigger deploy fra Github Releases

:pencil2: Fjern `deploy`-steget fra action filen du før har jobbet på. Fjern også steget `Archive artifacts`. 

TODO: legg inn eksempel

:pencil2: Opprett en ny actions fil som heter `deploy.yaml`. Legg følgende innhold inn:

```diff
name: Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./code
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
      - name: Archive artifacts
        uses: actions/upload-artifact@v4
        with:
          name: artifact
          path: ./code/dist
      - name: Deploy to GitHub Pages
        id: deployment
        artifact_name: artifact
        uses: actions/deploy-pages@v4
```
