# ucapan-selamat-ulang-tahun-
Ulang tahun 
<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Ucapan Selamat Ulang Tahun 🎉</title>
  <meta name="description" content="Kartu ucapan ulang tahun ibu winda dalam satu file HTML. Edit nama & pesan langsung atau via URL query (?to=ibu winda&from=Esa &msg=Pesan)." />
  <style>
    :root{
      --bg-1:#0ea5e9;/* sky-500 */
      --bg-2:#8b5cf6;/* violet-500 */
      --card-bg:rgba(255,255,255,.18);
      --card-border:rgba(255,255,255,.35);
      --txt:#0b1221;
      --accent:#f59e0b;/* amber */
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0;
      font-family: ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,Noto Sans,Helvetica,Arial,"Apple Color Emoji","Segoe UI Emoji";
      color:var(--txt);
      background: radial-gradient(1200px 800px at 10% 10%, #ffffff20, transparent 60%),
                  radial-gradient(1200px 800px at 90% 90%, #ffffff15, transparent 60%),
                  linear-gradient(135deg, var(--bg-1), var(--bg-2));
      overflow:hidden;
    }
    .wrap{
      position:relative; inset:0; height:100dvh; width:100%; display:grid; place-items:center;
    }
    /* Confetti canvas full-screen behind content */
    #confetti{
      position:absolute; inset:0; width:100%; height:100%; pointer-events:none; z-index:0;
    }
    /* Floating balloons */
    .balloons{position:absolute; inset:0; overflow:hidden; z-index:1; pointer-events:none}
    .balloon{position:absolute; bottom:-20vh; width:36px; aspect-ratio:3/4; border-radius:60% 60% 60% 60%;
      background:radial-gradient(40px 60px at 30% 30%, #fff8, #fff0 40%),
                 radial-gradient(80px 80px at 70% 60%, #0001, #0000 50%),
                 var(--accent);
      box-shadow: inset 0 10px 20px #fff5, inset 0 -10px 20px #0002;
      animation: float var(--dur) linear infinite; filter:saturate(1.1);
    }
    .balloon:after{content:""; position:absolute; bottom:-14px; left:50%; width:2px; height:14px; background:#0003; transform:translateX(-50%)}
    @keyframes float{
      0%{transform:translateY(0) translateX(0) rotate(0)}
      100%{transform:translateY(-120vh) translateX(var(--drift, 0)) rotate(8deg)}
    }

    /* Card */
    .card{
      position:relative; z-index:2; max-width:720px; width:clamp(320px, 80vw, 720px);
      background:var(--card-bg); backdrop-filter: blur(16px) saturate(120%);
      -webkit-backdrop-filter: blur(16px) saturate(120%);
      border:1px solid var(--card-border); border-radius:24px;
      box-shadow: 0 30px 60px #00000030, inset 0 1px 0 #ffffff60;
      padding:28px; text-align:center;
    }
    .title{font-size:clamp(28px, 5vw, 52px); line-height:1.05; margin:0 0 12px; letter-spacing:.5px}
    .subtitle{font-size:clamp(16px, 2.6vw, 22px); opacity:.9; margin:0 0 22px}
    .name{display:inline-block; padding:.15em .5em; border-radius:12px; background:#fff8; border:1px solid #fff6}
    .message{font-size:clamp(14px, 2.2vw, 18px); margin:0 auto 20px; max-width:60ch}

    .row{display:flex; flex-wrap:wrap; gap:10px; justify-content:center; margin-top:8px}
    button, .ghost{
      appearance:none; border:1px solid #ffffff70; background:#ffffffb3; color:#1f2937; border-radius:999px; padding:10px 14px; font-weight:600; cursor:pointer;
      box-shadow:0 6px 16px #0000001f; transition:transform .1s ease, box-shadow .2s ease, background .2s ease;
    }
    button:hover{transform:translateY(-1px); box-shadow:0 10px 22px #0000002b}
    button:active{transform:translateY(0)}
    .ghost{background:transparent; color:#fff; border-color:#ffffff60}
    .footer{margin-top:14px; font-size:12px; opacity:.8}
    .footer a{color:#fff; font-weight:600}

    /* Toast */
    .toast{position:fixed; left:50%; bottom:24px; transform:translateX(-50%) translateY(20px); opacity:0; transition:.25s ease; background:#111; color:#fff; border-radius:10px; padding:10px 14px; z-index:99}
    .toast.show{opacity:1; transform:translateX(-50%) translateY(0)}

    /* Responsive safe area */
    @supports (padding:max(0px)){
      .card{padding: max(28px, env(safe-area-inset-top));}
    }
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="confetti"></canvas>
    <div class="balloons" id="balloons" aria-hidden="true"></div>

    <main class="card" role="main" aria-label="Kartu Ucapan Ulang Tahun">
      <h1 class="title">Selamat Ulang Tahun, <span id="to" class="name">Ibu winda</span>! 🎂🎉</h1>
      <p class="subtitle">Semoga di hari spesial ini semua doa terbaik untuk ibu.</p>
      <p id="msg" class="message">Semoga panjang umur, sehat selalu, dimudahkan rezeki, dan bahagia setiap hari. Terima kasih . 💖</p>
      <p id="from" class="message" style="opacity:.9"><em>— Dari: <strong>Temanmu</strong></em></p>

      <div class="row" role="group" aria-label="Aksi kartu">
        <button id="edit"></button>
        <button id="share"></button>
        <button id="replay"> </button>
        <button id="palette" class="ghost" title="Ganti warna tema">Ganti Tema</button>
      </div>
      <div class="footer">Tip: Anda juga bisa memasukkan parameter di URL, misalnya <code>!</code></div>
    </main>
  </div>

  <div class="toast" id="toast" role="status" aria-live="polite">Tersalin ke clipboard!</div>

  <script>
    // ===== Helper: URL params & state =====
    const $ = (s, p=document) => p.querySelector(s);
    const params = new URLSearchParams(location.search);
    const state = {
      to: params.get('to') || 'Ibu Winda',
      from: params.get('from') || 'Esa',
      msg: params.get('msg') || 'Semoga panjang umur, sehat selalu, dimudahkan rezeki, dan bahagia setiap hari. Terima kasih . 💖',
      theme: Number(params.get('theme') || 0) % 4,
    };

    function applyState(){
      $('#to').textContent = state.to;
      $('#from').innerHTML = `<em>— Dari: <strong>${escapeHtml(state.from)}</strong></em>`;
      $('#msg').textContent = state.msg;
      setTheme(state.theme);
      history.replaceState({}, '', buildUrlWithParams());
    }

    function buildUrlWithParams(){
      const q = new URLSearchParams({to: state.to, from: state.from, msg: state.msg, theme: String(state.theme)});
      return location.pathname + '?' + q.toString();
    }

    function escapeHtml(str){
      return String(str).replace(/[&<>"']/g, s => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;','\'':'&#39;'}[s]));
    }

    // ===== Confetti =====
    const canvas = $('#confetti');
    const ctx = canvas.getContext('2d');
    let confettiPieces = [];

    function resizeCanvas(){
      const dpr = Math.max(1, Math.min(2, window.devicePixelRatio || 1));
      canvas.width = Math.floor(innerWidth * dpr);
      canvas.height = Math.floor(innerHeight * dpr);
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    }

    function createConfettiBurst(n=180){
      const colors = ['#f43f5e','#f59e0b','#84cc16','#22d3ee','#8b5cf6','#eab308'];
      for(let i=0;i<n;i++){
        confettiPieces.push({
          x: Math.random()*innerWidth,
          y: -10,
          w: 8+Math.random()*6,
          h: 6+Math.random()*8,
          r: Math.random()*Math.PI,
          vr: (Math.random()-.5)*0.3,
          vx: (Math.random()-.5)*3,
          vy: 2+Math.random()*3,
          color: colors[(Math.random()*colors.length)|0],
          life: 220+Math.random()*120
        });
      }
    }

    function step(){
      ctx.clearRect(0,0,innerWidth,innerHeight);
      confettiPieces = confettiPieces.filter(p=> p.life>0);
      for(const p of confettiPieces){
        p.x += p.vx; p.y += p.vy; p.r += p.vr; p.vy += 0.02; p.life--;
        if(p.y>innerHeight+20) p.life=0;
        ctx.save();
        ctx.translate(p.x, p.y); ctx.rotate(p.r);
        ctx.fillStyle = p.color; ctx.fillRect(-p.w/2,-p.h/2,p.w,p.h);
        ctx.restore();
      }
      requestAnimationFrame(step);
    }

    // ===== Balloons =====
    function spawnBalloons(){
      const host = $('#balloons');
      const palette = ['#ef4444','#f97316','#10b981','#0ea5e9','#8b5cf6','#eab308','#ec4899'];
      for(let i=0;i<14;i++){
        const b = document.createElement('div');
        b.className = 'balloon';
        b.style.left = (Math.random()*100)+'%';
        b.style.setProperty('--dur', (18+Math.random()*14)+'s');
        b.style.setProperty('--drift', (Math.random()*.6 - .3)*50 + 'vw');
        b.style.background = `radial-gradient(40px 60px at 30% 30%, #fff8, #fff0 40%),
                               radial-gradient(80px 80px at 70% 60%, #0001, #0000 50%),
                               ${palette[(Math.random()*palette.length)|0]}`;
        host.appendChild(b);
      }
    }

    // ===== Themes =====
    const THEMES = [
      {bg1:'#0ea5e9', bg2:'#8b5cf6', accent:'#f59e0b'}, // sky → violet
      {bg1:'#ec4899', bg2:'#ef4444', accent:'#22d3ee'}, // pink → red
      {bg1:'#10b981', bg2:'#22d3ee', accent:'#eab308'}, // green → cyan
      {bg1:'#1f2937', bg2:'#111827', accent:'#f59e0b'}, // dark neutral
    ];
    function setTheme(i){
      const t = THEMES[i%THEMES.length];
      document.documentElement.style.setProperty('--bg-1', t.bg1);
      document.documentElement.style.setProperty('--bg-2', t.bg2);
      document.documentElement.style.setProperty('--accent', t.accent);
    }

    // ===== Share / Copy =====
    async function sharePage(){
      const url = location.origin ? new URL(buildUrlWithParams(), location.origin).toString() : buildUrlWithParams();
      const text = `Selamat ulang tahun untuk ${state.to}!`;
      if(navigator.share){
        try{ await navigator.share({title:document.title, text, url}); }
        catch(e){ /* user canceled */ }
      } else {
        await navigator.clipboard.writeText(url);
        showToast('Tautan tersalin!');
      }
    }

    function showToast(msg){
      const t = $('#toast');
      t.textContent = msg; t.classList.add('show');
      setTimeout(()=> t.classList.remove('show'), 1600);
    }

    // ===== Edit dialog (prompt sederhana agar tetap 1 file) =====
    function editContent(){
      const to = prompt('Nama penerima', state.to);
      if(to===null) return; // cancel
      const msg = prompt('Pesan', state.msg);
      if(msg===null) return;
      const from = prompt('Dari siapa?', state.from);
      if(from===null) return;
      state.to = to.trim() || state.to;
      state.msg = msg.trim() || state.msg;
      state.from = from.trim() || state.from;
      applyState();
      createConfettiBurst(220);
    }

    // ===== Wire up UI =====
    $('#edit').addEventListener('click', editContent);
    $('#share').addEventListener('click', sharePage);
    $('#replay').addEventListener('click', ()=> createConfettiBurst(260));
    $('#palette').addEventListener('click', ()=>{ state.theme=(state.theme+1)%THEMES.length; applyState(); });

    // ===== Init =====
    addEventListener('resize', resizeCanvas);
    resizeCanvas();
    spawnBalloons();
    applyState();
    createConfettiBurst(320);
    step();
  </script>
</body>
</html>
