<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Retro Pokémon Game</title>
  <style>
    body {
      background: #111;
      color: white;
      font-family: sans-serif;
      margin: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    canvas {
      border: 2px solid white;
      background: black;
      margin-top: 10px;
    }
    #battleScreen, #starterSelection, #pokemonCenter, #statusScreen, #switchMenu, #storageScreen {
      display: none;
      text-align: center;
      margin-top: 10px;
    }
    .controls, .battle-controls {
      margin-top: 10px;
      text-align: center;
    }
    button {
      padding: 10px;
      background: black;
      color: lime;
      border: 2px solid lime;
      margin: 5px;
      font-size: 16px;
    }
    .hp-bar, .exp-bar {
      width: 100px;
      height: 10px;
      background: darkred;
      border: 1px solid white;
      margin: 4px auto;
      position: relative;
    }
    .hp-fill {
      background: lime;
      height: 100%;
    }
    .exp-fill {
background: cyan;
      height: 100%;
    }
  </style>
</head>
<body>
  <h1>Retro Pokémon</h1>
  <canvas id="gameCanvas" width="256" height="256"></canvas>
<div id="saveSlots">
    <h3>Save Slots</h3>
    <div>
      <button onclick="saveToSlot(1)">Save Slot 1</button>
      <button onclick="loadFromSlot(1)">Load Slot 1</button>
    </div>
    <div>
      <button onclick="saveToSlot(2)">Save Slot 2</button>
      <button onclick="loadFromSlot(2)">Load Slot 2</button>
    </div>
    <div>
      <button onclick="resetGame()">Reset Game</button>
    </div>
  </div>
  <div id="starterSelection">
    <h2>Choose Your Starter</h2>
    <button onclick="chooseStarter('Charmander')">Charmander</button>
    <button onclick="chooseStarter('Squirtle')">Squirtle</button>
    <button onclick="chooseStarter('Bulbasaur')">Bulbasaur</button>
  </div>
  <div id="battleScreen">
    <h2>Wild Encounter!</h2>
    <div id="enemyName"></div>
    <div class="hp-bar"><div id="enemyHp" class="hp-fill"></div></div>
    <img id="enemySprite" width="96" height="96" />
    <div id="playerName"></div>
    <div class="hp-bar"><div id="playerHp" class="hp-fill"></div></div>
    <div class="exp-bar"><div id="playerExp" class="exp-fill"></div></div>
    <div class="battle-controls">
      <button onclick="useMove()">Attack</button>
      <button onclick="catchPokemon()">Throw Pokéball</button>
      <button onclick="runFromBattle()">Run</button>
      <button onclick="showSwitchMenu()">Switch Pokémon</button>
    </div>
  </div>

  <div id="pokemonCenter">
    <h2>Pokémon Center</h2>
    <p>Heal your team?</p>
    <button onclick="showReleaseMenu()">Release Pokémon</button>
    <div id="releaseMenu" style="margin-top:10px; display:none;">
      <h3>Choose a Pokémon to release:</h3>
    <div id="releaseList"></div>
      <button onclick="closeReleaseMenu()">Cancel</button>
    </div>
    <button onclick="healTeam()">Heal</button>
    <button onclick="closeCenter()">Close</button>
  </div>

  <div id="statusScreen">
    <h2>Your Pokémon</h2>
    <div id="statusList"></div>
    <button onclick="closeStatus()">Close</button>
    <button onclick="saveGame()">Save Game</button>
  </div>

  <div id="switchMenu">
    <h3>Choose a Pokémon to switch:</h3>
    <div id="switchList"></div>
    <button onclick="closeSwitchMenu()">Cancel</button>
  </div>

  <div id="storageScreen">
    <h2>Pokémon Storage</h2>
    <div id="storageList"></div>
    <button onclick="closeStorage()">Close Storage</button>
  </div>

  <div class="controls">
    <div>
      <button onclick="move('up')">↑</button>
    </div>
    <div>
      <button onclick="move('left')">←</button>
      <button onclick="move('down')">↓</button>
      <button onclick="move('right')">→</button>
    </div>
    <button onclick="showStatus()">Pokémon Status</button>
  </div>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const tileSize = 32;
  const mapRows = 8;
  const mapCols = 8;

  const map = [
    [0,0,0,3,1,1,1,1,1,1,1],
    [0,2,0,0,1,1,1,1,1,1,1],
    [0,0,0,0,0,3,1,1,1,1,1],
[0,0,0,3,0,0,1,1,1,1,1],
    [0,0,0,0,0,0,0,0,0,3,0],
    [1,1,0,0,0,0,0,0,3,0,0],
    [1,1,3,0,0,0,0,0,0,0,0],
    [1,1,1,0,0,0,3,0,0,0,0],
  ];

  const tileColors = {
    0: 'green',
    1: 'blue',
    2: 'pink',
    3: 'green'
  };

  let player = { x: 0, y: 0, color: 'red' };

  let gameState = JSON.parse(localStorage.getItem("pokemonSave")) || {
    starter: null,
    coins: 50,
    level: 1,
    exp: 0,
    expToLevel: 10,
    hp: 20,
    maxHp: 20,
    caught: [],
    storage: []
  };

  const wildPokemons = [
    { name: 'Pidgey', maxHp: 10, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/pidgey.png' },
    { name: 'Rattata', maxHp: 12, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/rattata.png' },
    { name: 'Caterpie', maxHp: 8, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/caterpie.png' },
    { name: 'Zubat', maxHp: 10, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/zubat.png' },
    { name: 'Charizard', maxHp: 25, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/charizard.png' },
    { name: 'Pikachu', maxHp: 12, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/pikachu.png' },
    { name: 'Squirtle', maxHp: 12, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/squirtle.png' },
    { name: 'Bulbasaur', maxHp: 12, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/bulbasaur.png' },
    { name: 'Charmander', maxHp: 12, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/charmander.png' },
    { name: 'Blastoise', maxHp: 25, sprite: 'https://img.pokemondb.net/sprites/red-blue/normal/Blastoise.png' }
  ];

  let wildPokemon = {};

  function drawMap() {
    for (let row = 0; row < mapRows; row++) {
      for (let col = 0; col < mapCols; col++) {
        const tile = map[row][col];
        ctx.fillStyle = tileColors[tile];
        ctx.fillRect(col * tileSize, row * tileSize, tileSize, tileSize);
     }
    }
  }

  function drawPlayer() {
    ctx.fillStyle = player.color;
    ctx.fillRect(player.x * tileSize, player.y * tileSize, tileSize, tileSize);
  }

  function drawGame() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawMap();
    drawPlayer();
  }

  function move(direction) {
    if (direction === 'up' && player.y > 0) player.y--;
    if (direction === 'down' && player.y < mapRows - 1) player.y++;
    if (direction === 'left' && player.x > 0) player.x--;
    if (direction === 'right' && player.x < mapCols - 1) player.x++;
    drawGame();
    checkTile();
  }

  function checkTile() {
    const tile = map[player.y][player.x];
    if (!gameState.starter) {
      document.getElementById('starterSelection').style.display = 'block';
    } else if (tile === 3 && Math.random() < 0.6) {
      startBattle();
    } else if (tile === 2) {
      openCenter();
    }
  }

  function chooseStarter(name) {
    gameState.starter = name;
    document.getElementById('starterSelection').style.display = 'none';
    alert(`${name} has joined your team!`);
    saveGame();
  }

  function openCenter() {
    document.getElementById('pokemonCenter').style.display = 'block';
  }

  function closeCenter() {
    document.getElementById('pokemonCenter').style.display = 'none';
  }

  function healTeam() {
