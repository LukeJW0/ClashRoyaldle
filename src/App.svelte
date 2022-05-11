<script>
  import { tick } from "svelte";
  let storage = window.localStorage;
  // storage.clear();

  import { cards } from './lists.js';

  const r = Math.floor(Math.random() * cards.length);
  let randomCard = cards[r];

  function clearStuff() {
    localStorage.clear();
    console.log('test');
  }

  let guessFocused = false;

  function guessFocus() {
    guessFocused = !guessFocused;
  }

  let inputVar = '';

  let timesPlayed = storage.getItem("timesPlayed") || "0";
  timesPlayed = parseInt(timesPlayed);
  let timesCorrect = storage.getItem("timesCorrect") || "0";
  timesCorrect = parseInt(timesCorrect);
  let streak = storage.getItem("streak") || "0";
  streak = parseInt(streak);
  let lastCorrect = storage.getItem("lastCorrect") || "false";

  let guesses = [true, true, true, true, true, true];
  let guessIndex = 0;

  let guess = [['', '-', '', '', '', ''], ['', '-', '', '', '', ''], ['', '-', '', '', '', ''], ['', '-', '', '', '', ''], ['', '-', '', '', '', ''], ['', '-', '', '', '', '']];
  let guessColor = [['', '', '', '', '', ''], ['', '', '', '', '', ''], ['', '', '', '', '', ''], ['', '', '', '', '', ''], ['', '', '', '', '', ''], ['', '', '', '', '', '']];

  let mobileElement;
  
  function enterGuess() {
    if (won) return;
    let cardIndex = -1;
    for (let i = 0; i < cards.length; i++) {
      if (cards[i].Name.toUpperCase() === inputVar) {
        cardIndex = i;
        break;
      }
    }
    // if (cardIndex = -1) {
    //   return;
    // }
    guess[guessIndex][0] = cards[cardIndex].Name;
    guess[guessIndex][1] = cards[cardIndex].Rarity;
    guess[guessIndex][2] = cards[cardIndex].Health;
    guess[guessIndex][3] = cards[cardIndex].Damage;
    guess[guessIndex][4] = cards[cardIndex].Type;
    guess[guessIndex][5] = cards[cardIndex].Elixir;
    if (getComputedStyle(mobileElement).display !== "none") {
      if (cards[cardIndex].Rarity === "Common") {
        guess[guessIndex][1] = "Comm";
      }
      if (cards[cardIndex].Rarity === "Champion") {
        guess[guessIndex][1] = "Champ";
      }
      if (cards[cardIndex].Rarity === "Legendary") {
        guess[guessIndex][1] = "Legnd";
      }
      if (cards[cardIndex].Type === "Win Condition") {
        guess[guessIndex][4] = "WinCon";
      }
    }
    if (cards[cardIndex].Name === randomCard.Name) {
      guessColor[guessIndex][0] = "green";
      gameWon();
    } else {
      guessColor[guessIndex][0] = "gray";
    }
    if (cards[cardIndex].Rarity === randomCard.Rarity) {
      guessColor[guessIndex][1] = "green";
    } else {
      guessColor[guessIndex][1] = "gray";
    }
    if (parseInt(cards[cardIndex].Health) < parseInt(randomCard.Health)) {
      guessColor[guessIndex][2] = "red";
      guess[guessIndex][2] = cards[cardIndex].Health + "+";
    } else if (parseInt(cards[cardIndex].Health) > parseInt(randomCard.Health)) {
      guessColor[guessIndex][2] = "yellow";
      guess[guessIndex][2] = cards[cardIndex].Health + "-";
    } else {
      guessColor[guessIndex][2] = "green";
    }
    if (parseInt(cards[cardIndex].Damage) < parseInt(randomCard.Damage)) {
      guessColor[guessIndex][3] = "red";
      guess[guessIndex][3] = cards[cardIndex].Damage + "+";
    } else if (parseInt(cards[cardIndex].Damage) > parseInt(randomCard.Damage)) {
      guessColor[guessIndex][3] = "yellow";
      guess[guessIndex][3] = cards[cardIndex].Damage + "-";
    } else {
      guessColor[guessIndex][3] = "green";
    }
    if (cards[cardIndex].Type === randomCard.Type) {
      guessColor[guessIndex][4] = "green";
    } else {
      guessColor[guessIndex][4] = "gray";
    }
    if (cards[cardIndex].Elixir === randomCard.Elixir) {
      guessColor[guessIndex][5] = "green";
    } else {
      guessColor[guessIndex][5] = "gray";
    }
    guessIndex++;
    if (guessIndex > 5) {
      gameLost();
    }
  }

  let gameResult;
  let statsWindowShow = false;
  let infoWindowShow = false;

  function gameLost() {
    lastCorrect = "false";
    storage.setItem("lastCorrect", lastCorrect);
    timesPlayed++;
    storage.setItem("timesPlayed", JSON.stringify(timesPlayed));
    streak = 0;
    storage.setItem("streak", JSON.stringify(streak));
    gameResult = "YOU LOST";
    statsWindowShow = true;
    infoWindowShow = false;
  }

  let won = false;

  function gameWon() {
    won = true;
    lastCorrect = "true";
    if (lastCorrect === "true") {
      streak++;
      storage.setItem("streak", JSON.stringify(streak));
    }
    storage.setItem("lastCorrect", lastCorrect);
    timesPlayed++;
    storage.setItem("timesPlayed", JSON.stringify(timesPlayed));
    timesCorrect++;
    storage.setItem("timesCorrect", JSON.stringify(timesCorrect));
    gameResult = "YOU WON";
    statsWindowShow = true;
    infoWindowShow = false;
  }

  function showStats() {
    gameResult = "";
    statsWindowShow = true;
    infoWindowShow = false;
  }

  let firstAccess = storage.getItem("firstAccess") || "yes";
  if (firstAccess === "yes") {
    showInfo();
    storage.setItem("firstAccess", "no");
  }

  function closeStats() {
    statsWindowShow = false;
  }

  function showInfo() {
    statsWindowShow = false;
    infoWindowShow = true;
  }

  function closeInfo() {
    infoWindowShow = false;
  }

  let selected = '';

  let searchResults = [];
  async function filterSearch() {
    await tick();
    let inputValue = inputVar;
    searchResults = [];
    if (inputValue.length == 0) {
      return;
    }
    let i = 0;
    while (searchResults.length < 5 && i < cards.length) {
      if (cards[i].Name.toUpperCase().indexOf(inputValue.toUpperCase()) > -1) {
        searchResults.push(cards[i].Name);
        searchResults = searchResults;
      }
      i++;
    }
  }

  function chooseResult(i) {
    inputVar = searchResults[i].toUpperCase();
  }

  function enterGuessKey() {
    if (event.key !== 'Enter') return;

    // const current = document.activeElement;
    // guessFocus();
    // enterGuess();
    // current.blur();
    // guessFocus();
    enterGuess();
  }
  
</script>

<main>
  <div id=container>
    {#if statsWindowShow}
      <div class=window>
        <h1 class=gameresult>{gameResult}</h1>
        {#if gameResult !== ""}
          <h1 class=correctanswer>CARD: {randomCard.Name}</h1>
        {/if}
        <h1 class=statstitle>STATS:</h1>
        <h1 class=statstext>Times Played: {timesPlayed}</h1>
        <h1 class=statstext>Total Correct: {timesCorrect}</h1>
        <h1 class=statstext>Streak: {streak}</h1>
        <button class=closestats on:click={closeStats}>close</button>
      </div>
    {/if}
    {#if infoWindowShow}
      <div class=window>
        <h1 class=statstitle>INFO</h1>
        <h1 class=infotext>In Clash Royaldle there are 4 types of cards. Troop, Spell, Building, and Win Condition. In this game, win conditions are cards that specifically only attack buildings and towers. On health and damage, a red + means you need to guess a card with a higher value. A yellow - means you need to guess lower. On mobile, some things are abbreviated. Common is comm, Champion is champ, legendary is legnd, and win condition is wincon.</h1>
        <button class=closestats on:click={closeInfo}>close</button>
      </div>
    {/if}
    <div class=header>
      <div class=tiktokbuttondiv>
        <i class="fa-brands fa-tiktok fa-2xl tiktokbutton"></i>
      </div>
      <h1 class=title>CLASH ROYALDLE</h1>
      <div class=buttons>
        <i class="fa-solid fa-chart-simple fa-2xl statsbutton" on:mousedown={showStats}></i>
        <i class="fa-solid fa-circle-info fa-2xl infobutton" on:mousedown={showInfo}></i>
      </div>
    </div>
    <div class="buttons">
      <i class="fa-brands fa-tiktok fa-2xl tiktokbutton"></i>
      <i class="fa-solid fa-chart-simple fa-2xl statsbutton" on:mousedown={showStats}></i>
      <i class="fa-solid fa-circle-info fa-2xl infobutton" on:mousedown={showInfo}></i>
    </div>
    <h1 class=title>CLASH ROYALDLE</h1>
    <h1 class=mobile-title>CLASH ROYALDLE</h1>
    <div class=guessgroup>
      <input class="inputbox inputmovie" type="text" on:keydown={enterGuessKey} bind:value={inputVar} on:input={filterSearch} on:focus={guessFocus} on:blur={guessFocus}>
      {#if guessFocused}
        {#each searchResults as result, i}
          <button class=searchresult on:mousedown={() => chooseResult(i)}>{searchResults[i]}</button>
        {/each}
      {/if}
    </div>
    <button class=enterbutton on:click={enterGuess}>Enter</button>
    <div class=results>
      <div class=result>
        <div style="color: white" class="resultchild mobiledisable">Name</div>
        <div style="color: white" class=resultchild>Rarity</div>
        <div style="color: white" class=resultchild>Health</div>
        <div style="color: white" class=resultchild><span class=mobiledisable>Damage</span><span class=mobileenable>DMG</span></div>
        <div style="color: white" class=resultchild>Type</div>
        <div style="color: white" class=resultchild>Elixir</div>
      </div>
      {#each guesses as guessed, i}
        <h1 bind:this={mobileElement} style="color: {guessColor[i][0]}" class="name mobileenable">{guess[i][0]}</h1>
        <div class=result disabled={guessed}>
          <div style="color: {guessColor[i][0]}" class="resultchild mobiledisable">{guess[i][0]}</div>
          <div style="color: {guessColor[i][1]}" class=resultchild>{guess[i][1]}</div>
          <div style="color: {guessColor[i][2]}" class=resultchild>{guess[i][2]}</div>
          <div style="color: {guessColor[i][3]}" class=resultchild>{guess[i][3]}</div>
          <div style="color: {guessColor[i][4]}" class=resultchild>{guess[i][4]}</div>
          <div style="color: {guessColor[i][5]}" class=resultchild>{guess[i][5]}</div>
        </div>
      {/each}
    </div>
<!--       <button on:click={clearStuff}>Clear</button> -->
  </div>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }

  main {
    text-align: center;
    margin: 0 auto;
    background-color: #121212;
    min-height: 100vh;
    padding-bottom: 5rem;
  }

  .header {
    background-color: gray;
    display: none;
    align-items: center;
    margin-bottom: 1rem;
  }

  .statsbutton {
/*     color: #222222; */
    color: white;
    margin: 0 1%;
  }

  .infobutton {
/*     color: #222222; */
    color: white;
/*     margin: 0 10%; */
    margin: 0 1%;
  }

  .tiktokbutton {
/*     color: #222222; */
    color: white;
/*     margin: 0 15%; */
    margin: 0 1%;
  }

  .tiktokbuttondiv {
    flex-basis: 20%;
    display: flex;
    justify-content: left;
  }

  .buttons {
/*     flex-basis: 20%;
    display: flex;
    justify-content: right; */
    justify-content: center;
    margin-top: 2%;
    margin-bottom: 1.5%;
  }

  .statsbutton:hover {
    color: gray;
  }

  .infobutton:hover {
    color: gray;
  }

  .tiktokbutton:hover {
    color: gray;
  }

  .infotext {
    font-size: 2rem;
    margin: 2% auto;
    max-width: 90%;
    text-align: center;
  }

  .title {
    font-weight: 400;
    font-size: 4rem;
    text-align: center;
    flex-basis: 60%;
  }

  .name {
    font-size: 1.5rem;
    text-align: left;
    margin-left: 2.5%;
    margin-right: 2.5%;
    padding: 1% 0 0 1%;
    line-height: 1.25;
    color: #121212;
    border-left: 0.15rem solid #7d7d7d;
    border-right: 0.15rem solid #7d7d7d;
  }

  .select {
    margin: auto;
    max-width: fit-content;
    font-size: 1.5rem;
    font-family: 'Montserrat', sans-serif;
    font-weight: 200;
    text-align: center;
    text-transform: uppercase;
    background-color: #222222;
    border: 0.15rem solid #7d7d7d;
    color: white;
    border-radius: 0.5rem;
    flex-basis: 20%;
    padding: 0 1%;
/*     margin-right: auto; */
  }

  .select:hover {
    background-color: white;
    color: black;
  }

  .image {
    min-width: 35%;
    max-width: 35%;
    min-height: 17rem;
    max-height: 17rem;
    margin: 2rem 0;
    border: 0.25rem solid gray;
    object-fit: cover;
  }

  .hint {
    font-size: 3rem;
    max-width: 90%;
    margin: 0 auto;
  }

  .inputbox {
    margin: 0.5% auto;
    width: 40%;
    font-size: 2rem;
    font-family: 'Montserrat', sans-serif;
    font-weight: 200;
    text-align: center;
    text-transform: uppercase;
    background-color: #313438;
    border: 0.15rem solid #7d7d7d;
    color: white;
    padding: 0.5%;
  }

  .inputbox:disabled {
    background-color: #1a1c1f;
    color: gray;
  }

  .searchresult {
    width: 40%;
    margin: -0.5% auto 0 auto;
    font-size: 2rem;
    font-weight: 200;
    border: 0.1rem solid #7d7d7d;
    border-radius: 0;
    height: fit-content;
  }

  .window {
    position: absolute;
    width: 50%;
    margin-left: auto;
    margin-right: auto;
    top: 10rem;
    left: 0;
    right: 0;
    text-align: center;
    z-index: 1;
    background-color: #333333;
    border-radius: 0.5rem;
    border: 0.15rem solid white;
    padding: 1% 0;
    box-shadow: rgba(0, 0, 0, 0.25) 0px 54px 55px, rgba(0, 0, 0, 0.12) 0px -12px 30px, rgba(0, 0, 0, 0.12) 0px 4px 6px, rgba(0, 0, 0, 0.17) 0px 12px 13px, rgba(0, 0, 0, 0.09) 0px -3px 5px;
  }

  .statstext {
    font-size: 3rem;
    margin: 2% 0;
  }

  .statstitle {
    font-size: 3.5rem;
  }

  .gameresult {
    font-weight: 400;
    font-size: 4rem;
    margin-bottom: 2%;
    margin-top: 1%;
  }

  .correctanswer {
    font-size: 3.25rem;
    font-weight: 400;
    margin-bottom: 3%;
    margin-top: -0.5%;
  }

  .closestats {
    margin: 2% 0;
  }

  .guessgroup {
    display: flex;
    flex-direction: column;
    margin: 1rem 0;
    margin-top: 2rem;
  }

  .results {
    display: flex-column;
    justify-content: center;
    margin: 0 auto;
    margin-top: 5rem;
  }

  .result {
    display: flex;
    margin: 0 auto;
    width: 80%;
  }

  .resultchild {
    background-color: #222222;
    width: 16%;
    max-width: 16%;
    border: 0.15rem solid #7d7d7d;
    color: #222222;
    font-family: 'Montserrat', sans-serif;
    font-size: 1.5rem;
    padding: 1% 0;
    text-overflow: hidden;
  }
  
  h1 {
    color: white;
    text-transform: uppercase;
    font-family: 'Montserrat', sans-serif;
    font-weight: 300;
    line-height: 1.1;
    margin: 0;
    padding: 0;
    font-family: 'Montserrat', sans-serif;
  }

  button {
    font-size: 2rem;
    font-family: 'Montserrat', sans-serif;
    font-weight: 200;
    text-align: center;
    text-transform: uppercase;
    background-color: #222222;
    border: 0.15rem solid #7d7d7d;
    color: white;
    border-radius: 0.5rem;
    padding: 1%;
  }

  button:hover {
    background-color: #7d7d7d;
    color: black;
  }

  .enterbutton {
    border-radius: 0;
    padding: 0.5%;
  }

  p {
    max-width: 14rem;
    margin: 1rem auto;
    line-height: 1.35;
  }

  .desktop-title {
    display: block;
  }

  .mobile-title {
    display: none;
    font-size: 3.5rem;
    font-weight: 400;
    margin-top: 2rem;
  }

  .mobileenable {
    display: none;
  }

  @media screen and (max-width: 480px) {
    .mobile-title {
      display: block;
    }

    .header {
      padding: 2% 0;
      display: none;
    }
    
    .title {
      font-size: 1.5rem;
      flex-basis: 0;
      display: none;
    }

    .select {
      font-size: 1.5rem;
      padding: none;
      flex-basis: 50%;
    }

    .buttons {
      justify-content: center;
      margin-top: 5%;
    }

    .hint {
      font-size: 2.25rem;
      max-width: 90%;
      margin: 0 auto;
      font-weight: 300;
    }

    .image {
      min-width: 80%;
      max-width: 80%;
      min-height: 15rem;
      max-height: 15rem;
    }

    .inputbox {
      width: 80%;
    }

    .searchresult {
      width: 80%;
      font-size: 1.5rem;
    }

    .enterbutton {
      padding: 2%;
    }

    .window {
      width: 85%;
      max-height: 60vh;
      overflow: scroll;
    }

    .gameresult {
      font-size: 3rem;
    }

    .statstext {
      max-width: 90%;
      text-align: left;
      margin: 4% auto;
      font-size: 2rem;
    }

    h1 {
      font-size: 3rem;
    }

    .infotext {
      font-size: 1.5rem;
      text-align: left;
      line-height: 1.25;
    }

    .inputbox:disabled {
      color: white;
    }

    .mobiledisable {
      display: none;
    }

    .mobileenable {
      display: block;
    }

    .resultchild {
      width: 20%;
      max-width: 20%;
      font-size: 1rem;
      line-height: 2;
      overflow: hidden;
    }

    .result {
      width: 95%;
    }

    .tiktokbutton {
      color: white;
      margin: 0 1%;
    }

    .infobutton {
      color: white;
      margin: 0 1%;
    }

    .statsbutton {
      color: white;
      margin: 0 1%;
    }

    .statsbutton:hover {
      color: gray;
    }

    .infobutton:hover {
      color: gray;
    }

    .tiktokbutton:hover {
      color: gray;
    }

    .correctanswer {
      font-size: 2rem;
    }
  }
</style>
