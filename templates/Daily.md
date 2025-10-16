<%* var fileDate = moment(tp.file.title);
const dateD = fileDate.format("D");
const dateDDD = fileDate.format("DDD");
const dateYYYYMMDD = fileDate.format("YYYY-MM-DD");
// nb of days in month and year
let now = fileDate.toDate();
let nbDaysInMonth = (new Date(now.getFullYear(), now.getMonth() + 1, 0)).getDate();
let year = now.getFullYear();
let isLeapYear = (year % 400 === 0) || (year % 100 !== 0 && year % 4 === 0);
let nbDaysInYear = isLeapYear ? 366 : 365;
// moment dates are mutable
let baseDir = 'Bujo/';
let yearPattern = 'YYYY';
let quarterPattern = yearPattern + '/[Q]Q';
let monthPattern = quarterPattern + '/MM-MMMM';
let dayPattern = monthPattern + '/YYYY-MM-DD';
let weekPattern = yearPattern + '/[Semaines]/[S]WW';
let shortWeekPattern = yearPattern + '/[S]WW';
let prevDay = moment(fileDate).subtract(1, 'd').format(dayPattern);
let nextDay = moment(fileDate).add(1, 'd').format(dayPattern);
let yearLink = fileDate.format(yearPattern);
let quarterLink = fileDate.format(quarterPattern);
let monthLink = fileDate.format(monthPattern);
let weekLink = fileDate.format(weekPattern);
let shortWeekLink = fileDate.format(shortWeekPattern);
let rawFullTitle = fileDate.format('dddd D MMMM YYYY');
let fullTitle = rawFullTitle.charAt(0).toUpperCase() + rawFullTitle.slice(1);
%><%"---"%>
energy_level_morning: 8
energy_level_evening: 8
mood_morning: 
mood_evening: 
walked: 0
bicycle: 0
vaa: 0
working_place_morning:
working_place_afternoon:
tags:
- daily_note
- <% fileDate.format("YYYY-MM-DD") %>
- <% shortWeekLink %>
- <% monthLink %>
- <% quarterLink %>
<%"---"%>
<%*
let navStr = ` [[${baseDir}${prevDay}|❮❮]] ⋮ [[${baseDir}${yearLink}|${yearLink}]] › [[${baseDir}${quarterLink}|${fileDate.format('[Q]Q')}]] › [[${baseDir}${monthLink}|${fileDate.format('MM')}]] › [[${baseDir}${weekLink}|${fileDate.format('[S]WW')}]] ⋮ [[${baseDir}${nextDay}|❯❯]]`;
tR += navStr %>
```progressbar
kind: manual
name: Mois courant (<% dateD %>/<% nbDaysInMonth %>)
value: <% dateD %>
max: <% nbDaysInMonth %>
```
```progressbar
kind: manual
name: Cette année (<% dateDDD %>/<% nbDaysInYear %>)
value: <% dateDDD %>
max: <% nbDaysInYear %>
```
# <% fullTitle %>

## S'il n'y avait qu'un objectif à cette journée

- TODO

## Agenda

> [!todo]+ Aujourd'hui
> ```tasks
> filter by function task.status.name != 'Rescheduled' && task.status.name != 'Done' && task.status.name != 'Cancelled'
> happens <% dateYYYYMMDD %>
> hide recurrence rule
> hide due date
> hide scheduled date
> sort by priority
> ```

> [!abstract]+ Toujours en cours
> ```tasks
> filter by function task.status.name == 'In Progress'
> due on or after <% dateYYYYMMDD %>
> hide recurrence rule
> hide due date
> hide scheduled date
> sort by priority
> ```

> [!danger]+ Dépassé 
> ```tasks
> filter by function task.status.name != 'Rescheduled' && task.status.name != 'Done' && task.status.name != 'Cancelled'
> (due before <% dateYYYYMMDD %>) AND ((happens before <% dateYYYYMMDD %>) AND (priority is above none))
> hide recurrence rule
> sort by due date
> ```

> [!tip]- D'ici deux semaines
> ```tasks
> not done
> filter by function task.status.name != 'Rescheduled' && task.status.name != 'Done' && task.status.name != 'Cancelled'
> happens after <% dateYYYYMMDD %>
> happens before {{date+14d:YYYY-MM-DD}}
> hide recurrence rule
> hide due date
> hide scheduled date
> group by happens
> ```

## Journal

TODO

## Routine

### 🤹 Kendama

Entraînement : (kendama_minutes_played::0) minutes
Nouvelles figures :
- TODO
Figures réussies :
- TODO

### 📖 Livre en cours de lecture

<%*
const bookFolder = app.vault.getAbstractFileByPath("Personnel/Livres");
const files = await app.vault.adapter.list(bookFolder.path);

let booksInProgress = [];
for (const file of files.files) {
  const content = await app.vault.adapter.read(file);
  if (content.includes('currently_reading: true')) {
    booksInProgress.push({ path: file, content: content });
  }
}

if (booksInProgress.length > 0) {
  let tableContent = "";
  
  booksInProgress.forEach(book => {
    const tR = book.path;
    const content = book.content;
    const metadata = content.match(/---([\s\S]*?)---/)[1];
    const pagesTotalMatch = metadata.match(/pages_total:\s+"(\d+)"/);
    const pagesCurrentMatch = metadata.match(/pages_current:\s+"(\d+)"/);
    const pagesTotal = pagesTotalMatch ? pagesTotalMatch[1] : "0";
    const pagesCurrent = pagesCurrentMatch ? pagesCurrentMatch[1] : "0";
    const coverMatch = content.match(/!\[\[(.*?\.(?:jpg|png|gif|jpeg))(\|[0-9]+)?\]\]/);
    const cover = coverMatch ? coverMatch[1] : "";

    const title = tR.split('/').pop().replace('.md', '');

	const tripleQ = '```';
    tableContent += `#### [[${title}]]\n\n![[${cover}|200]]\n ${tripleQ}progressbar\nkind: manual\nname: "Progression (${pagesCurrent}/${pagesTotal})"\nvalue: ${pagesCurrent}\nmax: ${pagesTotal}\n${tripleQ}\n\n`;
  });

  tR += tableContent;
} else {
  tR += "Aucun livre en cours de lecture.\n\n";
}
%>
### 🕹️ Jeux vidéo en cours

<%*
const gamesFolderPath = "Personnel/Jeux vidéos";
const gamesFolder = app.vault.getAbstractFileByPath(gamesFolderPath);

if (!gamesFolder) {
  tR += "Dossier des jeux introuvable.\n\n";
} else {
  const folderContents = await app.vault.adapter.list(gamesFolderPath);
  let currentlyPlayingGames = [];

  for (const subFolder of folderContents.folders) {
    if (!subFolder.includes('.DS_Store')) {
      const subFolderContents = await app.vault.adapter.list(subFolder);

      // Rechercher les fichiers .md dans chaque sous-dossier
      for (const gameFile of subFolderContents.files) {
        if (gameFile.endsWith(".md")) {
          const gameContent = await app.vault.adapter.read(gameFile);

          // Vérifier si le jeu est en cours (currently_playing: "true")
          if (gameContent.includes('currently_playing: "true"')) {
            const gameTemplate = gameContent.match(/{{daily-template}}([\s\S]*?){{daily-template-end}}/);
            if (gameTemplate) {
              currentlyPlayingGames.push(gameTemplate[1].trim());
            }
          }
        }
      }
    }
  }

  // Ajouter chaque section pour les jeux actuellement joués
  if (currentlyPlayingGames.length > 0) {
    currentlyPlayingGames.forEach(game => {
      tR += `${game}\n\n`;
    });
  } else {
    tR += "Aucun jeu actuellement en cours.\n\n";
  }
}
%>
## Tâches terminées aujourd'hui

> [!success]+ Terminé aujourd'hui
> ```tasks
> done <% dateYYYYMMDD %>
> filter by function task.status.name == 'Done'
> hide recurrence rule
> hide due date
> hide scheduled date
> sort by priority
> ```
## Notes modifiées

```dataview
LIST
WHERE dateformat(file.mtime, "yyyy-MM-dd") = "<% dateYYYYMMDD %>" AND this.file.name != file.name
```

## Notes créées

```dataview
LIST
WHERE dateformat(file.ctime, "yyyy-MM-dd") = "<% dateYYYYMMDD %>" AND this.file.name != file.name
```
