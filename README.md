<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Jogo de Aventura Complexo</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>
</head>
<body>
  <script>

    const CANVAS_SIZE = 600;
    const PLAYER_SIZE = 40;
    const ITEM_SIZE = 20;
    const ENEMY_SIZE = 30;
    const PLAYER_SPEED = 4;
    const ENEMY_SPEED = 2;
    const ITEM_MARGIN = 50;
    const INITIAL_LIVES = 3;

    let player;
    let items = [];
    let enemies = [];
    let score = 0;
    let lives = INITIAL_LIVES;
    let currentLevel = 1;

    function setup() {
      createCanvas(CANVAS_SIZE, CANVAS_SIZE);
      
      // Inicializa o jogador
      player = new Player(50, 50, PLAYER_SIZE);

      // Inicializa os itens e inimigos
      spawnItems(5); // 5 itens para coletar
      spawnEnemies(3); // 3 inimigos para evitar
    }

    function draw() {
      background(100, 150, 200);

      // Exibe e movimenta o jogador
      player.update();
      player.show();

      // Exibe e verifica colisões com itens
      for (let i = items.length - 1; i >= 0; i--) {
        items[i].show();
        if (player.collidesWith(items[i])) {
          score += 10;
          items.splice(i, 1); // Remove o item coletado
        }
      }

      // Exibe e movimenta os inimigos, verifica colisão com o jogador
      for (let enemy of enemies) {
        enemy.move();
        enemy.show();
        if (player.collidesWith(enemy)) {
          player.takeDamage();
        }
      }

      // Verifica se todos os itens foram coletados
      if (items.length === 0) {
        nextLevel();
      }

      // Exibe a pontuação e vidas
      displayHUD();

      // Verifica se o jogador perdeu
      if (lives <= 0) {
        gameOver();
      }
    }

    // Função para mostrar a pontuação e vidas na tela
    function displayHUD() {
      fill(255);
      textSize(18);
      text(`Pontuação: ${score}`, 10, 20);
      text(`Vidas: ${lives}`, 10, 40);
      text(`Fase: ${currentLevel}`, 10, 60);
    }

    // Próxima fase: mais itens e inimigos
    function nextLevel() {
      currentLevel++;
      spawnItems(5 + currentLevel); // Mais itens a cada nível
      spawnEnemies(3 + currentLevel); // Mais inimigos a cada nível
    }

    // Função para gerar itens no mapa
    function spawnItems(count) {
      items = [];
      for (let i = 0; i < count; i++) {
        items.push(new Item(random(ITEM_MARGIN, CANVAS_SIZE - ITEM_MARGIN), random(ITEM_MARGIN, CANVAS_SIZE - ITEM_MARGIN), ITEM_SIZE));
      }
    }

    // Função para gerar inimigos no mapa
    function spawnEnemies(count) {
      enemies = [];
      for (let i = 0; i < count; i++) {
        enemies.push(new Enemy(random(ITEM_MARGIN, CANVAS_SIZE - ITEM_MARGIN), random(ITEM_MARGIN, CANVAS_SIZE - ITEM_MARGIN), ENEMY_SIZE));
      }
    }

    // Exibe mensagem de fim de jogo
    function gameOver() {
      background(0);
      fill(255, 0, 0);
      textSize(32);
      textAlign(CENTER, CENTER);
      text("Fim de Jogo", CANVAS_SIZE / 2, CANVAS_SIZE / 2);
      noLoop(); // Para o jogo
    }

    // Classe do jogador
    class Player {
      constructor(x, y, size) {
        this.x = x;
        this.y = y;
        this.size = size;
        this.isInvincible = false;
        this.invincibilityTimer = 0;
      }

      update() {
        if (keyIsDown(LEFT_ARROW)) {
          this.x -= PLAYER_SPEED;
        }

        if (keyIsDown(RIGHT_ARROW)) {
          this.x += PLAYER_SPEED;
        }

        if (keyIsDown(UP_ARROW)) {
          this.y -= PLAYER_SPEED;
        }

        if (keyIsDown(DOWN_ARROW)) {
          this.y += PLAYER_SPEED;
        }

        // Limita os movimentos às bordas da tela
        this.x = constrain(this.x, 0, CANVAS_SIZE - this.size);
        this.y = constrain(this.y, 0, CANVAS_SIZE - this.size);

        // Reduz o tempo de invencibilidade se estiver ativo
        if (this.isInvincible) {
          this.invincibilityTimer--;
          if (this.invincibilityTimer <= 0) {
            this.isInvincible = false;
          }
        }
      }

      show() {
        fill(this.isInvincible ? color(255, 255, 0) : color(0, 100, 255));
        rect(this.x, this.y, this.size, this.size);
      }

      takeDamage() {
        if (!this.isInvincible) {
          lives--;
          this.isInvincible = true;
          this.invincibilityTimer = 60; // 1 segundo de invencibilidade
        }
      }

      collidesWith(other) {
        let d = dist(this.x + this.size / 2, this.y + this.size / 2, other.x, other.y);
        return d < this.size / 2 + other.size / 2;
      }
    }

    // Classe do item de coleta
    class Item {
      constructor(x, y, size) {
        this.x = x;
        this.y = y;
        this.size = size;
      }

      show() {
        fill(0, 255, 0);
        ellipse(this.x, this.y, this.size);
      }
    }

    // Classe dos inimigos
    class Enemy {
      constructor(x, y, size) {
        this.x = x;
        this.y = y;
        this.size = size;
        this.direction = createVector(random([-1, 1]), random([-1, 1]));
      }

      move() {
        this.x += this.direction.x * ENEMY_SPEED;
        this.y += this.direction.y * ENEMY_SPEED;

        // Inverte a direção ao colidir com as bordas
        if (this.x <= 0 || this.x >= CANVAS_SIZE - this.size) {
          this.direction.x *= -1;
        }
        if (this.y <= 0 || this.y >= CANVAS_SIZE - this.size) {
          this.direction.y *= -1;
        }
      }

      show() {
        fill(255, 0, 0);
        rect(this.x, this.y, this.size, this.size);
      }
    }

  </script>
</body>
</html>

