# MY-PR0JEC
<!--
File: index.html
Purpose: Night sky with falling stars animation. Put this file as the repo root (or in /docs) and enable GitHub Pages to host it (it will serve as your page).

Hosting steps (short):
1. Create a new GitHub repository (or use existing). Add this file as index.html in the repository root (or /docs) and commit.
2. In GitHub: Settings -> Pages -> Select branch (main) and folder (/root or /docs) -> Save. Your site will be published at https://<your-username>.github.io/<repo-name>/ (wait a minute for propagation).
3. Use the Python snippet at the bottom (or any QR generator) to create a QR code that points to that URL.

You can customize colors, star density, or add audio in the JS section.
-->

<!doctype html>
<html lang="th">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ท้องฟ้าดาวตก - Falling Stars</title>
  <style>
    html,body{height:100%;margin:0}
    body{
      background: radial-gradient(ellipse at bottom,#112349 0%, #03132b 60%, #000000 100%);
      overflow:hidden;
      font-family:system-ui,-apple-system,Segoe UI,Roboto, "Noto Sans", "Helvetica Neue", Arial;
      color:#fff;
      display:flex;align-items:center;justify-content:center;
    }
    canvas{position:fixed;left:0;top:0;width:100%;height:100%;display:block}
    .center-card{
      position:relative;z-index:5;padding:18px 22px;border-radius:12px;background:rgba(0,0,0,0.25);backdrop-filter:blur(6px);box-shadow:0 6px 30px rgba(2,6,23,0.6);
    }
    .title{font-size:18px;margin:0 0 6px 0}
    .subtitle{font-size:13px;opacity:0.85;margin:0}
    /* small responsive hint */
    @media (max-width:420px){.center-card{padding:12px 14px}}
  </style>
</head>
<body>
  <canvas id="sky"></canvas>

  <div class="center-card" role="note" aria-label="hint">
    
  </div>

  <script>
    // Falling stars canvas animation
    const canvas = document.getElementById('sky');
    const ctx = canvas.getContext('2d');

    let W = canvas.width = innerWidth;
    let H = canvas.height = innerHeight;

    // starfield
    const numStars = Math.floor((W*H)/10000 * 1.2) + 60; // density scales with screen
    let stars = [];

    function rand(min, max){ return Math.random() * (max-min) + min }

    function makeStars(){
      stars = [];
      for(let i=0;i<numStars;i++){
        stars.push({
          x: Math.random()*W,
          y: Math.random()*H,
          r: Math.random()*1.6 + 0.2,
          alpha: Math.random()*0.9 + 0.1,
          twinkleSpeed: Math.random()*0.02+0.002
        });
      }
    }

    // shooting stars pool
    let shooting = [];
    const maxShooting = 3;

    function spawnShooting(){
      if(shooting.length >= maxShooting) return;
      // spawn from top-left area heading down-right or random angle
      const startX = rand(0, W*0.6);
      const startY = rand(0, H*0.2);
      const speed = rand(6, 14);
      const length = rand(120, 320);
      const angle = rand(0.25, 0.6); // radians ~ diagonal down-right
      shooting.push({x:startX,y:startY,vx:Math.cos(angle)*speed, vy:Math.sin(angle)*speed, life:0, maxLife: rand(60,110), len: length});
    }

    // background gradient stars + subtle nebula
    function drawBackground(){
      const g = ctx.createLinearGradient(0,0,0,H);
      g.addColorStop(0,'rgba(9,18,28,0.85)');
      g.addColorStop(0.6,'rgba(2,6,23,0.95)');
      ctx.fillStyle = g;
      ctx.fillRect(0,0,W,H);

      // tiny soft nebula
      for(let i=0;i<5;i++){
        const cx = W * (i/5 + 0.1);
        const cy = H * (0.15 + i*0.12);
        const rad = Math.min(W,H) * (0.12 - i*0.015);
        const g2 = ctx.createRadialGradient(cx,cy,0,cx,cy,rad);
        g2.addColorStop(0, 'rgba(40,60,100,0.04)');
        g2.addColorStop(1, 'rgba(0,0,0,0)');
        ctx.fillStyle = g2;
        ctx.beginPath(); ctx.arc(cx,cy,rad,0,Math.PI*2); ctx.fill();
      }
    }

    function update(){
      ctx.clearRect(0,0,W,H);
      drawBackground();

      // draw stars
      for(let s of stars){
        s.alpha += Math.sin(Date.now()*s.twinkleSpeed)*0.0015; // subtle twinkle
        s.alpha = Math.max(0.05, Math.min(1, s.alpha));
        ctx.globalAlpha = s.alpha;
        ctx.beginPath();
        ctx.fillStyle = 'white';
        ctx.arc(s.x, s.y, s.r, 0, Math.PI*2);
        ctx.fill();
      }

      ctx.globalAlpha = 1;

      // update shooting stars
      for(let i = shooting.length-1; i>=0; i--){
        const star = shooting[i];
        star.x += star.vx; star.y += star.vy; star.life++;

        // trail
        const trailLen = Math.min(star.len, star.life*6);
        ctx.beginPath();
        ctx.moveTo(star.x, star.y);
        ctx.lineTo(star.x - star.vx*trailLen/10, star.y - star.vy*trailLen/10);
        ctx.strokeStyle = 'rgba(255,255,255,' + (1 - star.life/star.maxLife) + ')';
        ctx.lineWidth = 2;
        ctx.stroke();

        // head
        ctx.beginPath();
        ctx.fillStyle = 'rgba(255,255,255,' + (1 - star.life/star.maxLife) + ')';
        ctx.arc(star.x, star.y, 2.4, 0, Math.PI*2);
        ctx.fill();

        if(star.life > star.maxLife || star.x > W+50 || star.y > H+50){ shooting.splice(i,1); }
      }

      // occasionally spawn new shooting stars
      if(Math.random() < 0.015) spawnShooting();

      requestAnimationFrame(update);
    }

    // resize handler
    addEventListener('resize', ()=>{ W = canvas.width = innerWidth; H = canvas.height = innerHeight; makeStars(); });

    // initial
    makeStars(); update();

    // optional: let user tap to create a shooting star
    addEventListener('pointerdown', (e)=>{
      if(shooting.length < maxShooting){
        shooting.push({x: e.clientX, y: e.clientY, vx: rand(8,14), vy: rand(6,12), life:0, maxLife: rand(60,100), len: rand(120,220)});
      }
    });
  </script>

  <!--
  Python QR code example (save as generate_qr.py) - run locally after your GitHub Pages URL is ready.

  pip install qrcode[pil]

  from PIL import Image
  import qrcode

  url = 'https://<your-username>.github.io/<repo-name>/'
  qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_H, box_size=10, border=2)
  qr.add_data(url)
  qr.make(fit=True)
  img = qr.make_image(fill_color="black", back_color="white")
  img.save('falling_stars_qr.png')

  -->
</body>
</html>
