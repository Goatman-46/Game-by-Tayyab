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


