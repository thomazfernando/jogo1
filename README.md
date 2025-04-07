!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meu Primeiro Jogo Multiplayer</title>
    <style>
        #screen {
            border: 1px solid #ccc;
            image-rendering: pixelated;
            display: block;
            margin-bottom: 10px;
        }

        #scoreboard {
            font-family: sans-serif;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <canvas id="screen"></canvas>
    <div id="scoreboard"></div>

    <script>
        const screen = document.getElementById('screen');
        const context = screen.getContext('2d');

        const gridSize = 10;
        const cellSize = 40;

        // Define o tamanho real do canvas (400x400)
        screen.width = gridSize * cellSize;
        screen.height = gridSize * cellSize;

        const currentPlayerId = 'player1';

        const state = {
            players: {},
            fruits: {}
        };

        function addPlayer(command) {
            const { playerId, playerx, playery } = command;
            state.players[playerId] = {
                x: playerx,
                y: playery,
                score: 0
            };
        }

        function removePlayer(command) {
            const { playerId } = command;
            delete state.players[playerId];
        }

        function addFruit(command) {
            const { fruitId, fruitx, fruity } = command;
            state.fruits[fruitId] = {
                x: fruitx,
                y: fruity
            };
        }

        function removeFruit(command) {
            const { fruitId } = command;
            delete state.fruits[fruitId];
        }

        function createGame() {
            function movePlayer(command) {
                const acceptedMoves = {
                    ArrowUp(player) {
                        if (player.y - 1 >= 0) player.y--;
                    },
                    ArrowRight(player) {
                        if (player.x + 1 < gridSize) player.x++;
                    },
                    ArrowDown(player) {
                        if (player.y + 1 < gridSize) player.y++;
                    },
                    ArrowLeft(player) {
                        if (player.x - 1 >= 0) player.x--;
                    }
                };

                const { keypressed, playerId } = command;
                const player = state.players[playerId];
                const moveFunction = acceptedMoves[keypressed];

                if (player && moveFunction) {
                    moveFunction(player);
                    checkForFruitCollision();
                }
            }

            function checkForFruitCollision() {
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
                state
            };
        }

        function createKeyboardListener() {
            const state = {
                observers: []
            };

            function subscribe(observerFunction) {
                state.observers.push(observerFunction);
            }

            function notifyAll(command) {
                for (const observerFunction of state.observers) {
                    observerFunction(command);
                }
            }

            function handleKeyDown(event) {
                const keypressed = event.key;
                const command = {
                    playerId: currentPlayerId,
                    keypressed
                };
                notifyAll(command);
            }

            document.addEventListener('keydown', handleKeyDown);

            return {
                subscribe
            };
        }

        const game = createGame();
        const keyboardListener = createKeyboardListener();
        keyboardListener.subscribe(game.movePlayer);

        // Adiciona jogadores iniciais
        game.addPlayer({ playerId: 'player1', playerx: 0, playery: 0 });
        game.addPlayer({ playerId: 'player2', playerx: 5, playery: 5 });
        game.addPlayer({ playerId: 'player3', playerx: 4, playery: 4 });
        game.addPlayer({ playerId: 'player4', playerx: 3, playery: 4 });

        // Gera frutas aleatórias
        function generateRandomFruit() {
            const x = Math.floor(Math.random() * gridSize);
            const y = Math.floor(Math.random() * gridSize);

            const positionIsOccupied = Object.values(game.state.players).some(p => p.x === x && p.y === y)
                || Object.values(game.state.fruits).some(f => f.x === x && f.y === y);

            if (!positionIsOccupied) {
                const fruitId = 'fruit-' + Math.floor(Math.random() * 100000);
                game.addFruit({ fruitId, fruitx: x, fruity: y });
            }
        }

        setInterval(generateRandomFruit, 2000); // a cada 2 segundos

        function renderScreen() {
            context.clearRect(0, 0, screen.width, screen.height);
            context.fillStyle = 'white';
            context.fillRect(0, 0, screen.width, screen.height);

            for (const playerId in game.state.players) {
                const player = game.state.players[playerId];
                context.fillStyle = 'black';
                context.fillRect(player.x * cellSize, player.y * cellSize, cellSize, cellSize);
            }

            for (const fruitId in game.state.fruits) {
                const fruit = game.state.fruits[fruitId];
                context.fillStyle = 'green';
                context.fillRect(fruit.x * cellSize, fruit.y * cellSize, cellSize, cellSize);
            }

            const scoreboard = document.getElementById('scoreboard');
            scoreboard.innerHTML = "<h3>Placar:</h3>" + Object.entries(game.state.players).map(([id, player]) => {
                return `${id}: ${player.score}`;
            }).join("<br>");

            requestAnimationFrame(renderScreen);
        }

        renderScreen();
    </script>
</body>
</html>
