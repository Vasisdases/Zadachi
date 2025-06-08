# Zadachi
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Решение задачи о назначениях</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 2rem; background: #f2f2f2; }
    h1 { color: #333; }
    table { border-collapse: collapse; margin: 1rem 0; }
    td, th { border: 1px solid #999; padding: 0.5rem 1rem; text-align: center; }
    textarea { width: 100%; height: 150px; margin-top: 1rem; font-family: monospace; }
    .output { white-space: pre-wrap; background: #fff; padding: 1rem; border: 1px solid #ccc; }
    button { padding: 0.6rem 1.2rem; margin-top: 1rem; font-size: 1rem; }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/munkres-js@1.0.3/munkres.min.js"></script>
</head>
<body>
  <h1>Решение задачи о назначениях</h1>

  <p>Введите матрицу затрат (по строкам, значения разделяйте пробелами, строки — переводом строки):</p>
  <textarea id="matrixInput" placeholder="Например:
7 5 12 1 11
6 6 13 6 10
10 2 13 7 12
6 7 11 3 5
6 4 4 5 7"></textarea>
  <br>
  <label><input type="checkbox" id="maximize"> Задача на максимум</label><br>
  <button onclick="solveAssignment()">Решить</button>

  <div class="output" id="result"></div>

  <script>
    function solveAssignment() {
      const input = document.getElementById('matrixInput').value.trim();
      const maximize = document.getElementById('maximize').checked;
      const lines = input.split('\n');
      const matrix = lines.map(line => line.trim().split(/\s+/).map(Number));

      if (!matrix.every(row => row.length === matrix.length)) {
        document.getElementById('result').textContent = 'Ошибка: матрица должна быть квадратной.';
        return;
      }

      const n = matrix.length;
      const original = matrix.map(row => row.slice());
      const maxVal = Math.max(...matrix.flat());
      const cost = maximize
        ? matrix.map(row => row.map(val => maxVal - val))
        : matrix.map(row => row.slice());

      const result = [];
      result.push('Шаг 1: Исходная матрица затрат:\n');
      result.push(formatTable(original));

      result.push('\nШаг 2: Редукция по строкам:');
      for (let i = 0; i < n; i++) {
        const min = Math.min(...cost[i]);
        result.push(`  Строка ${i + 1}: минимум = ${min}`);
        for (let j = 0; j < n; j++) cost[i][j] -= min;
      }
      result.push('\nПосле редукции по строкам:\n' + formatTable(cost));

      result.push('\nШаг 3: Редукция по столбцам:');
      for (let j = 0; j < n; j++) {
        let col = cost.map(row => row[j]);
        const min = Math.min(...col);
        result.push(`  Столбец ${j + 1}: минимум = ${min}`);
        for (let i = 0; i < n; i++) cost[i][j] -= min;
      }
      result.push('\nПосле редукции по столбцам:\n' + formatTable(cost));

      const munkres = new Munkres();
      const indices = munkres.compute(cost);
      let totalCost = 0;
      const assignments = indices.map(([i, j]) => {
        totalCost += original[i][j];
        return `  Рабочий ${i + 1} — Задание ${j + 1} (стоимость ${original[i][j]})`;
      });

      result.push('\nШаг 4: Оптимальное назначение:');
      result.push(assignments.join('\n'));
      result.push(`\nСуммарные затраты: ${totalCost}`);

      document.getElementById('result').textContent = result.join('\n');
    }

    function formatTable(matrix) {
      return matrix.map(row => row.map(v => v.toString().padStart(4)).join('')).join('\n');
    }
  </script>
</body>
</html>
