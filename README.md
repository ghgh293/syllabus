<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>СТК Расписание</title><style>body{
font-family: Arial, sans-serif;  background: #f4f6fb;
  margin: 0;} 
  header{  background:#1976d2;color: white;
  padding: 20px;
  text-align: center;}
.container {
  padding: 20px;
  max-width: 900px;
  margin: auto;
}
select {
  width: 100%;
  padding: 12px;
  margin-bottom: 15px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 16px;}
.day {
  background: white;
  margin-bottom: 15px;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);}
.lesson {
  border-bottom: 1px solid #eee;
  padding: 10px 0;}
.lesson:last-child {
  border-bottom: none;}
.time {color: #1976d2;
  font-weight: bold;}
.error { background: #ffecec;
  color: #b00020;
  padding: 15px;
  border-radius: 10px;} </style> </head> <body> <header>
  <h1>СТК Расписание</h1></header><div class="container">
  <select id="group" onchange="render()"></select>
  <div id="app">Загрузка расписания...</div></div>
<script>
const sheetId = "1-ADgnmolIwio09kzPEaD_Atu0OyVxgob-VqKhQK2e8Q";
const sheetName = "Расписание";
let schedule = {};
function loadData() {
  return new Promise((resolve, reject) => {
    const callbackName = "googleSheetCallback_" + Date.now(); window[callbackName] = function(json) {
      try {
        const rows = json.table.rows;
        const data = {}; rows.forEach(row => {
          const group = row.c[0]?.v?.toString().trim();
          const day = row.c[1]?.v?.toString().trim();
          const num = row.c[2]?.v?.toString().trim();
          const subject = row.c[3]?.v?.toString().trim();
          const room = row.c[4]?.v?.toString().trim();
          const teacher = row.c[5]?.v?.toString().trim();
          if (!group || !day || !num || !subject) return;
          if (!data[group]) data[group] = {};
          if (!data[group][day]) data[group][day] = [];
          data[group][day].push({
            n: num,
            name: subject,
            room: room || "—",
            teacher: teacher || "—"
          });
        });
        for (let group in data) {
          for (let day in data[group]) {
            data[group][day].sort((a, b) => Number(a.n) - Number(b.n));
          }
        }
        resolve(data);
      } catch (error) {
        reject(error);
      }
      delete window[callbackName];
      script.remove();
    };
    const script = document.createElement("script");
    script.src =
      "https://docs.google.com/spreadsheets/d/" +
      sheetId +
      "/gviz/tq?tqx=responseHandler:" +
      callbackName +
      "&sheet=" +
      encodeURIComponent(sheetName);
    script.onerror = function() {
      reject(new Error("Не удалось загрузить таблицу. Проверь доступ."));
      delete window[callbackName];
      script.remove();
    };
    document.body.appendChild(script);
  });
}
function fillGroups() {
  const select = document.getElementById("group");
  select.innerHTML = "";
  const groups = Object.keys(schedule).sort();
  groups.forEach(g => {
    const opt = document.createElement("option");
    opt.value = g;
    opt.textContent = g;
    select.appendChild(opt);
  });
}
function render() {
  const group = document.getElementById("group").value;
  const app = document.getElementById("app");
  app.innerHTML = "";
  if (!group || !schedule[group]) {
    app.innerHTML = "<div class='error'>Расписание для группы не найдено</div>";
    return;
  }
  const days = schedule[group];
  const dayOrder = [
    "Понедельник",
    "Вторник",
    "Среда",
    "Четверг",
    "Пятница",
    "Суббота"
  ];
  const sortedDays = Object.keys(days).sort((a, b) => {
    return dayOrder.indexOf(a) - dayOrder.indexOf(b);
  });
  sortedDays.forEach(day => {
    const dayDiv = document.createElement("div");
    dayDiv.className = "day";
    const title = document.createElement("h3");
    title.textContent = day;
    dayDiv.appendChild(title);
    days[day].forEach(lesson => {
      const div = document.createElement("div");
      div.className = "lesson";
      div.innerHTML = `
        <div class="time">${lesson.n} пара</div>
        <div><b>${lesson.name}</b></div>
        <div>${lesson.room} каб. • ${lesson.teacher}</div>
      `;
      dayDiv.appendChild(div);
    });
    app.appendChild(dayDiv);
  });
}
async function start() {
  try {
    schedule = await loadData();
    if (Object.keys(schedule).length === 0) {
      document.getElementById("app").innerHTML =
        "<div class='error'>В таблице нет данных. Проверь строки.</div>";
      return;
    }
    fillGroups();
    render();
  } catch (error) {
    document.getElementById("app").innerHTML =
      `<div class="error">Ошибка: ${error.message}</div>`;
  }
}
start();
</script>
</body>
</html>
