<%*
const fullTitle = tp.file.folder(true) + "/" + tp.file.title;
const parts = fullTitle.split('/');
const weekDate = moment(parts[1] + '-W' + parts[3].split('S')[1]);

// moment dates are mutable
let baseDir = 'Bujo/';
let yearPattern = 'YYYY';
let quarterPattern = yearPattern + '/[Q]Q';
let monthPattern = quarterPattern + '/MM-MMMM';
let dayPattern = monthPattern + '/YYYY-MM-DD';
let weekPattern = yearPattern + '/[Semaines]/[S]WW';
let shortWeekPattern = yearPattern + '/[S]WW';

// Calculate previous and next weeks
let prevWeek = moment(weekDate).subtract(1, 'week').format(weekPattern);
let nextWeek = moment(weekDate).add(1, 'week').format(weekPattern);

// Links
let yearLink = weekDate.format(yearPattern);
let weekLink = weekDate.format(weekPattern);
let shortWeekLink = weekDate.format(shortWeekPattern);

// Calculate the quarters associated with the week
let endOfWeek = weekDate.clone().endOf('isoWeek');
let startQuarter = weekDate.clone().quarter();
let endQuarter = endOfWeek.clone().quarter();

let quarterLinks = [];
if (startQuarter === endQuarter) {
  let quarterDate = weekDate.clone().startOf('quarter');
  quarterLinks.push(`[[${baseDir}${quarterDate.format(quarterPattern)}|${quarterDate.format('[Q]Q')}]]`);
} else {
  let startQuarterDate = weekDate.clone().startOf('quarter');
  let endQuarterDate = endOfWeek.clone().startOf('quarter');
  quarterLinks.push(`[[${baseDir}${startQuarterDate.format(quarterPattern)}|${startQuarterDate.format('[Q]Q')}]]`);
  quarterLinks.push(`[[${baseDir}${endQuarterDate.format(quarterPattern)}|${endQuarterDate.format('[Q]Q')}]]`);
}
let currentQuarterLink = quarterLinks.join(' | ');

// Calculate months associated with the week
let monthLinks = [];
let currentMonth = weekDate.clone().startOf('month');
let endOfWeekMonth = endOfWeek.clone().startOf('month');
monthLinks.push(`[[${baseDir}${currentMonth.format(monthPattern)}|${currentMonth.format('MM')}]]`);
if (currentMonth.format('YYYY-MM') !== endOfWeekMonth.format('YYYY-MM')) {
    monthLinks.push(`[[${baseDir}${endOfWeekMonth.format(monthPattern)}|${endOfWeekMonth.format('MM')}]]`);
}
let currentMonthLink = monthLinks.join(' | ');

// Generate navigation links
let navStr = ` [[${baseDir}${prevWeek}|❮❮]] ⋮ [[${baseDir}${yearLink}|${yearLink}]] › ${currentQuarterLink} › ${currentMonthLink} ⋮ [[${baseDir}${nextWeek}|❯❯]]`;

// Generate daily links
let days = [];
let dayLabels = ['lundi', 'mardi', 'mercredi', 'jeudi', 'vendredi', 'samedi', 'dimanche'];
let currentDay = weekDate.clone();

for (let i = 0; i < 7; i++) {
  let dayLink = currentDay.format(dayPattern);
  days.push(`[[${baseDir}${dayLink}|${dayLabels[i]}]]`);
  currentDay.add(1, 'day');
}

let dayLinks = days.join(' | ');

// Output the navigation strings
navStr += '\n' + dayLinks;
%><%"---"%>
tags:
- weekly_note
- <% shortWeekLink %>
<%"---"%>
<% navStr %>

## Objectifs de la semaine

- #esn 
	- 
- #client 
	- 
- #perso 
	- 
- #tech 
	- 
- #mtg
	- 
- #kendama 
	- 

## Récapitulatif de la semaine

- #esn 
	- 
- #client 
	- 
- #perso 
	- 
- #tech 
	- 
- #mtg
	- 
- #kendama 
	- 

## Statistiques

```dataviewjs
// DataviewJS code

const week = `<% weekDate.format("WW") %>`;
let year = `<% weekDate.format("GGGG") %>`;  // GGGG is iso week

function capitalizeFirstLetter(string) {
    if (typeof string !== 'string' || string.length === 0) {
        return '';
    }
    return string.charAt(0).toUpperCase() + string.slice(1);
}

const startOfWeek = moment().year(year).week(week).startOf('isoWeek');
const endOfWeek = moment().year(year).week(week).endOf('isoWeek');

// MTG stats variables
let totalMTGMinutes = 0;
let totalMatchesPlayed = 0;
let totalVictories = 0;
let totalDefeats = 0;

// Loop agains each daily of the week
for (let date = startOfWeek.clone(); date.isBefore(endOfWeek); date.add(1, 'day')) {
    const yearForDay = date.isoWeekYear();
    const monthForDay = date.month() + 1;
    const quarterForDay = Math.ceil(monthForDay / 3);

    // Current day of the week daily filepath
    const filePath = `Bujo/${yearForDay}/Q${quarterForDay}/${String(monthForDay).padStart(2, '0')}-${date.format('MMMM')}/${date.format('YYYY-MM-DD')}`;

    // Get the data of the page
    let page = dv.page(filePath);

    // Add data from this daily
    if (page) {
        if (page.mtga_minutes_played) totalMTGMinutes += page.mtga_minutes_played;
        if (page.mtgo_minutes_played) totalMTGMinutes += page.mtgo_minutes_played;

        if (page.mtga_bo1_matches) totalMatchesPlayed += page.mtga_bo1_matches;
        if (page.mtga_bo3_matches) totalMatchesPlayed += page.mtga_bo3_matches;
        if (page.mtgo_bo1_matches) totalMatchesPlayed += page.mtgo_bo1_matches;
        if (page.mtgo_bo3_matches) totalMatchesPlayed += page.mtgo_bo3_matches;
        if (page.mtgo_commander_matches) totalMatchesPlayed += page.mtgo_commander_matches;

        if (page.mtga_victories) totalVictories += page.mtga_victories;
        if (page.mtgo_victories) totalVictories += page.mtgo_victories;
        if (page.mtga_defeats) totalDefeats += page.mtga_defeats;
        if (page.mtgo_defeats) totalDefeats += page.mtgo_defeats;
    }
}

// Converts minutes in hours and remaining minutes
let totalMTGHours = Math.floor(totalMTGMinutes / 60);
let remainingMinutes = totalMTGMinutes % 60;

// Compute the winrate
let winRate = totalMatchesPlayed > 0 ? ((totalVictories / totalMatchesPlayed) * 100).toFixed(2) : 0;

// Path to Magic the Gathering image
const imagePath = app.vault.adapter.getResourcePath("Images/mtg_logo.png");

// Create a DIV panel to show the MTG stats
let div = document.createElement('div');
div.style.textAlign = 'center';
div.innerHTML = `
    <div style="display: flex; justify-content: center; align-items: center; margin-bottom: 20px;">
        <div style="margin-right: 20px; display: flex; align-items: center;">
            <span style="font-size: 60px; font-weight: 900;"><img src="${imagePath}" style="height:90px"/></span>
        </div>
        <div style="margin-right: 20px; display: flex; align-items: center;">
            <span style="font-size: 60px; font-weight: 900;">${totalMTGHours}</span>
            <div style="display: inline-flex; flex-direction: column; align-items: center; margin-left: 8px;">
                <span style="font-size: 24px;">h</span>
                <span style="font-size: 24px; line-height: 0.8;">&nbsp;</span>
            </div>
        </div>
        <div style="display: flex; align-items: center;">
            <span style="font-size: 60px; font-weight: 900;">${remainingMinutes}</span>
            <div style="display: inline-flex; flex-direction: column; align-items: center; margin-left: 8px;">
                <span style="font-size: 24px;">min</span>
                <span style="font-size: 24px; line-height: 0.8;">&nbsp;</span>
            </div>
        </div>
    </div>
    <div style="display: flex; justify-content: center;">
        <div style="text-align: center; margin-right: 40px;">
            <span style="font-size: 24px; font-weight: 700;">Parties jouées</span><br>
            <span style="font-size: 36px; font-weight: 900;">${totalMatchesPlayed}</span><br>
            <span style="font-size: 24px; font-weight: 700;">Winrate</span><br>
            <span style="font-size: 36px; font-weight: 900;">${winRate}%</span>
        </div>
        <div style="text-align: center;">
            <span style="font-size: 24px; font-weight: 700;">Victoires</span><br>
            <span style="font-size: 36px; font-weight: 900;">${totalVictories}</span><br>
            <span style="font-size: 24px; font-weight: 700;">Défaites</span><br>
            <span style="font-size: 36px; font-weight: 900;">${totalDefeats}</span>
        </div>
    </div>
`;
this.container.appendChild(div);
```

```dataviewjs
// DataviewJS code

const week = `<% weekDate.format("WW") %>`;
let year = `<% weekDate.format("GGGG") %>`;  // GGGG is the ISO week

function capitalizeFirstLetter(string) {
    if (typeof string !== 'string' || string.length === 0) {
        return '';
    }
    return string.charAt(0).toUpperCase() + string.slice(1);
}

const startOfWeek = moment().year(year).week(week).startOf('isoWeek');
const endOfWeek = moment().year(year).week(week).endOf('isoWeek');

// Inits the number of minutes played
let dailyKendamaMinutes = {};

// Loop over the week days
for (let date = startOfWeek.clone(); date.isBefore(endOfWeek); date.add(1, 'day')) {
    const yearForDay = date.isoWeekYear();
    const monthForDay = date.month() + 1;
    const quarterForDay = Math.ceil(monthForDay / 3);
    const dayName = capitalizeFirstLetter(date.format('dddd'));

    // Current day of the week daily filepath
    const filePath = `Bujo/${yearForDay}/Q${quarterForDay}/${String(monthForDay).padStart(2, '0')}-${date.format('MMMM')}/${date.format('YYYY-MM-DD')}`;

    // Gets the current daily content
    let page = dv.page(filePath);

    // If there is no data for the current day of the week, set the value to zero
    if (!dailyKendamaMinutes[dayName]) {
        dailyKendamaMinutes[dayName] = 0;
    }

    // Add data for the day
    if (page && page.kendama_minutes_played) {
        dailyKendamaMinutes[dayName] += page.kendama_minutes_played;
    }
}

// Converts minutes in hours and remaining minutes
let totalKendamaMinutes = Object.values(dailyKendamaMinutes).reduce((a, b) => a + b, 0);
let totalKendamaHours = Math.floor(totalKendamaMinutes / 60);
let remainingMinutes = totalKendamaMinutes % 60;

// Path to kendama image
const imagePath = app.vault.adapter.getResourcePath("Images/kendama.png");

// DIV to show the results
let div = document.createElement('div');
div.style.textAlign = 'center';
div.innerHTML = `
    <div style="display: flex; justify-content: center; align-items: center;">
        <div style="margin-right: 40px; display: flex; align-items: center;">
            <span style="font-size: 60px; font-weight: 900;"><img src="${imagePath}" style="height:90px"/></span>
        </div>
        <div style="margin-right: 40px; display: flex; align-items: center;">
            <span style="font-size: 60px; font-weight: 900;">${totalKendamaHours}</span>
            <div style="display: inline-flex; flex-direction: column; align-items: center; margin-left: 8px;">
                <span style="font-size: 24px;">h</span>
                <span style="font-size: 24px; line-height: 0.8;">&nbsp;</span>
            </div>
        </div>
        <div style="display: flex; align-items: center;">
            <span style="font-size: 60px; font-weight: 900;">${remainingMinutes}</span>
            <div style="display: inline-flex; flex-direction: column; align-items: center; margin-left: 8px;">
                <span style="font-size: 24px;">min</span>
                <span style="font-size: 24px; line-height: 0.8;">&nbsp;</span>
            </div>
        </div>
    </div>
`;
this.container.appendChild(div);
```

```dataviewjs
// Weekly bike / walk / va'a

// This week constants
const year = `<% weekDate.format("GGGG") %>`;
const week = `<% weekDate.format("WW") %>`;

function capitalizeFirstLetter(string) {
    if (typeof string !== 'string' || string.length === 0) return '';
    return string.charAt(0).toUpperCase() + string.slice(1);
}

const startOfWeek = moment().year(year).week(week).startOf('week');
const endOfWeek   = moment().year(year).week(week).endOf('week');

console.log("Début de la semaine :", startOfWeek.format('YYYY-MM-DD'));
console.log("Fin de la semaine :", endOfWeek.format('YYYY-MM-DD'));

let dailyDataBike = {};
let dailyDataWalk = {};
let dailyDataVaa  = {};

// Loop over the days of the week
for (let date = startOfWeek.clone(); date.isBefore(endOfWeek); date.add(1, 'day')) {
    const yearForDay = date.year();
    const monthForDay = date.month() + 1; // moment: 0-11
    const quarterForDay = Math.ceil(monthForDay / 3);
    const dayName = capitalizeFirstLetter(date.format('dddd'));

    const filePath = `Bujo/${yearForDay}/Q${quarterForDay}/${String(monthForDay).padStart(2, '0')}-${date.format('MMMM')}/${date.format('YYYY-MM-DD')}`;
    let page = dv.page(filePath);

    // init
    if (!dailyDataBike[dayName]) dailyDataBike[dayName] = 0;
    if (!dailyDataWalk[dayName]) dailyDataWalk[dayName] = 0;
    if (!dailyDataVaa[dayName])  dailyDataVaa[dayName]  = 0;

    // additions (m -> km)
    if (page && page.bicycle) dailyDataBike[dayName] += (page.bicycle || 0) / 1000;
    if (page && page.walked)  dailyDataWalk[dayName] += (page.walked  || 0) / 1000;
    if (page && page.vaa)     dailyDataVaa[dayName]  += (page.vaa     || 0) / 1000;
}

console.log("Vélo :", dailyDataBike);
console.log("Marche :", dailyDataWalk);
console.log("Va'a :", dailyDataVaa);

// Histogram
let labels = ['Lundi','Mardi','Mercredi','Jeudi','Vendredi','Samedi','Dimanche'];
let valuesBike = labels.map(d => dailyDataBike[d] || 0);
let valuesWalk = labels.map(d => dailyDataWalk[d] || 0);
let valuesVaa  = labels.map(d => dailyDataVaa[d]  || 0);

console.log("Labels :", labels);
console.log("Vélo :", valuesBike);
console.log("Marche :", valuesWalk);
console.log("Va'a :", valuesVaa);

// Totals
let totalBike = valuesBike.reduce((a,b)=>a+b,0).toFixed(1);
let totalWalk = valuesWalk.reduce((a,b)=>a+b,0).toFixed(1);
let totalVaa  = valuesVaa.reduce((a,b)=>a+b,0).toFixed(1);

// DIV for the result
let div = document.createElement('div');
div.style.textAlign = 'center';
div.innerHTML = `
  <div style="display:flex; justify-content:center; align-items:center;">
    <div style="margin-right:40px; display:flex; align-items:center;">
      <span style="font-size:60px; font-weight:900;">${totalBike}</span>
      <div style="display:inline-flex; flex-direction:column; align-items:center; margin-left:8px;">
        <span style="font-size:24px;">km</span>
        <span style="font-size:24px; line-height:0.8;">🚴</span>
      </div>
    </div>
    <div style="margin-right:40px; display:flex; align-items:center;">
      <span style="font-size:60px; font-weight:900;">${totalWalk}</span>
      <div style="display:inline-flex; flex-direction:column; align-items:center; margin-left:8px;">
        <span style="font-size:24px;">km</span>
        <span style="font-size:24px; line-height:0.8;">🚶</span>
      </div>
    </div>
    <div style="display:flex; align-items:center;">
      <span style="font-size:60px; font-weight:900;">${totalVaa}</span>
      <div style="display:inline-flex; flex-direction:column; align-items:center; margin-left:8px;">
        <span style="font-size:24px;">km</span>
        <span style="font-size:24px; line-height:0.8;">🛶</span>
      </div>
    </div>
  </div>
`;
this.container.appendChild(div);

// Chart (Obsidian Charts)
let chart = {
  type: 'line',
  data: {
    labels,
    datasets: [
      {
        label: 'Km à vélo',
        data: valuesBike,
        backgroundColor: 'rgba(75, 192, 192, 0.2)',
        borderColor: 'rgba(75, 192, 192, 1)',
        borderWidth: 1,
        fill: false
      },
      {
        label: 'Km à pied',
        data: valuesWalk,
        backgroundColor: 'rgba(192, 75, 75, 0.2)',
        borderColor: 'rgba(192, 75, 75, 1)',
        borderWidth: 1,
        fill: false
      },
      {
        label: 'Km en va’a',
        data: valuesVaa,
        backgroundColor: 'rgba(255, 205, 86, 0.2)',
        borderColor: 'rgba(255, 205, 86, 1)',
        borderWidth: 1,
        fill: false
      }
    ]
  },
  options: {
    scales: { y: { beginAtZero: true } }
  }
};

window.renderChart(chart, this.container);
```