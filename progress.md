# Memory Cards Game Code Progress

### **index.html**
#### Building the basic HTML structure:
```
<html>
	
<head>
    <title>Loopers' name Memory Cards Game</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>
	<h1>Loopers' Memory Cards Game</h1>
    <div id="cards">
        
    </div>
    
	<p>Score: <span id="score"></span></p>
	
	<div id="actions">
        <button>Restart</button>
    </div>
	
	<script src="script.js"></script>
</body>

</html>
```
---
### **style.css**
#### Each Looper should now customize the following elements with CSS (here's an example):
```
body {
  min-height: 100vh;
  min-width: 100vh;
  background-color: #12181f;
  color: white;
}

h1 {
    text-align: center;
    font-weight: 700;
    font-size: 50px;
}

p {
    text-align: center;
    font-size: 30px;
    font-weight: bold;
}

#cards {
  display: grid;
  justify-content: center;
  grid-gap: 16px;
  grid-template-columns: repeat(6, 140px);
  grid-template-rows: repeat(2, 210px);
}


.card {
  height: 210px;
  width: 140px;
  border-radius: 10px;
  background-color: white;
  position: relative;
  transform-style: preserve-3d;
  transition: all 0.5s ease-in-out;
}


.front-image {
  width: 60px;
  height: 60px;
}

```
---
### **script.js**
#### Now we start building the backend/Logic of the game:
The memory cards game starts with all of the cards laid face down on a surface and two cards are flipped face up over each turn.
We will start by initializing some variables and arrays
Here you need to explain querySelector. Because we will use it to target different elements.
```
const cardsContainer = document.getElementById("cards");
let cards = [];
let firstCard, secondCard;
let lockBoard = false;
let score = 0;

// The score is stored in this variable and we target the element by querySelector
document.getElementById("score").textContent = score;
```

```
fetch("./data/cards.json")
  .then((res) => res.json())
  .then((data) => {
    cards = [...data, ...data];
	// Here we will call to functions: shuffleCards() & generateCards(), but we will do that after we write them.
	console.log(cards); // for now console log the cards array to show the Loopers that it indeed fetched the cards and stored them in an array of objects.
  });

```
Now we will build the shuffleCards() function:
In this part of the code we need to explain the Fisher-Yates Algorithm:
```
function shuffleCards() {
    let currentIndex = cards.length;
    let randomIndex;
    let temporaryValue;
  while (currentIndex !== 0) {
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex -= 1;
    temporaryValue = cards[currentIndex];
    cards[currentIndex] = cards[randomIndex];
    cards[randomIndex] = temporaryValue;
  }
}
```
After we write the function we call the function in the fetch and show the Loopers that it indeed shuffled the cards by console.log(cards).
```
fetch("./data/cards.json")
  .then((res) => res.json())
  .then((data) => {
    cards = [...data, ...data];
	shuffleCards();
	console.log(cards); // console log the cards array to show the Loopers that it indeed fetched the cards and stored them in an array of objects.
});
```
A for...of statement executes a loop that operates on a sequence of values sourced from an iterable object.
```
for(let ____ of _____) {
	// Code to be iterated over here
}

```
Using the for...of, we will iterate over the cards array which has this kind of object:
```
{image: '../assets/tomato.png', name: 'tomato'}
```
Now we will implement the generateCards() function:
Here we need to explain createElement, and how we can create new elements in our HTML file using JavaScript
```
function generateCards() {
    for (let card of cards) {
        const cardElement = document.createElement("div");
        cardElement.classList.add("card");
        cardElement.setAttribute("data-name", card.name);
        cardElement.innerHTML = `
        <div class="front">
            <img class="front-image" src=${card.image} />
        </div>
        <div class="back"></div>
        `;
        cardsContainer.appendChild(cardElement);
        // Here we will add the event listener after we implement the flipCard() function
    }
}
```
flipCard() function checks if the card is flipped, and if it's not it flips the card
```
function flipCard() {
  if (lockBoard) return;
  if (this === firstCard) return;

  this.classList.add("flipped");

  if (!firstCard) {
    firstCard = this;
    return;
  }

  secondCard = this;
  document.getElementById("score").textContent = score;
  lockBoard = true;

  // Here we will call the checkForMatch() function, after we implement it
}
```

Now we need to add the event listener to the generateCards() function to call the flipCard() function since we implemented it
```
function generateCards() {
    for (let card of cards) {
        const cardElement = document.createElement("div");
        cardElement.classList.add("card");
        cardElement.setAttribute("data-name", card.name);
        cardElement.innerHTML = `
        <div class="front">
            <img class="front-image" src=${card.image} />
        </div>
        <div class="back"></div>
        `;
        cardsContainer.appendChild(cardElement);
        cardElement.addEventListener("click", flipCard);
    }
}
```

For now we will implement the checkForMatch() function, which basically checks if the first and second cards match
```
function checkForMatch() {
  let isMatch = firstCard.dataset.name === secondCard.dataset.name;

  isMatch ? disableCards() : unflipCards();
}
```

```
isMatch ? disableCards() : unflipCards();

// is another way of typing

if(isMatch) {
	disableCards();
}
else {
	unflipCards();
}
```

Now we need to call the checkForMatch() function from the flipCard() function
```
function flipCard() {
  if (lockBoard) return;
  if (this === firstCard) return;

  this.classList.add("flipped");

  if (!firstCard) {
    firstCard = this;
    return;
  }

  secondCard = this;
  document.getElementById("score").textContent = score;
  lockBoard = true;

  checkForMatch();
}
```

This function is in order to unlock the board and to reset the two cards back since they were not guessed:
```
function unlockBoard() {
  firstCard = null;
  secondCard = null;
  lockBoard = false;
}
```

Function to disable guessed cards:
```
function disableCards() {
  firstCard.removeEventListener("click", flipCard);
  secondCard.removeEventListener("click", flipCard);
  score++;
  unlockBoard();
}
```

Function to unflip the cards back since they don't match:
```
function unflipCards() {
  setTimeout(() => {
    firstCard.classList.remove("flipped");
    secondCard.classList.remove("flipped");
    unlockBoard();
  }, 1000);
}
```

Now we need to implement the function that resets the game when we click on the restart button:
```
function restart() {
  unlockBoard();
  shuffleCards();
  score = 0;
  document.getElementById("score").textContent = score;
  cardsContainer.innerHTML = "";
  generateCards();
}
```
---
### **index.html**
#### Linking the restart() function to the button:
To link function to the HTML, add the following in the button element:
```
	<div id="actions">
        <button onclick="restart()">Restart</button>
    </div>
```
---
### **style.css**
#### Continue to customize the following elements with CSS:

In order to style the button we will add the following
```
#actions {
    display: flex;
    justify-content: center;
}

#actions button {
    padding: 8px 16px;
    font-size: 30px;
    border-radius: 10px;
    background-color: #27ae60;
    color: white;
}
```

```
.card.flipped {
	transform: rotateY(180deg);
}

.front, .back {
    backface-visibility: hidden;
    position: absolute;
    border-radius: 10px;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}
```

In order to center the image in the center of the cards we need to add the following lines to the style:
```
.card .front {
	display: flex;
	justify-content: center;
	align-items: center;
}
```

Now we need to style the back of the card:

```
.card .back {
	background-image: url(""); // here we will add the link to any image/from pattern monster
	background-position: center center;
	background-size: cover;
	backface-visibility: hidden;
}
```
Now in order to show the card front we need to add this line:
```
.card .front {
	transform: rotateY(180deg);
}
```
---
### **style.css**
#### Make the game mobile responsive
```
@media (max-width: 1000px) {
    /* Adjust font sizes */
    h1 {
      font-size: 36px;
    }
    .cards {
      grid-template-columns: repeat(3, 140px);
      grid-template-rows: repeat(4, 210px);
      grid-gap: 10px;
    }
  
    .card {
      height: 210px;
      width: 140px;
    }
  
    /* Adjust padding and font size for the restart button */
    .actions button {
      padding: 10px 20px;
      font-size: 24px;
    }
}
```
### **script.js**
#### Make the game mobile responsive
We need to add this line in generateCards() function
```
	...
	cardElement.addEventListener("touchstart", flipCard);
```

And these two lines in disableCards() function
```
	...
    firstCard.removeEventListener("touchstart", flipCard);
    secondCard.removeEventListener("touchstart", flipCard);
	...
```
---
### **index.html**
#### Adding Confetti effect: (EXTRA)

We will add this line to the body of the HTML file
```
<script src="confetti.js"></script>
```

```
<body>
    <h1>Loopers' Memory Cards Game</h1>
    <div id="cards">
        
    </div>
    
	<p>Score: <span id="score"></span></p>
	
	<div id="actions">
        <button onclick="restart()">Restart</button>
    </div>
    <script src="confetti.js"></script>
	<script src="script.js"></script>
</body>
```

### **script.js**
#### Adding Confetti effect: (EXTRA)

We need to check if the score matches the number of objects/2 if yes we can start the Confetti effect
```
function disableCards() {
    firstCard.removeEventListener("click", flipCard);
    secondCard.removeEventListener("click", flipCard);
    score = score + 1;
    if(score === 9) {
        startConfetti();
    }
    unlockBoard();
}
```

Also when clicking on the restart button we will need to add stopConfetti() in order to stop the effect
```
function restart() {
  unlockBoard();
  shuffleCards();
  score = 0;
  document.getElementById("score").textContent = score;
  cardsContainer.innerHTML = "";
  generateCards();
  stopConfetti();
}
```
