<script setup lang="ts"> 
  import { ref, nextTick, computed } from "vue";
  import words from "./database/words.json"
  
  let answer = ref<string[]>(['m', 'o','r','a','n']);
  const guessedWord = ref<string[]>(Array(5).fill(''));  
  const guessedWords = ref<EvaluatedGuess[][]>([]);    
  const inputs = ref<HTMLInputElement[]>([])
  let correctLetters = ref<number>(0);
  let turnsTaken = ref<number>(0);
  let turnsRemaining = ref<number>(5);
  let win = ref<boolean>(false);
  let lose = ref<boolean>(false);
  let gamesWon = ref<number>(0);
  let gamesPlayed = ref<number>(0);

  type LetterState = 'correct' | 'present' | 'absent'
  type EvaluatedGuess = {
    letter: string,
    state: LetterState
  }

  const wordBank = computed<string[][]>(() => {
    return words.words.map(words => words.split(''))
  })

  const wordSelect  = async () => {
    var chosenNumber = Math.floor(Math.random() * wordBank.value.length);
    answer.value = wordBank.value[chosenNumber];
    console.log(answer.value);

    correctLetters.value = 0;
    turnsTaken.value = 0;
    turnsRemaining.value = 5;
    win.value = false;
    lose.value = false;
    guessedWords.value = [];
    
    
    await nextTick()
    focusInput(0);
  }

  const focusInput = async (index: number) => {
    await nextTick();
    if (inputs.value[index]) {
    inputs.value[index]?.focus();
    }
  }

  const handleInput = (index: number) => {
    if (index < inputs.value.length -1) {
      focusInput(index + 1);
    }
  }

  const checkWin = () => {
    if (correctLetters.value === 5) {
      win.value = true;
      gamesWon.value = gamesWon.value + 1;
      gamesPlayed.value = gamesPlayed.value + 1;
      console.log('win: ' + win.value);

    } else if (turnsTaken.value === 6) {
      lose.value = true;
      gamesPlayed.value = gamesPlayed.value + 1;
      console.log('lose: ' + lose.value);
    }
  }


const addToArray = () => {
  const guess = guessedWord.value.map(l => l.toLowerCase());
  const target = answer.value.map(l => l.toLowerCase());
  
  // Initialize result array with placeholders
  const evaluation = Array(5).fill(null).map((_, i) => ({
    letter: guess[i],
    state: 'absent' as LetterState
  }));

  // 1. Create a frequency map of the answer
  const letterCounts: Record<string, number> = {};
  for (const letter of target) {
    letterCounts[letter] = (letterCounts[letter] || 0) + 1;
  }

  // 2. First Pass: Find all 'correct' (green) matches first
  guess.forEach((letter, i) => {
    if (letter === target[i]) {
      evaluation[i].state = 'correct';
      letterCounts[letter]--;
      correctLetters.value++;
    }
  });

  // 3. Second Pass: Find 'present' (yellow) matches for what's left
  guess.forEach((letter, i) => {
    // Only check letters that weren't already marked 'correct'
    if (evaluation[i].state !== 'correct') {
      if (letterCounts[letter] > 0) {
        evaluation[i].state = 'present';
        letterCounts[letter]--;
      }
    }
  });

  // Push the final evaluation to history
  guessedWords.value.push(evaluation);


  // const addToArray = () => {
  //   const evaluatedGuess = guessedWord.value.map((letter, index) => {
  //     let state: LetterState = 'absent';
  //     const lowerLetter = letter.toLowerCase()
  //     if(answer.value[index] === lowerLetter) 
  //     {state = 'correct';
  //       correctLetters.value ++ ;
  //     }
  //     else if (answer.value.includes(lowerLetter)) 
  //     {state = 'present'
  //     }
  //     return {letter:lowerLetter, state}
  //   })  
  //   guessedWords.value.push(evaluatedGuess);
    
  
    
    turnsTaken.value ++ ;
    if(turnsRemaining.value > 0) {
      turnsRemaining.value -- ;}
    checkWin();       

    guessedWord.value = Array(5).fill('');
    correctLetters.value = 0;
    focusInput(0);
  }

</script>

<template>
  <main>
    <div class="score">
      <h2>Score: {{ gamesWon }} / {{ gamesPlayed }}</h2>
    </div>
    
    <h1 id="head">Pacman (obviously)</h1>
    <h2 v-if="win">You won! You are so SMART</h2>
    <h2 v-if="lose">Oh no! You ran out of turns.</h2>
    <div 
    v-for="guessedWord in guessedWords"
    class="guessedWord"
    >
      <span 
      v-for="letter in guessedWord"
      :class="letter.state">
      {{ letter.letter }}
      </span>
    </div>
    <div 
    class="input-boxes" 
    @keypress.enter='addToArray'
    v-if="!lose && !win" 
    >
      <input 
      v-for="(_, index) in answer" 
      :key="index"
      v-model="guessedWord[index]"      
      :ref="(el) => {if (el) inputs[index] = el as HTMLInputElement}"
      type="text"
      maxlength="1"
      @input="handleInput(index)"
      />
    </div>
    <div 
    v-for="remaining in turnsRemaining"
    class="futureTurns"
    >
      <span v-for="num in answer" ></span>      
      
    </div>
    <div>
      
    </div>
   <button 
    v-if="win || lose"
    class="playAgain"
    @click="wordSelect()"
    >Play again you fool</button>
  </main>  
</template>

<style scoped>

main {
  background: aliceblue;
  height: 100vh;
  width: 100vw;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-flow: column wrap;
  color: black
}

.score {
  position: absolute;
  padding: 10px;
  top: 10px;
  right: 10px;
  border: 3px solid black;
  
}

.input-boxes {
  display: flex;
  justify-content: center;  
  margin-bottom: 10px;
  margin: 10px;  
}

input, span  {
  border: 2px solid green;
  border-radius: 8px;
  width: 50px;
  height: 50px;
  background-color: whitesmoke;
  text-align: center;
  font-size: large;
  padding: 10px;
  margin: 5px;
}

span {
  display: inline-block;
}

.correct {
  background-color: green;
}

.present {
  background-color: yellow;
}

.logo {
  margin: 0 2rem 0 0;
}

header .wrapper {
  display: flex;
  place-items: flex-start;
  flex-wrap: wrap;
}

button {
  width: 150px;
  height: 50px;
}
</style>