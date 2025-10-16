<%*
// Extract the year and the quarterly from the file path
const fullTitle = tp.file.folder(true) + "/" + tp.file.title;
const parts = fullTitle.split('/');
const year = parts[1];
const quarter = parseInt(parts[2].replace('Q', '')); 

const getFirstMonthOfQuarter = (q) => {
    return (q - 1) * 3 + 1;
};

// fileDate is init to the first day of the quarter
let fileDate = moment(`${year}-${getFirstMonthOfQuarter(quarter)}-01`);

// Navigation paths
let baseDir = 'Bujo/';
let yearPattern = 'YYYY';
let quarterPattern = yearPattern + '/[Q]Q';
let monthPattern = quarterPattern + '/MM-MMMM';

// Previous and next quarters
let prevQuarter = moment(fileDate).subtract(3, 'months').format(quarterPattern);
let nextQuarter = moment(fileDate).add(3, 'months').format(quarterPattern);

// Navigation links formats
let yearLink = fileDate.format(yearPattern);
let quarterLink = fileDate.format(quarterPattern);
let months = [0, 1, 2].map(offset => fileDate.clone().add(offset, 'months').format(monthPattern));

let navStr = ` [[${baseDir}${prevQuarter}|❮❮]] ⋮ [[${baseDir}${yearLink}|${yearLink}]] ⋮ [[${baseDir}${nextQuarter}|❯❯]]\n`;
navStr += months.map(m => `[[${baseDir}${m}|${m.split('/').pop().split('-')[1]}]]`).join(" | ");
%><%"---"%>
tags:
- quarterly_note
- <% quarterLink %>
<%"---"%>
<% navStr %>
## 📌 Objectifs de ce trimestre

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

## 📊 Récapitulatif

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

## 📈 Statistiques

```dataviewjs
const thisQuarter = "<% quarterLink %>";

// Inits statistics variables
let totalMTGMinutes = 0;
let totalMatchesPlayed = 0;
let totalVictories = 0;
let totalDefeats = 0;

// Dataview requests to crawl daily notes from this quarter
dv.pages(`"Bujo/${thisQuarter}"`)
    .where(p => p.file.name.match(/^\d{4}-\d{2}-\d{2}$/)) // Filters on dailies
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

const thisQuarter = "<% quarterLink %>";

// Inits Kendama total playing time in minutes
let totalKendamaMinutes = 0;

// Dataview request to get Kendama stats from this quarter's dailies
dv.pages(`"Bujo/${thisQuarter}"`)
    .where(p => p.file.name.match(/^\d{4}-\d{2}-\d{2}$/)) // Filters only on dailies from this quarters
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

// Create a DIV panel to show the Kendama stats
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
// === DataviewJS – Quarterly bike/walk/vaa ===============================

// This quarter related constants
const year = <%year%>;
const quarter = <%quarter%>;

// Months for this quarter
const startMonth = <%getFirstMonthOfQuarter(quarter)%>;
const months = [startMonth, startMonth + 1, startMonth + 2];
const monthNames = ["janvier","février","mars","avril","mai","juin","juillet","août","septembre","octobre","novembre","décembre"];
const labels = months.map(m => monthNames[m - 1]);

// Monthly totals (km)
let monthlyBike = { [months[0]]: 0, [months[1]]: 0, [months[2]]: 0 };
let monthlyWalk = { [months[0]]: 0, [months[1]]: 0, [months[2]]: 0 };
let monthlyVaa  = { [months[0]]: 0, [months[1]]: 0, [months[2]]: 0 };

// Aggregates data from the dailies of this quarter
dv.pages(`"Bujo/${year}/Q${quarter}"`)
  .where(p => p.file.name.match(/^\d{4}-\d{2}-\d{2}$/))
  .forEach(p => {
    const m = parseInt(p.file.name.split('-')[1], 10);
    if (!months.includes(m)) return;
    if (p.bicycle) monthlyBike[m] += p.bicycle / 1000; // m → km
    if (p.walked)  monthlyWalk[m] += p.walked  / 1000;
    if (p.vaa)     monthlyVaa[m]  += p.vaa     / 1000;
  });

// Series (displayed with one decimal)
let valuesBike = months.map(m => monthlyBike[m].toFixed(1));
let valuesWalk = months.map(m => monthlyWalk[m].toFixed(1));
let valuesVaa  = months.map(m => monthlyVaa[m].toFixed(1));

// Quarters totals (one decimal)
let totalBike = valuesBike.reduce((a,b)=> parseFloat(a)+parseFloat(b), 0).toFixed(1);
let totalWalk = valuesWalk.reduce((a,b)=> parseFloat(a)+parseFloat(b), 0).toFixed(1);
let totalVaa  = valuesVaa.reduce((a,b)=> parseFloat(a)+parseFloat(b), 0).toFixed(1);

// Div for the totals
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

// Chart (Obsidian Charts / Chart.js)
let chart = {
  type: 'bar',
  data: {
    labels,
    datasets: [
      {
        label: 'Km à vélo',
        data: valuesBike,
        backgroundColor: 'rgba(75, 192, 192, 0.6)',
        borderColor: 'rgba(75, 192, 192, 1)',
        borderWidth: 1
      },
      {
        label: 'Km à pied',
        data: valuesWalk,
        backgroundColor: 'rgba(192, 75, 75, 0.6)',
        borderColor: 'rgba(192, 75, 75, 1)',
        borderWidth: 1
      },
      {
        label: 'Km en va’a',
        data: valuesVaa,
        backgroundColor: 'rgba(255, 205, 86, 0.6)',   // amber/gold
        borderColor: 'rgba(255, 205, 86, 1)',
        borderWidth: 1
      }
    ]
  },
  options: {
    scales: { y: { beginAtZero: true } }
  }
};

window.renderChart(chart, this.container);
```
