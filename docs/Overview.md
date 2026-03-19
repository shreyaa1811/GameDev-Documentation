# Game Development 101: 2-Day DIY Kit 

# Overview

Unity is a cross-platform game engine used to develop 2D, 3D, AR, and VR applications . It abstracts low-level systems (rendering, physics, memory, input) and provides a component-based framework for building interactive applications using C#.

## Key Learning Outcomes

By the end of this tutorial. You would have learnt to create a 2D roguelike game. A roguelike is a traditional game genre that has taken many forms over the years, but it usually shares some common elements, such as the following:

- It’s procedurally generated, meaning the levels in the game aren't created by a human and thus always the same, but instead they’re assembled randomly by the code, so every time the game is played, the levels are different.
- The game is played on a grid, meaning all entities (player and enemies) move from cell to cell on that grid.
- The game is turn-based, meaning nothing will happen until the player takes an action (move, attack, etc.), which gives players the chance to think strategically between each action.

In this tutorial, you’ll start by creating the Unity project for this game. You’ll then import the assets that will be used throughout the tutorial, set up all the settings for rendering, and set up Unity Version Control. Lastly, before you start creating, you’ll organize ideas about the game to create a plan on how to make it.

## Day 1
Objective:
You will learn how to design and build a 2D roguelike game. You will learn to prioritise and organise tasks, learn some programming concepts, and learn how to use pixel art in unity.

Duration: 3 Hours

### 1. Introduction
 Short note on unity- What is unity, basic functions (game view, scene view,
components, inspector, projects)

### 2. Overview
What a roguelike game. You will learn to create a 2D roguelike project, how to import assets and Architecting.

### 3. Creating the environment
You will learn to create basic game board that randomly generates tiles on start.
• A border for the game board that “traps” the player character in the confines of the game
board.
• Use Unity Version Control to check in changes to your projects’ repository so you don’t lose any progress and can revert the state of your game to a previous one.

### 4. Adding player characters
 You will learn to:
• Created a Player Character Game Object using one of the provided assets.
• Ensured that the Player Character Game Object is rendered in the correct level, so it is
visible on the board.
• Created a Player Controller script that allows the Player Character Game Object to be
controlled by user inputs.

### 5. Add a Mechanic System:
You will learn to add a turn system. This game is a turn-based game, which means it needs
a way to manage those turns.
• Created a Turn Manager script that will track every time the player character moves and
count that as one turn.
• Reworked the initialization code for the game to accommodate the new system.

### 6. Add gameplay elements
Now the game can count turns, You can start introducing gameplay elements like
resource management. By the end of this , You would have learnt
the following:
• Added a food resource that will spawn randomly on the game board at start and that
the player character can collect.
• Used the UI Builder window to create an in game label that shows the amount of food the
player currently has.
• Coded the functionality to have the food resource decrease with each turn of the game and increase every time the player character collects a food resource.

## Day 2

### 7. Add gameplay elements(continuation):
To make the game more challenging, some obstacles will be added to make navigating the
game board less straightforward. By the end of this tutorial, you would have learnt
the following:
• Create wall prefabs.
• Code the functionality to have a random number of Wall Game Objects spawn on the
game board when the game starts.
• Code the functionality that denies the player character access to the tile the Wall Game Object occupies.
• Add the functionality that allows the player character to damage the Wall Game Object, and once destroyed, move to the tile it used to occupy.

### 8. Animation
To increase the polish of a game, you should add animations to your player character for different actions: an idle animation, a walk animation, and an attack animation. By the end of this tutorial, you would have done the following:
• Added an Animator component to the Player Character Game Object to facilitate animation.
• Used the Animation window to create keyframes for the animation using sprites from the Sprite Sheet.
• Used the Animator window to link the Animation states for the player character so that each animation flows fluidly from one to another.

### 9. Adding an NPC character
Nearing the end of the game. You have added all the basic elements of a traditional roguelike except one: an enemy character whose purpose is to increase the game’s difficulty. Adding an enemy to the game will involve everything the attendees have learnt so far. By the end of this tutorial, you would have done the following:
• Added an Enemy Game Object to the game.
• Coded the Enemy Game Object’s functionality so that it seeks out, moves toward, and attacks the player character on its turn.