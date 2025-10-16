<%*
const fullTitle = tp.file.folder(true) + "/" + tp.file.title;
const parts = fullTitle.split('/');
const fileDate = moment(parts[1] + '-' + parts[3].split('-')[0] + '-01');

// moment dates are mutable
let baseDir = 'Bujo/';
let yearPattern = 'YYYY';
let quarterPattern = yearPattern + '/[Q]Q';
let monthPattern = quarterPattern + '/MM-MMMM';
let dayPattern = monthPattern + '/YYYY-MM-DD';
let weekPattern = yearPattern + '/[Semaines]/[S]WW';

// Calculate previous and next months
let prevMonth = moment(fileDate).subtract(1, 'month').format(monthPattern);
let nextMonth = moment(fileDate).add(1, 'month').format(monthPattern);

// Format year, quarter, and current month
let yearLink = fileDate.format(yearPattern);
let quarterLink = fileDate.format(quarterPattern);
let monthLink = fileDate.format(monthPattern);

// Generate navigation links
let navStr = ` [[${baseDir}${prevMonth}|❮❮]] ⋮ [[${baseDir}${yearLink}|${yearLink}]] › [[${baseDir}${quarterLink}|${fileDate.format('[Q]Q')}]] ⋮ [[${baseDir}${nextMonth}|❯❯]]`;

// Generate weekly links
let weeks = [];
let startOfMonth = moment(fileDate).startOf('month');
let endOfMonth = moment(fileDate).endOf('month');
let startOfWeek = startOfMonth.startOf('isoWeek');

while (startOfWeek.isBefore(endOfMonth)) {
  let weekLink = startOfWeek.format(weekPattern);
  weeks.push(`[[${baseDir}${weekLink}|${startOfWeek.format('[S]WW')}]]`);
  startOfWeek.add(1, 'week');
}

let weekLinks = weeks.join(' | ');

// Generate daily links
let days = [];
let currentDay = fileDate.clone();

while (currentDay.isBefore(endOfMonth)) {
  let dayLink = currentDay.format(dayPattern);
  days.push(`[[${baseDir}${dayLink}|${currentDay.format('D')}]]`);
  currentDay.add(1, 'day');
}

let dayLinks = days.join(' | ');

// Output the navigation strings
navStr += '\n' + weekLinks + '\n' + dayLinks;
%><%"---"%>
tags:
- monthly_note
- <% quarterLink %>
- <% monthLink %>
<%"---"%>
<% navStr %>
## Objectifs de ce mois

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

## Récapitulatif

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

// Files starting with this pattern are dailies of the current month
const thisMonth = '<% fileDate.format("YYYY-MM") %>';

// Dataview request to get MTG games data from this month's dailies
let totalMTGMinutes = 0;
let totalMatchesPlayed = 0;
let totalVictories = 0;
let totalDefeats = 0;

dv.pages(`"<% baseDir + monthLink %>"`)
    .where(p => p.file.name.startsWith(thisMonth))
    .forEach(p => {
        // Add total play time from MTGO and MTGA play time
        if (p.mtga_minutes_played) totalMTGMinutes += p.mtga_minutes_played;
        if (p.mtgo_minutes_played) totalMTGMinutes += p.mtgo_minutes_played;
        
        // Add number of matches for MTGO/MTGA, gathering BO1/BO3/Commander matches
        if (p.mtga_bo1_matches) totalMatchesPlayed += p.mtga_bo1_matches;
        if (p.mtga_bo3_matches) totalMatchesPlayed += p.mtga_bo3_matches;
        if (p.mtgo_bo1_matches) totalMatchesPlayed += p.mtgo_bo1_matches;
        if (p.mtgo_bo3_matches) totalMatchesPlayed += p.mtgo_bo3_matches;
        if (p.mtgo_commander_matches) totalMatchesPlayed += p.mtgo_commander_matches;
        
        // Add MTGO/MTGA victories and defeats
        if (p.mtga_victories) totalVictories += p.mtga_victories;
        if (p.mtgo_victories) totalVictories += p.mtgo_victories;
        if (p.mtga_defeats) totalDefeats += p.mtga_defeats;
        if (p.mtgo_defeats) totalDefeats += p.mtgo_defeats;
    });

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

// Files starting with this pattern are dailies of the current month
const thisMonth = "<% fileDate.format("YYYY-MM") %>";

// Dataview request to get Kendama stats from this month's dailies
let totalKendamaMinutes = 0;
dv.pages(`"<% baseDir + monthLink %>"`)
	.where(p => p.file.name.startsWith(thisMonth))
	.forEach(p => {
	        if (p.kendama_minutes_played) {
	            totalKendamaMinutes += p.kendama_minutes_played;
	        }
	    });

// Converts minutes in hours and remaining minutes
let totalKendamaHours = Math.floor(totalKendamaMinutes / 60);
let remainingMinutes = totalKendamaMinutes % 60;

// Path to Kendama image
const imagePath = app.vault.adapter.getResourcePath("Images/kendama.png");

// Create a DIV panel to show the MTG stats
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
// === DataviewJS – Monthly bike/walk/vaa ===============================

// This month related constants
const year = `<% fileDate.format("YYYY") %>`;
const month = `<% fileDate.format("MM") %>`;
const monthName = `<% fileDate.format("MMMM") %>`;
const thisMonth = `${year}-${month}`;

// Get pages for the current month
let data = dv.pages(`"Bujo/${year}/Q${Math.ceil(month / 3)}/${month}-${monthName}"`)
             .where(p => p.file.name.startsWith(thisMonth));

// Gather data from dailies (km)
let dailyDataBike = {};
let dailyDataWalk = {};
let dailyDataVaa  = {};
for (let page of data) {
    let date = page.file.name;
    let day = date.split('-').pop().padStart(2, '0');

    // NOTE: fields are in meters -> converts them to kilometers
    if (page.bicycle) dailyDataBike[day] = (page.bicycle || 0) / 1000;
    if (page.walked)  dailyDataWalk[day] = (page.walked  || 0) / 1000;
    if (page.vaa)     dailyDataVaa[day]  = (page.vaa     || 0) / 1000;
}

// Labels and values
let daysInMonth = new Date(year, month, 0).getDate();
let labels = Array.from({ length: daysInMonth }, (_, i) => (i + 1).toString().padStart(2, '0'));

let valuesBike = labels.map(day => dailyDataBike[day] || 0);
let valuesWalk = labels.map(day => dailyDataWalk[day] || 0);
let valuesVaa  = labels.map(day => dailyDataVaa[day]  || 0);

// Totals (with one decimal)
let totalBike = valuesBike.reduce((a, b) => a + b, 0).toFixed(1);
let totalWalk = valuesWalk.reduce((a, b) => a + b, 0).toFixed(1);
let totalVaa  = valuesVaa.reduce((a, b) => a + b, 0).toFixed(1);

// ====== Totals DIV ===================================================
let div = document.createElement('div');
div.style.textAlign = 'center';

const metric = (value, label, emoji) => `
  <div style="margin-right: 40px; display: flex; align-items: center;">
    <span style="font-size: 60px; font-weight: 900;">${value}</span>
    <div style="display: inline-flex; flex-direction: column; align-items: center; margin-left: 8px;">
      <span style="font-size: 24px;">km</span>
      <span style="font-size: 24px; line-height: 0.8;">${emoji}</span>
    </div>
  </div>
`;

div.innerHTML = `
  <div style="display:flex; justify-content:center; align-items:center;">
    ${metric(totalBike, 'Vélo', '🚴')}
    ${metric(totalWalk, 'Marche', '🚶')}
    ${metric(totalVaa,  'Va’a', '🛶')}
  </div>
`;
this.container.appendChild(div);

// ====== Chart (Obsidian Charts / Chart.js) ===============================
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
        fill: false,
        tension: 0,
        pointRadius: 0
      },
      {
        label: 'Km à pied',
        data: valuesWalk,
        backgroundColor: 'rgba(192, 75, 75, 0.2)',
        borderColor: 'rgba(192, 75, 75, 1)',
        borderWidth: 1,
        fill: false,
        tension: 0,
        pointRadius: 0
      },
      {
        label: 'Km en va’a',
        data: valuesVaa,
        backgroundColor: 'rgba(255, 205, 86, 0.2)',   // ambre
        borderColor: 'rgba(255, 205, 86, 1)',
        borderWidth: 1,
        fill: false,
        tension: 0,
        pointRadius: 0
      }
    ]
  },
  options: {
    scales: { y: { beginAtZero: true } },
    plugins: {
      legend: { position: 'top' },
      tooltip: { mode: 'index', intersect: false }
    },
    interaction: { mode: 'nearest', axis: 'x', intersect: false }
  }
};

window.renderChart(chart, this.container);
```

## Lieu de travail
```dataviewjs
const year = `<% fileDate.format("YYYY") %>`;
const month = `<% fileDate.format("MM") %>`;
const daysInMonth = moment(`${year}-${month}`, "YYYY-MM").daysInMonth();

const holidays = [
  `${year}-01-01`, `${year}-04-01`, `${year}-05-01`, `${year}-05-08`, `${year}-05-09`, `${year}-05-20`, 
  `${year}-07-14`, `${year}-08-15`, `${year}-11-01`, `${year}-11-11`, `${year}-12-25`
];

let table = `<table style="border-collapse: collapse; table-layout: fixed; width: 100%">`;
table += `<tr><th style="padding: 0;"></th>`;
for (let i = 1; i <= daysInMonth; i++) {
    const dayOfWeek = moment(`${year}-${month}-${i}`, "YYYY-MM-DD").day();
    const isWeekend = dayOfWeek === 6 || dayOfWeek === 0;
    const isHoliday = holidays.includes(`${year}-${String(month).padStart(2, '0')}-${String(i).padStart(2, '0')}`);
    let style = isWeekend || isHoliday ? 'background-color: #5D4E58;' : '';
    table += `<th style="padding: 0; font-size:0.78em; text-align: center; ${style}">${i}</th>`;
}
table += `</tr>`;

const createRow = (label, timeOfDay, bgColor, altColor) => {
    let row = `<tr><td style="padding: 0; font-size: 0.68em; background-color: ${bgColor};">${label}</td>`;
    for (let i = 1; i <= daysInMonth; i++) {
        const dateStr = `${year}-${String(month).padStart(2, '0')}-${String(i).padStart(2, '0')}`;
        const filePath = `Bujo/${year}/${moment(dateStr).format('[Q]Q')}/${moment(dateStr).format('MM-MMMM')}/${dateStr}`;
        let cellContent = "";
        
        const page = app.plugins.plugins.dataview.api.page(filePath);
        if (page) {
            const workingPlace = page[`working_place_${timeOfDay}`];
            if (workingPlace) {
                if (workingPlace === "ESN") cellContent = `<span title="ESN">🏢</span>`;
                else if (workingPlace === "Client") cellContent = `<span title="Client">💼</span>`;
                else if (workingPlace === "Maison") cellContent = `<span title="Maison">🏠</span>`;
                else if (workingPlace === "Congé") cellContent = `<span title="Maison">🏖️</span>`;
                else if (workingPlace === "Maladie") cellContent = `<span title="Maison">🤢</span>`;
            }
        }

        const dayOfWeek = moment(dateStr, "YYYY-MM-DD").day();
        const isWeekend = dayOfWeek === 6 || dayOfWeek === 0;
        const isHoliday = holidays.includes(dateStr);
        let style = isWeekend || isHoliday ? `background-color: ${altColor};` : `background-color: ${bgColor};`;
        row += `<td style="padding: 0; font-size:0.78em; text-align: center; ${style}">${cellContent}</td>`;
    }
    row += `</tr>`;
    return row;
}

table += createRow("AM", "morning", "#03396C", "#4F5B66");
table += createRow("PM", "afternoon", "#005B96", "#65737E");

table += `</table>`;
dv.paragraph(table);
```
