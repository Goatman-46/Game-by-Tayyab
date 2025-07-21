<html>
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
if (gameState.coins >= 10) {
      gameState.hp = gameState.maxHp;
      gameState.caught.forEach(p => p.hp = p.hp || 10);
      gameState.coins -= 10;
      alert('Your team is fully healed!');
    } else {
      alert('Not enough coins!');
    }
    closeCenter();
  }

  function startBattle() {
    const random = wildPokemons[Math.floor(Math.random() * wildPokemons.length)];
    wildPokemon = {
      name: random.name,
      hp: random.maxHp,
      maxHp: random.maxHp,
      sprite: random.sprite
    };
    document.getElementById('enemySprite').src = wildPokemon.sprite;
    document.getElementById('battleScreen').style.display = 'block';
    document.getElementById('enemyName').textContent = 'Wild ' + wildPokemon.name;
    document.getElementById('playerName').textContent = gameState.starter;
    updateBars();
  }

  function useMove() {
    wildPokemon.hp -= 5;
    if (wildPokemon.hp <= 0) {
      wildPokemon.hp = 0;
      gameState.exp += 5;
      gameState.coins += 5;
      alert('You defeated the wild Pokémon!');
      document.getElementById('battleScreen').style.display = 'none';
      checkLevelUp();
      return;
    }

    gameState.hp -= 3;
    if (gameState.hp <= 0) {
      alert('You fainted!');
      gameState.hp = 1;
      document.getElementById('battleScreen').style.display = 'none';
    }
    updateBars();
  }

  function catchPokemon() {
    if (Math.random() < wildPokemon.hp / wildPokemon.maxHp) {
      alert(`Oh no! ${wildPokemon.name} broke free!`);
    } else {
      alert(`You caught ${wildPokemon.name}!`);
      gameState.caught.push({
        name: wildPokemon.name,
        hp: wildPokemon.maxHp,
        level: 1,
        sprite: wildPokemon.sprite
      });
      document.getElementById('battleScreen').style.display = 'none';
    }
  }

  function runFromBattle() {
    alert('You ran away safely!');
    document.getElementById('battleScreen').style.display = 'none';
  }

  function updateBars() {
    document.getElementById('enemyHp').style.width = (wildPokemon.hp / wildPokemon.maxHp * 100) + '%';
    document.getElementById('playerHp').style.width = (gameState.hp / gameState.maxHp * 100) + '%';
    document.getElementById('playerExp').style.width = (gameState.exp / gameState.expToLevel * 100) + '%';
  }

  function checkLevelUp() {
    if (gameState.exp >= gameState.expToLevel) {
      gameState.level++;
      gameState.exp = 0;
      gameState.expToLevel += 5;
      gameState.maxHp += 5;
      gameState.hp = gameState.maxHp;
      alert(`${gameState.starter} leveled up to level ${gameState.level}!`);
    }
  }

  function showStatus() {
    const list = document.getElementById('statusList');
    list.innerHTML = '';
    if (gameState.starter) {
      list.innerHTML += `<p>${gameState.starter} - HP: ${gameState.hp}/${gameState.maxHp}, Level: ${gameState.level}</p>`;
    }
    gameState.caught.forEach(p => {
      list.innerHTML += `<p>${p.name} - HP: ${p.hp}, Level: ${p.level}</p>`;
    });
    document.getElementById('statusScreen').style.display = 'block';
  }

  function closeStatus() {
    document.getElementById('statusScreen').style.display = 'none';
  }

  function saveGame() {
    localStorage.setItem("pokemonSave", JSON.stringify(gameState));
    alert("Game Saved!");
  }

  function showSwitchMenu() {
    const switchList = document.getElementById('switchList');
    switchList.innerHTML = '';
    if (gameState.caught.length === 0) {
      alert('You have no other Pokémon!');
      return;
    }
    gameState.caught.forEach((poke, index) => {
      const btn = document.createElement('button');
      btn.textContent = `${poke.name} (HP: ${poke.hp})`;
      btn.onclick = () => switchPokemon(index);
      switchList.appendChild(btn);
    });
    document.getElementById('switchMenu').style.display = 'block';
  }

  function closeSwitchMenu() {
    document.getElementById('switchMenu').style.display = 'none';
  }

  function switchPokemon(index) {
    gameState.caught.push({
      name: gameState.starter,
      hp: gameState.hp,
      level: gameState.level,
      sprite: null
    });

    const selected = gameState.caught.splice(index, 1)[0];
    gameState.starter = selected.name;
    gameState.hp = selected.hp;
    gameState.level = selected.level;

    alert(`You switched to ${selected.name}!`);
    updateBars();
    closeSwitchMenu();
  }

  function showStorage() {
    const list = document.getElementById("storageList");
    list.innerHTML = '';
    gameState.storage.forEach((poke, i) => {
      const div = document.createElement('div');
      div.innerHTML = `${poke.name} Lv.${poke.level} (HP: ${poke.hp}) `;

      const swap = document.createElement('button');
swap.textContent = 'Switch';
      swap.onclick = () => {
        if (gameState.caught.length < 6) {
          gameState.caught.push(poke);
          gameState.storage.splice(i, 1);
        } else {
          const removed = gameState.caught.pop();
          gameState.caught.push(poke);
          gameState.storage.splice(i, 1, removed);
        }
        showStorage();
      };
      div.appendChild(swap);

      const release = document.createElement('button');
      release.textContent = 'Release';
      release.onclick = () => {
        gameState.storage.splice(i, 1);
        showStorage();
      };
      div.appendChild(release);
      list.appendChild(div);
    });
    document.getElementById("storageScreen").style.display = 'block';
  }

  function closeStorage() {
    document.getElementById("storageScreen").style.display = 'none';
  }

  drawGame();
  setInterval(saveGame, 50000);
</script>
  <script>
    function saveToSlot(slot) {
      localStorage.setItem(`pokemonSaveSlot${slot}`, JSON.stringify(gameState));
      alert(`Game saved to Slot ${slot}`);
    }

    function loadFromSlot(slot) {
      const saved = localStorage.getItem(`pokemonSaveSlot${slot}`);
      if (saved) {
        gameState = JSON.parse(saved);
        localStorage.setItem("pokemonSave", saved);
        alert(`Game loaded from Slot ${slot}`);
        drawGame();
      } else {
        alert("No save data in this slot.");
      }
    }

    function resetGame() {
      if (confirm("Are you sure you want to reset your game? This cannot be undone.")) {
        localStorage.removeItem("pokemonSave");
        localStorage.removeItem("pokemonSaveSlot1");
        localStorage.removeItem("pokemonSaveSlot2");
        location.reload();
      }
    }
    function showReleaseMenu() {
  const list = document.getElementById('releaseList');
  list.innerHTML = '';

  if (gameState.caught.length === 0) {
    list.innerHTML = '<p>You have no caught Pokémon to release.</p>';
    return;
  }

  gameState.caught.forEach((poke, index) => {
    const div = document.createElement('div');
    div.innerHTML = `${poke.name} (HP: ${poke.hp}, Lv.${poke.level}) `;

    const releaseBtn = document.createElement('button');
    releaseBtn.textContent = 'Release';
    releaseBtn.onclick = () => {
      if (confirm(`Are you sure you want to release ${poke.name}?`)) {
        gameState.caught.splice(index, 1);
        alert(`${poke.name} was released.`);
        showReleaseMenu(); // refresh list
      }
    };

    div.appendChild(releaseBtn);
    list.appendChild(div);
  });

  document.getElementById('releaseMenu').style.display = 'block';
}

function closeReleaseMenu() {
  document.getElementById('releaseMenu').style.display = 'none';
}
  </script>
</body>
</html>
