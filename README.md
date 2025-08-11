<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>لعبة صيد الزومبي</title>
<style>
  body { margin:0; overflow:hidden; background:#111; color:#eee; font-family: Tahoma, sans-serif; }
  #gameCanvas { background:#222; display: block; margin: auto; touch-action: none; }
  #info { text-align:center; margin: 10px; }
</style>
</head>
<body>
  <h1 style="text-align:center;">لعبة صيد الزومبي</h1>
  <canvas id="gameCanvas" width="400" height="600"></canvas>
  <div id="info">حاول تصيد كل الزومبي! استخدم الأسهم أو اللمس للتحرك.</div>

<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  const width = canvas.width;
  const height = canvas.height;

  const playerSize = 20;
  const zombieSize = 20;
  const speed = 3;
  const zombieSpeed = 1.5;

  let player = { x: width/2, y: height - 50 };
  let zombies = [];
  const maxZombies = 8;

  // انشاء الزومبي في أماكن عشوائية
  function spawnZombies() {
    zombies = [];
    for(let i=0; i<maxZombies; i++) {
      zombies.push({
        x: Math.random() * (width - zombieSize),
        y: Math.random() * (height/2),
        caught: false,
      });
    }
  }

  spawnZombies();

  // تحكم باللاعب عبر الأسهم أو اللمس
  let keys = {};
  window.addEventListener('keydown', e => { keys[e.key] = true; });
  window.addEventListener('keyup', e => { keys[e.key] = false; });

  // دعم اللمس لتحريك اللاعب
  let touchStartX = null;
  let touchStartY = null;

  canvas.addEventListener('touchstart', e => {
    if(e.touches.length === 1) {
      touchStartX = e.touches[0].clientX;
      touchStartY = e.touches[0].clientY;
    }
  });

  canvas.addEventListener('touchmove', e => {
    e.preventDefault();
    if(e.touches.length === 1 && touchStartX !== null) {
      let deltaX = e.touches[0].clientX - touchStartX;
      let deltaY = e.touches[0].clientY - touchStartY;
      if(Math.abs(deltaX) > 10) {
        player.x += deltaX > 0 ? speed : -speed;
        touchStartX = e.touches[0].clientX;
      }
      if(Math.abs(deltaY) > 10) {
        player.y += deltaY > 0 ? speed : -speed;
        touchStartY = e.touches[0].clientY;
      }
      clampPlayer();
    }
  });

  canvas.addEventListener('touchend', e => {
    touchStartX = null;
    touchStartY = null;
  });

  function clampPlayer() {
    if(player.x < 0) player.x = 0;
    if(player.x > width - playerSize) player.x = width - playerSize;
    if(player.y < 0) player.y = 0;
    if(player.y > height - playerSize) player.y = height - playerSize;
  }

  // تحديث حركة اللاعب حسب الضغطات
  function updatePlayer() {
    if(keys['ArrowLeft']) player.x -= speed;
    if(keys['ArrowRight']) player.x += speed;
    if(keys['ArrowUp']) player.y -= speed;
    if(keys['ArrowDown']) player.y += speed;
    clampPlayer();
  }

  // الزومبي يهربون من اللاعب بحركة بسيطة
  function updateZombies() {
    zombies.forEach(z => {
      if(z.caught) return;

      let dx = z.x - player.x;
      let dy = z.y - player.y;
      let dist = Math.sqrt(dx*dx + dy*dy);
      if(dist < 25) {
        z.caught = true;
        score++;
        document.getElementById('info').textContent = `صيدت الزومبي: ${score} من ${maxZombies}`;
      } else {
        // حركة هروب بسيطة مع حدود
        if(dist > 0) {
          z.x += (dx / dist) * zombieSpeed;
          z.y += (dy / dist) * zombieSpeed;
          z.x = Math.min(Math.max(z.x, 0), width - zombieSize);
          z.y = Math.min(Math.max(z.y, 0), height - zombieSize);
        }
      }
    });
  }

  // رسم اللاعب والزومبي
  function draw() {
    ctx.clearRect(0, 0, width, height);

    // اللاعب: مستطيل أخضر
    ctx.fillStyle = 'lime';
    ctx.fillRect(player.x, player.y, playerSize, playerSize);

    // الزومبي: مستطيل أحمر، مخفي لو صيد
    zombies.forEach(z => {
      if(!z.caught) {
        ctx.fillStyle = 'red';
        ctx.fillRect(z.x, z.y, zombieSize, zombieSize);
      }
    });
  }

  let score = 0;

  // اللعبة تنتهي لما تصيد كل الزومبي
  function gameLoop() {
    updatePlayer();
    updateZombies();
    draw();

    if(score >= maxZombies) {
      alert('مبروك! صيدت كل الزومبي.');
      spawnZombies();
      score = 0;
      document.getElementById('info').textContent = 'حاول تصيد كل الزومبي! استخدم الأسهم أو اللمس للتحرك.';
    } else {
      requestAnimationFrame(gameLoop);
    }
  }

  gameLoop();
})();
</script>

</body>
</html>
