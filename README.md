<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Jogo Multiplayer</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      font-family: sans-serif;
      margin: 0;
      padding: 0;
    }

    #screen {
      border: 1px solid #ccc;
      image-rendering: pixelated;
      background: #f9f9f9;
    }

    #scoreboard {
      margin: 10px;
    }

    #controls {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      margin: 20px 0;
    }

    .btn {
      width: 60px;
      height: 60px;
      margin: 5px;
      font-size: 24px;
      text-align: center;
      line-height: 60px;
      border-radius: 10px;
      background-color: #eee;
      border: 1px solid #aaa;
      user-select: none;
    }

    .center {
      width: 100%;
      text-align: center;
    }

    @media (max-width: 500px) {
      .btn {
        width: 50px;
        height: 50px;
        font-size: 20px;
        line-height: 50px;
      }
    }
  </style>
</head>
<body>
  <h2>Jogo Multiplayer</h2>
  <canvas id="screen"></canvas>
  <div id="scoreboard"></div>

  <div id="controls">
    <div class="center">
      <div class="btn" data-key="ArrowUp">⬆️</div>
    </div>
    <div>
      <div class="btn" data-key="ArrowLeft">⬅️</div>
      <div class="btn" data-key="ArrowDown">⬇️</div>
      <div class="btn" data-key="ArrowRight">➡️</div>
    </div>
  </div>

  <script>
    const screen = document.getElementById("screen");
    const context = screen.getContext("2d");

    const gridSize = 10;
    const cellSize = Math.min(window.innerWidth, 400) / gridSize;

    screen.width = gridSize * cellSize;
    screen.height = gridSize * cellSize;

    const currentPlayerId = "player1";

    const state = {
      players: {},
      fruits: {},
    };

    function addPlayer({ playerId, playerx, playery }) {
      state.players[playerId] = {
        x: playerx,
        y: playery,
        score: 0,
      };
    }

    function removePlayer({ playerId }) {
      delete state.players[playerId];
    }

    function addFruit({ fruitId, fruitx, fruity }) {
      state.fruits[fruitId] = {
        x: fruitx,
        y: fruity,
      };
    }

    function removeFruit({ fruitId }) {
      delete state.fruits[fruitId];
    }

    function createGame() {
      function movePlayer({ keypressed, playerId }) {
        const player = state.players[playerId];
        if (!player) return;

        const moves = {
          ArrowUp: () => player.y > 0 && player.y--,
          ArrowRight: () => player.x < gridSize - 1 && player.x++,
          ArrowDown: () => player.y < gridSize - 1 && player.y++,
          ArrowLeft: () => player.x > 0 && player.x--,
        };

        if (moves[keypressed]) {
          moves[keypressed]();
          checkFruitCollision();
        }
      }

      function checkFruitCollision() {
        for (const playerId in state.players) {
          const player = state.players[playerId];
          for (const fruitId in state.fruits) {
            const fruit = state.fruits[fruitId];
            if (player.x === fruit.x && player.y === fruit.y) {
              player.score++;
              removeFruit({ fruitId });
            }
          }
        }
      }

      return {
        addPlayer,
        removePlayer,
        addFruit,
        removeFruit,
        movePlayer,
        state,
      };
    }

    function createKeyboardListener() {
      const observers = [];

      function subscribe(fn) {
        observers.push(fn);
      }

      function notifyAll(command) {
        observers.forEach((fn) => fn(command));
      }

      function handleKey(event) {
        notifyAll({ playerId: currentPlayerId, keypressed: event.key });
      }

      document.addEventListener("keydown", handleKey);

      // suporte ao toque
      document.querySelectorAll(".btn").forEach((btn) => {
        btn.addEventListener("touchstart", (e) => {
          const key = btn.dataset.key;
          notifyAll({ playerId: currentPlayerId, keypressed: key });
          e.preventDefault();
        });
      });

      return {
        subscribe,
      };
    }

    const game = createGame();
    const keyboard = createKeyboardListener();
    keyboard.subscribe(game.movePlayer);

    // jogadores iniciais
    game.addPlayer({ playerId: "player1", playerx: 0, playery: 0 });
    game.addPlayer({ playerId: "player2", playerx: 5, playery: 5 });

    // frutas automáticas
    function generateRandomFruit() {
      const x = Math.floor(Math.random() * gridSize);
      const y = Math.floor(Math.random() * gridSize);
      const occupied = Object.values(game.state.players).some(p => p.x === x && p.y === y) ||
                       Object.values(game.state.fruits).some(f => f.x === x && f.y === y);
      if (!occupied) {
        const fruitId = 'fruit-' + Math.floor(Math.random() * 100000);
        game.addFruit({ fruitId, fruitx: x, fruity: y });
      }
    }
    setInterval(generateRandomFruit, 2000);

    function renderScreen() {
      context.clearRect(0, 0, screen.width, screen.height);

      for (const playerId in game.state.players) {
        const player = game.state.players[playerId];
        context.fillStyle = "black";
        context.fillRect(player.x * cellSize, player.y * cellSize, cellSize, cellSize);
      }

      for (const fruitId in game.state.fruits) {
        const fruit = game.state.fruits[fruitId];
        context.fillStyle = "green";
        context.fillRect(fruit.x * cellSize, fruit.y * cellSize, cellSize, cellSize);
      }

      const scoreboard = document.getElementById("scoreboard");
      scoreboard.innerHTML = "<b>Placar:</b><br>" +
        Object.entries(game.state.players).map(([id, p]) => `${id}: ${p.score}`).join("<br>");

      requestAnimationFrame(renderScreen);
    }

    renderScreen();
  </script>
</body>
</html>

