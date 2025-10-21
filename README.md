<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Отчет по правкам</title>
  <script src="https://p.trellocdn.com/power-up.min.js"></script>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
      padding: 20px;
      background: #f4f5f7;
    }
    h2 {
      color: #172b4d;
      margin-bottom: 20px;
    }
    #report {
      background: white;
      border-radius: 8px;
      padding: 20px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
    }
    th, td {
      padding: 12px;
      text-align: left;
      border-bottom: 1px solid #e0e0e0;
    }
    th {
      background: #0079bf;
      color: white;
      font-weight: 600;
    }
    tr:hover {
      background: #f9f9f9;
    }
    .stats {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 15px;
      margin-bottom: 20px;
    }
    .stat-card {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 15px;
      border-radius: 8px;
      text-align: center;
    }
    .stat-value {
      font-size: 32px;
      font-weight: bold;
      margin: 5px 0;
    }
    .stat-label {
      font-size: 14px;
      opacity: 0.9;
    }
    button {
      background: #0079bf;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 14px;
      margin-bottom: 15px;
    }
    button:hover {
      background: #026aa7;
    }
    .loading {
      text-align: center;
      padding: 40px;
      color: #666;
    }
  </style>
</head>
<body>
  <h2>📊 Отчет по правкам</h2>
  <button onclick="generateReport()">🔄 Обновить отчет</button>
  <div id="report">
    <div class="loading">Загрузка данных...</div>
  </div>

  <script>
    const t = window.TrelloPowerUp.iframe();

    async function generateReport() {
      const reportDiv = document.getElementById('report');
      reportDiv.innerHTML = '<div class="loading">Формирование отчета...</div>';

      try {
        // Получаем данные через Trello API (без API ключа!)
        const board = await t.board('id', 'customFields');
        const cards = await t.cards('id', 'name', 'shortUrl', 'idList', 'customFieldItems');
        const lists = await t.lists('id', 'name');
        
        // Получаем кастомные поля доски
        const customFields = board.customFields || [];
        
        // Находим поле "Правки"
        const pravkiField = customFields.find(field => 
          field.name.toLowerCase().includes('правки')
        );

        if (!pravkiField) {
          reportDiv.innerHTML = '<p style="color: red;">❌ Кастомное поле "Правки" не найдено на доске</p>';
          return;
        }

        // Создаем карту списков
        const listsMap = {};
        lists.forEach(list => listsMap[list.id] = list.name);

        // Собираем данные
        const reportData = [];
        let totalPravki = 0;

        cards.forEach(card => {
          const customFieldItem = card.customFieldItems?.find(
            item => item.idCustomField === pravkiField.id
          );

          if (customFieldItem) {
            const pravkiValue = customFieldItem.value?.number || 
                               Number(customFieldItem.value?.text) || 
                               0;

            if (pravkiValue > 0) {
              reportData.push({
                name: card.name,
                pravki: pravkiValue,
                url: card.shortUrl,
                list: listsMap[card.idList] || 'Неизвестно'
              });
              totalPravki += pravkiValue;
            }
          }
        });

        // Сортируем по убыванию правок
        reportData.sort((a, b) => b.pravki - a.pravki);

        // Формируем HTML отчет
        const avgPravki = reportData.length > 0 ? (totalPravki / reportData.length).toFixed(2) : 0;
        
        let html = `
          <div class="stats">
            <div class="stat-card">
              <div class="stat-label">Всего правок</div>
              <div class="stat-value">${totalPravki}</div>
            </div>
            <div class="stat-card">
              <div class="stat-label">Карточек с правками</div>
              <div class="stat-value">${reportData.length}</div>
            </div>
            <div class="stat-card">
              <div class="stat-label">Среднее</div>
              <div class="stat-value">${avgPravki}</div>
            </div>
          </div>
          <table>
            <thead>
              <tr>
                <th>Карточка</th>
                <th>Правок</th>
                <th>Список</th>
                <th>Ссылка</th>
              </tr>
            </thead>
            <tbody>
        `;

        reportData.forEach(item => {
          html += `
            <tr>
              <td><strong>${item.name}</strong></td>
              <td><span style="background: #ff6b6b; color: white; padding: 4px 10px; border-radius: 12px; font-weight: bold;">${item.pravki}</span></td>
              <td>${item.list}</td>
              <td><a href="${item.url}" target="_blank" style="color: #0079bf;">Открыть</a></td>
            </tr>
          `;
        });

        html += '</tbody></table>';
        
        if (reportData.length === 0) {
          html = '<p style="text-align: center; padding: 40px; color: #999;">Карточек с правками не найдено</p>';
        }

        reportDiv.innerHTML = html;

      } catch (error) {
        reportDiv.innerHTML = `<p style="color: red;">Ошибка: ${error.message}</p>`;
        console.error('Error:', error);
      }
    }

    // Автозагрузка при открытии
    generateReport();
  </script>
</body>
</html>
