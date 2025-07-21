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
