<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Calculadora Simples</title>
  <style>
    :root{
      --bg:#0f1724;
      --panel:#0b1220;
      --accent:#06b6d4;
      --btn:#1f2937;
      --btn-ops:#334155;
      --text:#e6eef6;
      --success:#10b981;
      --danger:#ef4444;
    }
    *{box-sizing:border-box}
    body{
      margin:0; min-height:100vh;
      display:flex; align-items:center; justify-content:center;
      font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;
      background:linear-gradient(180deg,var(--bg),#061024);
      color:var(--text);
      padding:24px;
    }

    .calc {
      width:360px;
      max-width:95vw;
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(0,0,0,0.05));
      border-radius:16px;
      padding:18px;
      box-shadow:0 10px 30px rgba(2,6,23,0.6);
      border:1px solid rgba(255,255,255,0.03);
    }

    .display {
      height:72px;
      background:var(--panel);
      border-radius:12px;
      padding:12px;
      display:flex;
      flex-direction:column;
      justify-content:center;
      align-items:flex-end;
      gap:6px;
      margin-bottom:14px;
      overflow:hidden;
    }

    .history {
      font-size:13px;
      opacity:0.6;
      white-space:nowrap;
      text-overflow:ellipsis;
      overflow:hidden;
      width:100%;
    }

    .output {
      font-size:28px;
      font-weight:600;
      width:100%;
      word-break:break-all;
      text-align:right;
    }

    .grid {
      display:grid;
      grid-template-columns: repeat(4,1fr);
      gap:10px;
    }

    button {
      height:56px;
      border-radius:10px;
      border:0;
      background:var(--btn);
      color:var(--text);
      font-size:18px;
      cursor:pointer;
      box-shadow: 0 4px 12px rgba(2,6,23,0.5);
      transition:transform .05s ease, box-shadow .12s ease, opacity .08s;
      user-select:none;
    }
    button:active{ transform: translateY(1px) scale(.998); }
    button:focus{ outline:2px solid rgba(6,182,212,0.14); }

    .span-2 { grid-column: span 2; }
    .op { background:var(--btn-ops); font-weight:600; }
    .equal { background:linear-gradient(180deg,var(--accent), #028a9a); color:#022; font-weight:700; }
    .clear { background:linear-gradient(180deg,#ef4444,#e11d48); }
    .del { background:linear-gradient(180deg,#f97316,#f59e0b); }

    /* small responsiveness */
    @media (max-width:420px){
      .display{ height:64px }
      .output{ font-size:24px }
      button{ height:52px; font-size:17px }
    }
  </style>
</head>
<body>
  <main class="calc" role="application" aria-label="Calculadora">
    <div class="display" aria-live="polite">
      <div class="history" id="history" aria-hidden="true"></div>
      <div class="output" id="output">0</div>
    </div>

    <div class="grid" role="group" aria-label="Teclado da calculadora">
      <button class="clear span-2" data-action="clear">C</button>
      <button class="del" data-action="del">DEL</button>
      <button class="op" data-value="/">÷</button>

      <button data-value="7">7</button>
      <button data-value="8">8</button>
      <button data-value="9">9</button>
      <button class="op" data-value="*">×</button>

      <button data-value="4">4</button>
      <button data-value="5">5</button>
      <button data-value="6">6</button>
      <button class="op" data-value="-">−</button>

      <button data-value="1">1</button>
      <button data-value="2">2</button>
      <button data-value="3">3</button>
      <button class="op" data-value="+">+</button>

      <button data-value="0" class="span-2">0</button>
      <button data-value=".">.</button>
      <button class="equal" data-action="equals">=</button>

      <button class="op span-2" data-action="percent">%</button>
      <button class="op" data-value="(">(</button>
      <button class="op" data-value=")">)</button>
    </div>
  </main>

  <script>
    (function(){
      const outputEl = document.getElementById('output');
      const historyEl = document.getElementById('history');

      let expression = ''; // expressão atual mostrada

      function updateDisplay(){
        outputEl.textContent = expression === '' ? '0' : expression;
      }

      function appendValue(val){
        // evitar dois pontos decimais seguidos como "."
        expression += val;
        updateDisplay();
      }

      function clearAll(){
        expression = '';
        historyEl.textContent = '';
        updateDisplay();
      }

      function deleteLast(){
        expression = expression.slice(0, -1);
        updateDisplay();
      }

      function applyPercent(){
        // transforma o último número na expressão em (n / 100)
        // exemplo: "200+50" => encontra "50" e substitui por "(50/100)"
        const re = /(\d+(\.\d+)?)(?!.*\d)/; // último número (grosso)
        const match = expression.match(re);
        if (!match) return;
        const num = match[0];
        const replaced = `(${num}/100)`;
        expression = expression.slice(0, match.index) + replaced;
        updateDisplay();
      }

      function sanitizeExpression(expr){
        // permite apenas dígitos, espaços, operadores básicos e parênteses
        // ponto para casas decimais; proibir outros caracteres
        const allowed = /^[0-9+\-*/().\s]+$/;
        return allowed.test(expr);
      }

      function calculate(){
        const expr = expression.trim();
        if (!expr) return;
        if (!sanitizeExpression(expr)){
          historyEl.textContent = 'Entrada inválida';
          return;
        }
        try {
          // eslint-disable-next-line no-new-func
          const result = Function('"use strict"; return (' + expr + ')')();
          if (typeof result === 'number' && isFinite(result)) {
            historyEl.textContent = expr + ' =';
            expression = String(result);
            updateDisplay();
          } else {
            historyEl.textContent = 'Resultado inválido';
          }
        } catch(e){
          historyEl.textContent = 'Erro na expressão';
        }
      }

      // Delegação de clique
      document.addEventListener('click', (ev) => {
        const btn = ev.target.closest('button');
        if (!btn) return;
        const v = btn.dataset.value;
        const action = btn.dataset.action;

        if (action === 'clear') { clearAll(); return; }
        if (action === 'del') { deleteLast(); return; }
        if (action === 'percent') { applyPercent(); return; }
        if (action === 'equals') { calculate(); return; }

        if (v !== undefined) {
          appendValue(v);
        }
      });

      // Suporte a teclado físico
      window.addEventListener('keydown', (ev) => {
        // permitir teclas: 0-9, + - * / . ( ) Enter Backspace Esc %
        const key = ev.key;
        if ((/^[0-9]$/).test(key)) { appendValue(key); ev.preventDefault(); return; }
        if (key === '+' || key === '-' || key === '*' || key === '/' ) { appendValue(key); ev.preventDefault(); return; }
        if (key === '.') { appendValue('.'); ev.preventDefault(); return; }
        if (key === '(' || key === ')') { appendValue(key); ev.preventDefault(); return; }
        if (key === 'Enter' || key === '=') { calculate(); ev.preventDefault(); return; }
        if (key === 'Backspace') { deleteLast(); ev.preventDefault(); return; }
        if (key === 'Escape') { clearAll(); ev.preventDefault(); return; }
        if (key === '%') { applyPercent(); ev.preventDefault(); return; }
      });

      // Inicializa
      clearAll();
    })();
  </script>
</body>
</html># calculadora
Para fazer calculos simples
