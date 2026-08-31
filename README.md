<!DOCTYPE html>
<html lang="en" data-phosphor="amber">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="color-scheme" content="dark">
<title>Abhishek Wali — terminal</title>
<style>
  :root{
    --amber:#ffb454;
    --green:#59ff9c;
    --paper:#eef0e2;
    --phosphor:var(--amber);
  }
  :root[data-phosphor="green"]{ --phosphor:var(--green); }
  :root[data-phosphor="white"]{ --phosphor:var(--paper); }

  *{ box-sizing:border-box; }
  html,body{ height:100%; }
  body{
    margin:0;
    min-height:100vh;
    display:flex;
    flex-direction:column;
    align-items:center;
    justify-content:center;
    gap:16px;
    padding:32px 16px;
    background: radial-gradient(ellipse at 50% 30%, #171310 0%, #060504 60%, #020202 100%);
    font-family: ui-monospace, "SF Mono", "Cascadia Code", "Courier New", Courier, monospace;
  }

  .sr-only{ position:absolute; width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0; }

  .monitor{
    width:min(860px, 100%);
    border-radius: clamp(16px, 3vw, 26px);
    padding: clamp(16px, 2.4vw, 26px) clamp(16px, 2.4vw, 26px) 16px;
    background: linear-gradient(155deg, #3c3325, #241f17 45%, #110e09);
    box-shadow:
      0 40px 70px -25px rgba(0,0,0,.7),
      inset 0 1px 0 rgba(255,255,255,.07),
      inset 0 -2px 6px rgba(0,0,0,.55);
    position:relative;
  }

  .screen-frame{
    background:#020101;
    border-radius: clamp(10px,1.6vw,16px);
    padding:10px;
    box-shadow: inset 0 0 0 2px #000, inset 0 2px 24px rgba(0,0,0,.9);
  }

  .screen{
    position:relative;
    border-radius:8px;
    overflow:hidden;
    background: radial-gradient(ellipse at 50% 35%, #0b0805 0%, #000 78%);
    animation: flicker 6.5s infinite;
  }
  .monitor.off .screen{ animation:none; background:#020101; }
  .screen.powering{ animation: powerOn .55s ease-out; }

  @keyframes powerOn{
    0%{ transform:scaleY(.02); filter:brightness(3); opacity:.2; }
    45%{ transform:scaleY(1); filter:brightness(2.1); opacity:1; }
    100%{ transform:scaleY(1); filter:brightness(1); opacity:1; }
  }
  @keyframes flicker{
    0%,100%{ opacity:1; }
    92%{ opacity:1; }
    93%{ opacity:.94; }
    94%{ opacity:1; }
    97%{ opacity:.97; }
  }

  .scanlines{
    position:absolute; inset:0; z-index:3; pointer-events:none;
    background: repeating-linear-gradient(to bottom, rgba(0,0,0,0) 0px, rgba(0,0,0,0) 1px, rgba(0,0,0,.35) 2px, rgba(0,0,0,0) 3px);
  }
  .vignette{
    position:absolute; inset:0; z-index:4; pointer-events:none;
    background: radial-gradient(ellipse at 50% 45%, rgba(0,0,0,0) 55%, rgba(0,0,0,.55) 100%);
  }
  #matrix-canvas{
    position:absolute; inset:0; z-index:1; opacity:0; pointer-events:none;
    transition: opacity .35s ease;
  }

  .crt-content{
    position:relative; z-index:2;
    display:flex; flex-direction:column;
    height:min(56vh, 400px);
    padding: clamp(14px,2.2vw,22px);
    color: var(--phosphor);
    text-shadow: 0 0 2px currentColor, 0 0 10px currentColor;
    font-size: clamp(12.5px, 1.6vw, 15px);
    line-height:1.55;
    opacity:0;
    transition: opacity .25s ease;
  }
  .monitor.ready .crt-content{ opacity:1; }
  .monitor.off .crt-content{ opacity:0 !important; }

  .output{ flex:1; min-height:0; overflow-y:auto; scrollbar-width:thin; }
  .output::-webkit-scrollbar{ width:8px; }
  .output::-webkit-scrollbar-thumb{ background: rgba(255,255,255,.15); border-radius:4px; }

  .line{ white-space:pre-wrap; word-break:break-word; }
  .line.err{ opacity:.75; }
  .out-block{ margin:2px 0 10px; font:inherit; color:inherit; white-space:pre-wrap; word-break:break-word; }
  .out-block a{ color:inherit; text-decoration:underline; text-underline-offset:2px; }
  .out-block a:hover{ opacity:.8; }

  .input-line{
    display:flex; align-items:center; gap:8px;
    padding-top:8px; flex-shrink:0;
    opacity:0; transition:opacity .3s ease .1s;
  }
  .monitor.ready .input-line{ opacity:1; }
  .prompt{ white-space:nowrap; }
  .cmd-input{
    flex:1; min-width:0; padding:0; margin:0;
    background:transparent; border:none; outline:none; appearance:none; -webkit-appearance:none;
    font:inherit; color:var(--phosphor); text-shadow:inherit;
    caret-color: var(--phosphor);
  }
  .cmd-input::selection{ background: var(--phosphor); color:#000; }
  .cmd-input:disabled{ opacity:.4; }

  .control-panel{
    display:flex; align-items:center; justify-content:space-between;
    gap:12px; padding:14px 4px 2px; flex-wrap:wrap;
  }
  .power-btn{
    appearance:none; -webkit-appearance:none;
    width:36px; height:36px; border-radius:50%;
    display:flex; align-items:center; justify-content:center;
    background: radial-gradient(circle at 35% 30%, #4a4030, #1c1710);
    border:1px solid rgba(0,0,0,.6);
    box-shadow: inset 0 1px 1px rgba(255,255,255,.15), 0 2px 4px rgba(0,0,0,.5);
    cursor:pointer; flex-shrink:0; padding:0;
  }
  .power-led{
    width:8px; height:8px; border-radius:50%;
    background: var(--phosphor);
    box-shadow: 0 0 6px var(--phosphor), 0 0 2px var(--phosphor);
    transition: background .3s, box-shadow .3s;
  }
  .monitor.off .power-led{ background:#3a352c; box-shadow:none; }

  .brand-plate{
    font-family: system-ui,-apple-system,"Segoe UI",sans-serif;
    font-size:10.5px; letter-spacing:.14em; text-transform:uppercase;
    color:#8c8272; text-align:center; flex:1; min-width:120px;
  }
  .brand-plate span{ display:block; color:#5f5748; font-size:9.5px; margin-top:2px; }

  .knob-group{ display:flex; gap:4px; flex-shrink:0; }
  .knob{
    appearance:none; -webkit-appearance:none;
    font-family: system-ui,-apple-system,"Segoe UI",sans-serif;
    font-size:9.5px; letter-spacing:.08em;
    padding:6px 8px; border-radius:5px;
    background:#1a160f; color:#7c7362;
    border:1px solid rgba(0,0,0,.6);
    cursor:pointer;
  }
  .knob.active[data-color="amber"]{ color:#ffb454; border-color:#ffb454; box-shadow:0 0 6px rgba(255,180,84,.45); }
  .knob.active[data-color="green"]{ color:#59ff9c; border-color:#59ff9c; box-shadow:0 0 6px rgba(89,255,156,.45); }
  .knob.active[data-color="white"]{ color:#eef0e2; border-color:#eef0e2; box-shadow:0 0 6px rgba(238,240,226,.4); }

  button:focus-visible, .cmd-input:focus-visible{ outline:2px solid var(--phosphor); outline-offset:2px; }

  .hint{ font-family: system-ui,-apple-system,"Segoe UI",sans-serif; font-size:12.5px; color:#7c7566; text-align:center; margin:0; }
  .hint code{ font-family:inherit; color:#a89c85; }
  .quote{ font-family: system-ui,-apple-system,"Segoe UI",sans-serif; font-size:12px; color:#5b5548; letter-spacing:.03em; font-style:italic; text-align:center; margin:-6px 0 0; }

  @media (prefers-reduced-motion: reduce){
    .screen, .screen.powering{ animation:none !important; }
    *{ transition-duration:.01ms !important; }
  }
</style>
</head>
<body>
  <main class="monitor" id="monitor">
    <div class="screen-frame">
      <div class="screen" id="screen">
        <canvas id="matrix-canvas" aria-hidden="true"></canvas>
        <div class="scanlines" aria-hidden="true"></div>
        <div class="vignette" aria-hidden="true"></div>
        <div class="crt-content">
          <label for="cmd-input" class="sr-only">Terminal command input</label>
          <div class="output" id="output" role="log" aria-live="polite"></div>
          <div class="input-line" id="input-line">
            <span class="prompt">guest@awali:~$</span>
            <input id="cmd-input" class="cmd-input" type="text" autocomplete="off" autocapitalize="off" autocorrect="off" spellcheck="false" disabled>
          </div>
        </div>
      </div>
    </div>
    <div class="control-panel">
      <button type="button" class="power-btn" id="power-btn" aria-label="Toggle power" aria-pressed="true">
        <span class="power-led" id="power-led"></span>
      </button>
      <div class="brand-plate">A.W. Terminal<span>Model 7 Mk.II</span></div>
      <div class="knob-group" role="group" aria-label="Phosphor color">
        <button type="button" class="knob active" data-color="amber" aria-pressed="true">Amber</button>
        <button type="button" class="knob" data-color="green" aria-pressed="false">Green</button>
        <button type="button" class="knob" data-color="white" aria-pressed="false">White</button>
      </div>
    </div>
  </main>
  <p class="hint">Click the screen and start typing — try <code>help</code></p>
  <p class="quote">"Build things that matter."</p>

<script>
(function(){
  const monitor = document.getElementById('monitor');
  const screenEl = document.getElementById('screen');
  const output = document.getElementById('output');
  const input = document.getElementById('cmd-input');
  const powerBtn = document.getElementById('power-btn');
  const knobs = Array.from(document.querySelectorAll('.knob'));
  const canvas = document.getElementById('matrix-canvas');
  const ctx = canvas.getContext('2d');

  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  let history = [];
  let historyIndex = 0;
  let matrixOn = false;
  let matrixRAF = null;
  let matrixDrops = [];
  let poweredOn = false;
  let booting = false;

  function sleep(ms){ return new Promise(function(r){ setTimeout(r, reduceMotion ? 0 : ms); }); }

  function scrollToEnd(){ output.scrollTop = output.scrollHeight; }

  function printLine(text, cls){
    const div = document.createElement('div');
    div.className = 'line' + (cls ? ' ' + cls : '');
    div.textContent = text;
    output.appendChild(div);
    scrollToEnd();
    return div;
  }

  function printBlock(lines){
    const pre = document.createElement('pre');
    pre.className = 'out-block';
    pre.textContent = lines.join('\n');
    output.appendChild(pre);
    scrollToEnd();
  }

  function printLinks(rows){
    const pre = document.createElement('pre');
    pre.className = 'out-block';
    rows.forEach(function(rowData){
      const label = rowData[0], url = rowData[1], shown = rowData[2];
      pre.appendChild(document.createTextNode(label.padEnd(11, ' ')));
      const a = document.createElement('a');
      a.href = url;
      a.target = '_blank';
      a.rel = 'noopener noreferrer';
      a.textContent = shown;
      pre.appendChild(a);
      pre.appendChild(document.createTextNode('\n'));
    });
    output.appendChild(pre);
    scrollToEnd();
  }

  async function typeLine(text, speed){
    const div = document.createElement('div');
    div.className = 'line';
    output.appendChild(div);
    if (reduceMotion){ div.textContent = text; scrollToEnd(); return; }
    for (let i = 0; i <= text.length; i++){
      div.textContent = text.slice(0, i);
      if (i % 2 === 0) scrollToEnd();
      await sleep(speed);
    }
    scrollToEnd();
  }

  function dotLine(label, status, width){
    width = width || 20;
    const dots = '.'.repeat(Math.max(3, width - label.length));
    return label + dots + ' ' + status;
  }

  function row(label, value, width){
    width = width || 12;
    return label.padEnd(width, ' ') + value;
  }

  const bootLines = [
    'A.W. TERMINAL — MODEL 7 MK.II',
    '(c) 2026 · built with coffee and curiosity',
    '',
    'running power-on self test...',
    '',
    dotLine('CORE LANGUAGES', 'OK'),
    dotLine('FULL-STACK', 'OK'),
    dotLine('AI / LLM', 'OK'),
    dotLine('MENTOR (120+)', 'OK'),
    dotLine('COFFEE', 'OK'),
    '',
    'loading profile: abhishek_wali.dat ... done',
    'session ready.',
    '',
    "type 'help' to look around."
  ];

  async function boot(){
    if (booting) return;
    booting = true;
    monitor.classList.remove('off');
    output.innerHTML = '';
    screenEl.classList.add('powering');
    setTimeout(function(){ screenEl.classList.remove('powering'); }, 600);
    for (const line of bootLines){
      if (line === ''){ printLine(''); await sleep(150); continue; }
      await typeLine(line, 9);
      await sleep(60);
    }
    monitor.classList.add('ready');
    poweredOn = true;
    booting = false;
    input.disabled = false;
    input.focus();
  }

  function help(){
    printBlock([
      'available commands',
      '-------------------',
      'about        who I am and what I care about',
      'skills       tech stack, by category',
      'focus        what I\'m building / learning now',
      'philosophy   the loop I try to run on',
      'contact      ways to reach me',
      'whoami       quick identity check',
      'matrix       ...you know why',
      'phosphor     amber / green / white',
      'power        turn the monitor off (or on)',
      'clear        clear the screen'
    ]);
  }

  function about(){
    printBlock([
      'about',
      '-----',
      'Software engineer focused on scalable backend',
      'systems, AI-powered applications, and modern',
      'web products.',
      '',
      'cares about:',
      ' > backend engineering',
      ' > AI & LLMs',
      ' > system design',
      ' > developer tools',
      ' > machine learning',
      ' > mentoring developers'
    ]);
  }

  function skills(){
    printBlock([
      'tech stack',
      '----------',
      row('Languages', 'Java, JavaScript, TypeScript, Python, SQL'),
      row('Frontend', 'React, HTML5, CSS3'),
      row('Backend', 'Node.js, Express'),
      row('Database', 'MongoDB, PostgreSQL, Supabase'),
      row('Tools', 'Git, Docker, Linux, GitHub Actions')
    ]);
  }

  function focus(){
    printBlock([
      'current focus',
      '-------------',
      row('Learning', 'System design', 14),
      row('Building', 'AI developer tools', 14),
      row('Practicing', 'DSA', 14),
      row('Working with', 'Node.js, React, TypeScript', 14),
      row('Exploring', 'LLMs & agents', 14)
    ]);
  }

  function philosophy(){
    printBlock([
      'philosophy',
      '----------',
      'while (alive) {',
      '    Learn();',
      '    Build();',
      '    Share();',
      '    Repeat();',
      '}',
      '',
      '>> currently on iteration #\u221e'
    ]);
  }

  function whoami(){
    printBlock([
      'guest',
      row('role', 'Software engineer / Full-stack developer / AI enthusiast / Mentor', 9),
      row('status', 'Currently building production-ready software.', 9)
    ]);
  }

  function contact(){
    printLine('contact');
    printLine('-------');
    printLinks([
      ['LinkedIn', 'https://www.linkedin.com/in/abhishek-wali-0628a524b', 'linkedin.com/in/abhishek-wali-0628a524b'],
      ['GitHub', 'https://github.com/Abhishekh3007', 'github.com/Abhishekh3007'],
      ['Portfolio', 'https://abhishekh-wali-portfolio.vercel.app/', 'abhishekh-wali-portfolio.vercel.app'],
      ['LeetCode', 'https://leetcode.com/u/Abhishek_wali/', 'leetcode.com/u/Abhishek_wali'],
      ['Email', 'mailto:waliabhishek120@gmail.com', 'waliabhishek120@gmail.com']
    ]);
  }

  function setPhosphor(color){
    const valid = ['amber','green','white'];
    if (valid.indexOf(color) === -1){
      printLine('!! usage: phosphor <amber|green|white>', 'err');
      return;
    }
    document.documentElement.setAttribute('data-phosphor', color);
    knobs.forEach(function(k){
      const on = k.dataset.color === color;
      k.classList.toggle('active', on);
      k.setAttribute('aria-pressed', String(on));
    });
    printLine('phosphor set to ' + color + '.');
  }

  function sizeCanvas(){
    const rect = screenEl.getBoundingClientRect();
    canvas.width = rect.width;
    canvas.height = rect.height;
    const cols = Math.floor(canvas.width / 15);
    matrixDrops = new Array(cols).fill(0).map(function(){ return Math.random() * -40; });
  }

  function currentPhosphor(){
    return getComputedStyle(document.documentElement).getPropertyValue('--phosphor').trim() || '#ffb454';
  }

  function drawMatrix(){
    ctx.fillStyle = 'rgba(0,0,0,0.09)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = currentPhosphor();
    ctx.font = '14px monospace';
    const chars = 'アカサタナ01ABHISHEK{}<>/;=+*';
    for (let i = 0; i < matrixDrops.length; i++){
      const ch = chars[Math.floor(Math.random() * chars.length)];
      ctx.fillText(ch, i * 15, matrixDrops[i] * 15);
      if (matrixDrops[i] * 15 > canvas.height && Math.random() > 0.975) matrixDrops[i] = 0;
      matrixDrops[i]++;
    }
    matrixRAF = requestAnimationFrame(drawMatrix);
  }

  function toggleMatrix(){
    matrixOn = !matrixOn;
    if (matrixOn){
      sizeCanvas();
      canvas.style.opacity = '0.5';
      drawMatrix();
      printLine("entering the matrix... (type 'matrix' again to leave)");
    } else {
      cancelAnimationFrame(matrixRAF);
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      canvas.style.opacity = '0';
      printLine('back to reality.');
    }
  }

  function powerOff(){
    poweredOn = false;
    monitor.classList.add('off');
    monitor.classList.remove('ready');
    input.disabled = true;
    if (matrixOn) toggleMatrix();
    powerBtn.setAttribute('aria-pressed', 'false');
  }

  async function powerOn(){
    powerBtn.setAttribute('aria-pressed', 'true');
    await boot();
  }

  function sudo(rest){
    if (rest.indexOf('make me a sandwich') !== -1){
      printLine('[sudo] password for guest:');
      printLine('permission granted.');
      printLine("still can't make sandwiches — try the kitchen.");
      return;
    }
    printLine('permission denied: guest is not in the sudoers file.', 'err');
    printLine('(this incident will not be reported. probably.)');
  }

  function runCommand(raw){
    const trimmed = raw.trim();
    const echo = document.createElement('div');
    echo.className = 'line';
    echo.textContent = 'guest@awali:~$ ' + trimmed;
    output.appendChild(echo);
    if (!trimmed){ scrollToEnd(); return; }

    history.push(trimmed);
    historyIndex = history.length;

    const parts = trimmed.split(/\s+/);
    const cmd = parts[0].toLowerCase();
    const rest = parts.slice(1).join(' ').toLowerCase();

    switch(cmd){
      case 'help': help(); break;
      case 'about': about(); break;
      case 'skills':
      case 'stack':
      case 'tech': skills(); break;
      case 'focus': focus(); break;
      case 'philosophy': philosophy(); break;
      case 'whoami': whoami(); break;
      case 'contact':
      case 'links': contact(); break;
      case 'clear': output.innerHTML = ''; break;
      case 'matrix': toggleMatrix(); break;
      case 'phosphor':
      case 'theme': setPhosphor(rest); break;
      case 'power': poweredOn ? powerOff() : powerOn(); break;
      case 'sudo': sudo(rest); break;
      case 'rm': printLine('nice try. this terminal backs up to /dev/null.', 'err'); break;
      default: printLine("!! command not found: " + cmd + " — try 'help'", 'err');
    }
    scrollToEnd();
  }

  input.addEventListener('keydown', function(e){
    if (e.key === 'Enter'){
      const val = input.value;
      input.value = '';
      runCommand(val);
    } else if (e.key === 'ArrowUp'){
      e.preventDefault();
      if (history.length){
        historyIndex = Math.max(0, historyIndex - 1);
        input.value = history[historyIndex] || '';
      }
    } else if (e.key === 'ArrowDown'){
      e.preventDefault();
      if (history.length){
        historyIndex = Math.min(history.length, historyIndex + 1);
        input.value = history[historyIndex] || '';
      }
    }
  });

  screenEl.addEventListener('click', function(){ if (!input.disabled) input.focus(); });
  powerBtn.addEventListener('click', function(){ poweredOn ? powerOff() : powerOn(); });
  knobs.forEach(function(k){ k.addEventListener('click', function(){ setPhosphor(k.dataset.color); }); });
  window.addEventListener('resize', function(){ if (matrixOn) sizeCanvas(); });

  boot();
})();
</script>
</body>
</html>
