<!DOCTYPE html>
<html>
<head>
    <title>Lgate6913.officiel - BOSS FIGHT</title>
    <style>
        body { margin: 0; background: #050505; color: white; font-family: 'Arial Black', sans-serif; display: flex; justify-content: center; overflow: hidden; }
        .side { width: 220px; padding: 20px; background: #000; border: 2px solid #222; z-index: 10; }
        canvas { background: #222; border: 4px solid #3498db; }
        
        #bossUI { 
            position: absolute; top: 80px; width: 400px; text-align: center; 
            display: none; /* Apparaît seulement au boss */
        }
        .boss-life { width: 80%; height: 15px; background: #555; border: 2px solid #fff; margin: 5px auto; }
        #bossBar { width: 100%; height: 100%; background: #e74c3c; transition: 0.2s; }
        
        .mission-info { position: absolute; top: 10px; width: 400px; text-align: center; background: rgba(0,0,0,0.8); padding: 10px; border-bottom: 2px solid #3498db; }
        .progress-bar { width: 100%; background: #333; height: 8px; margin-top: 5px; }
        #progFill { width: 0%; height: 100%; background: #3498db; }
    </style>
</head>
<body>

    <div class="side">
        <h3>Lgate6913</h3>
        <p>🚗 SANTÉ : <progress id="pHealth" value="100" max="100" style="width:100%"></progress></p>
        <hr>
        <p style="font-size:11px">AIDEZ LGATE !</p>
        <button style="background:#e91e63; color:white; padding:10px; width:100%" onclick="interact('Fan', 'Rose')">🌹 Rose (Réparer)</button>
        <button style="background:#f1c40f; color:black; padding:10px; width:100%; margin-top:5px" onclick="interact('Hero', 'Zap')">⚡ Éclair (Dégâts Boss)</button>
        <p style="font-size:10px; color:#777; margin-top:10px">Appuyez sur 'M' pour simuler l'arrivée à Marseille !</p>
    </div>

    <div style="position: relative;">
        <div class="mission-info">
            <span id="loc">EN ROUTE VERS MARSEILLE</span>
            <div class="progress-bar"><div id="progFill"></div></div>
            <small><span id="dist">0</span>m / 5000m</small>
        </div>

        <div id="bossUI">
            <b style="color:#e74c3c">🚁 HÉLICOPTÈRE GENDARMERIE</b>
            <div class="boss-life"><div id="bossBar"></div></div>
        </div>

        <canvas id="game"></canvas>
    </div>

<script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    canvas.width = 400; canvas.height = 600;

    let player = { x: 175, y: 500, w: 50, h: 80, hp: 100, speed: 4 };
    let boss = { active: false, x: 150, y: 80, hp: 100, w: 100, h: 40, dir: 1 };
    let projectiles = []; // Missiles du boss
    let distance = 0;
    let frame = 0;

    function interact(user, type) {
        if(type === 'Rose') player.hp = Math.min(100, player.hp + 20);
        if(type === 'Zap' && boss.active) {
            boss.hp -= 10;
            if(boss.hp <= 0) win();
        }
    }

    function win() {
        alert("FÉLICITATIONS ! Lgate6913 a semé l'hélicoptère et conquis Marseille !");
        location.reload();
    }

    function update() {
        frame++;
        if(!boss.active) {
            distance += 2;
            document.getElementById('progFill').style.width = (distance/5000)*100 + "%";
            document.getElementById('dist').innerText = distance;
            if(distance >= 5000) { boss.active = true; document.getElementById('bossUI').style.display = "block"; }
        } else {
            // Logique du Boss
            boss.x += 2 * boss.dir;
            if(boss.x > 300 || boss.x < 0) boss.dir *= -1;
            
            // Attaque : Le boss tire toutes les 60 frames
            if(frame % 60 === 0) {
                projectiles.push({x: boss.x + 50, y: boss.y + 40, w: 10, h: 20});
            }
            document.getElementById('bossBar').style.width = boss.hp + "%";
        }

        // Projectiles
        projectiles.forEach((p, i) => {
            p.y += 7;
            if(p.x < player.x + player.w && p.x + p.w > player.x && p.y < player.y + player.h && p.y + p.h > player.y) {
                player.hp -= 10;
                projectiles.splice(i, 1);
            }
            if(p.y > 600) projectiles.splice(i, 1);
        });

        document.getElementById('pHealth').value = player.hp;
        if(player.hp <= 0) { alert("GAME OVER ! La gendarmerie vous a eu."); location.reload(); }
    }

    function draw() {
        ctx.clearRect(0,0,400,600);
        // Route
        ctx.fillStyle = "#333"; ctx.fillRect(0,0,400,600);
        ctx.strokeStyle = "#555"; ctx.setLineDash([20,30]);
        ctx.beginPath(); ctx.moveTo(200,0); ctx.lineTo(200,600); ctx.stroke();

        // Joueur
        ctx.fillStyle = "#0055A4"; ctx.fillRect(player.x, player.y, player.w, player.h);
        ctx.fillStyle = "white"; ctx.font = "12px Arial";
        ctx.fillText("LGATE", player.x+8, player.y+45);

        // Boss (Hélicoptère)
        if(boss.active) {
            ctx.fillStyle = "#1e272e";
            ctx.fillRect(boss.x, boss.y, boss.w, boss.h); // Corps
            ctx.fillStyle = "#7f8c8d";
            ctx.fillRect(boss.x - 20, boss.y + 15, 140, 5); // Pales
            // Projecteur (Lumière)
            ctx.fillStyle = "rgba(255, 255, 0, 0.2)";
            ctx.beginPath();
            ctx.moveTo(boss.x+50, boss.y+40);
            ctx.lineTo(boss.x-50, 600);
            ctx.lineTo(boss.x+150, 600);
            ctx.fill();
        }

        // Missiles / Projectiles
        ctx.fillStyle = "#f1c40f";
        projectiles.forEach(p => ctx.fillRect(p.x, p.y, p.w, p.h));
    }

    function loop() { update(); draw(); requestAnimationFrame(loop); }

    window.addEventListener('keydown', (e) => {
        if(e.key === "ArrowLeft" && player.x > 0) player.x -= 20;
        if(e.key === "ArrowRight" && player.x < 350) player.x += 20;
        if(e.key === "m") distance = 4990; // Triche pour tester le boss !
    });

    loop();
</script>
</body>
</html>
