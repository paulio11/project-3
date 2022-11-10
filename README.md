# Hangman - Python Terminal Game

![Image of website on multiple devices](#)

[Link to live GitHub deployment](#)

## Contents 



## Introduction
Hangman is a Python terminal game. It runs on a mock terminal on Heroku.

Hangman is my third milestone project required to complete my Diploma in Full Stack Software Development at The Code Institute. This project required me to showcase my newly learned Python skills to create a command-line application.

The goal of the game is to guess the mystery word one letter at a time. The time taken, the number of incorrect guesses and word length will determine their score and their position on the scoreboard.

[Back to top 🔺](#hangman---python-terminal-game)

## The User Experience

### User Goals
As a user I want:
- To have fun and be challenged.
- To learn how to play and understand the game.
- The game to offer replay value with a large number of words and categories to play with.
- To easily navigate through available options and have clear feedback on my inputs.
- To know how weel I did and be able to compare myself to others.

### Project Goals
As the designer I want:
- Users to meet their goals (above).
- The game to be functional and without errors.

[Back to top 🔺](#hangman---python-terminal-game)

## Design and Flow

### Flow Chart
[Lucid Chart](https://www.lucidchart.com/) was used to plan out the flow of the program, from functions to user input/variable checks.

![Flow chart](https://raw.githubusercontent.com/paulio11/project-3/main/documentation/flowchart-updated.png)

### Flow Chart with Function Names and Variables

![Flow chart with Functions and Variables](https://raw.githubusercontent.com/paulio11/project-3/main/documentation/flowchart-functions.png)

### Titles and Headings
- The titles for the scoreboard and category selection screens were made using the [pyfiglet](https://pypi.org/project/pyfiglet/0.7/) package.
- The game over and hangman titles were made using [Text ASCII Art Generator](https://patorjk.com/software/taag/).

[Back to top 🔺](#hangman---python-terminal-game)

## Features
### User Input and Feedback
This user input design and positioning are constant throughout the program. Always below a dotted line and on the same line in the terminal.
Positive feedback is represented with green text. Negative feedback or invalid input is represented with red text. This functionality is part of the [colorama](https://pypi.org/project/colorama/) package. Consistency improves the experience for the user.

### Main Menu
The first screen presented to the user is the main menu. It shows the game title, some ASCII art and a row of options. Below the dotted line is a python input field waiting to accept the user's choice.

Created by the `main_menu()` function. User input is verified by an if-else statement and an array of valid user inputs.

```
valid_choices = ['1', '2', '3', '4']
```

### How To Play
A simple text-only screen. Contains instructions for the user on how to play and win the game. 

Beneath is a dotted line and the text "Press ENTER to return to the main menu...". Input from the user here will run the `main_menu()` function and return them to the main menu. This is repeated throughout the program so to avoid repeated lines of code it is all part of a function called `bottom_input()` that is called when necessary.

```
def bottom_input():
    print('-' * 80)
    input('Press ENTER to return to the menu...')
    main_menu()
```

### High Scores and Last Five Scores
Using the `get_all_values()` function from the [gspread](https://docs.gspread.org/en/latest/) google sheets plugin, data is pulled from a spreadsheet containing all player scores. The data is sliced and sorted in different ways depending on the option selected by the user from the main menu. 

Last Five Scores `last_five_scores()` slices the bottom 5 rows from the spreadsheet and then sorts the arrays using a lambda function by the values in the second (score) column. This data is then passed to the `draw_table()` function.

```
def last_five_scores():
    score_data = SCORE_SHEET.get_all_values()
    score_slice = score_data[-1:-6:-1]
    score_slice = sorted(score_slice, key=lambda x: int(x[1]), reverse=True)
    draw_table(score_slice, 'Latest Scores', 'Last 5 Scores:')
```

High Scores `highscores()` sorts all data pulled from the spreadsheet again with a lambda function by the values in the score column. Then passes just the first 5 arrays to the `draw_table()` function.

```
def highscores():
    score_data = SCORE_SHEET.get_all_values()
    score_sorted = sorted(score_data, key=lambda x: int(x[1]), reverse=True)
    draw_table(score_sorted[:5], 'High Scores', 'All Time Top 5 Scores:')
```

`draw_table()` takes the data and uses a for loop to print the arrays one at a time as a presentable table of scores.

```
def draw_table(scores, heading, heading2):
    rank = 1
    rank_h = 'RANK'
    name_h = 'NAME'
    score_h = 'SCORE'
    time_h = 'TIME'
    headers = f'{rank_h : <6}{name_h : <12}{score_h : <9}{time_h : <5}'
    print(headers)
    print('=' * 33)
    for line in scores:
        row = f'{rank : <6}{line[0] : <12}{line[1] : <9}{line[2] : <5}'
        cprint(row)
        cprint('-' * 33)
        rank += 1
```

[Back to top 🔺](#hangman---python-terminal-game)

## The Game

### Category Selection and The Game Word
The game itself features several categories the user can select from. The selected category changes the variable `WORD_LIST`. Each column in the word_list spreadsheet represents the possible words for each category.

A random word is pulled using the python [random](https://docs.python.org/3/library/random.html) module and assigned as the variable `GAME_WORD`. This will be the word the user has to guess as the object of the game. 

```
GAME_WORD = random.choice(word_list)
```

During gameplay, this mystery word is represented by a line of underscores. This is the `HIDDEN_WORD` variable, initially created by assigning it the value of one underscore multiplied by the length of the game word.

```
HIDDEN_WORD = '_' * len(GAME_WORD)
```

### The Main Game Screen
The most prominent feature of the game screen is the ASCII art landscape showing a castle, the hangman platform, and a tree. This is the first index of an array called `HANGMAN_STAGES`. Throughout the game, if the user makes an incorrect guess the `GAME_STAGE` variable is iterated by 1, and then when the game screen is reprinted the next ASCII art landscape in the array is called.

```
print(HANGMAN_STAGES[GAME_STAGE])
```

Beneath the game stage is a line containing two pieces of important information for the user. First, is a list of letters already guessed. This information is useful to the player when making further guesses. 

```
guessed_letters = []
guessed_letters.append(guess)
guessed_letters_str = ' '.join(guessed_letters)
```

Second, is a string of hearts `♥` representing the number of incorrect guesses left before a game over. Upon an incorrect guess, the variable `PLAYER_LIVES` is iterated by -1.

```
life_bar = ' ♥' * PLAYER_LIVES
```

As with all screens so far, the user input is on the same line of the terminal, below a line of dashes. This is where the user will make their guess, either one letter at a time, or as a whole word.

The user input once validated by `user_input()` is passed to the `check_guess()` function. There are three outcomes of this function, either the guess is correct and the function `update_hidden_word()` is called, the user is out of guesses and the game is over, or the guess is correct and the game stage is redrawn so the game can continue.

### Updating the Hidden Word
The `update_hidden_word()` function is fairly complex but it can be broken up into a few simple actions. 
1. It will take the user's guess and look for that letter in the `GAME_WORD` variable, returning an array of numbers, each representing the position of the guess in the word. 

```
positions = [i for i, a in enumerate(GAME_WORD) if a == guess]
```

2. Then it creates a new variable from `HIDDEN_WORD` using the `list` function to return an array. 

```
hidden_word_arr = list(HIDDEN_WORD)
```

3. A `for` loop then assigns the value of the user's guess to the correct positions in the array.

```
for num in positions:
        hidden_word_arr[num] = guess
```

4. `HIDDEN_WORD` is then reassigned a value using the `join` function.

```
HIDDEN_WORD = ''.join(hidden_word_arr)
```

At this point, it is possible the user has guessed correctly all the letters in the game word. If so the `game_win_trigger()` function can be called. An easy way to check for this is whether or not any underscores remain in the hidden word.

```
if '_' not in HIDDEN_WORD:
        game_win_trigger()
```

### Game Over Screen
When the variable `GAME_STAGE` has a value of 9, the game is over. This will occur after that many incorrect guesses from the user. This check is at the bottom of an if statement in the `check_guess()` function.

```
        END_TIME = time.time()
        PLAYER_LIVES -= 1
        GAME_STAGE += 1
        GAME_OVER = True
        game_display(FAIL_HEADER)
```

When `GAME_OVER` is `True` and the game display is reprinted it will show the game over title instead, as well as the reveal of the game word to the player. `GAME_OVER` being `True` also stops the `while` loop within the `user_input()` function, and therefore will no longer ask the user to input a guess. 

```
while GAME_OVER is False:
```

```
elif GAME_OVER and GAME_WIN is False:
        left_text = 'Oh dear he died!'
        right_text = f'The Mystery {CATEGORY} was {GAME_WORD}.'
        print(f'{left_text : <25}{right_text : >55}')
```

The function `bottom_input` is then called once again to return the user to the main menu.

### Winning the Game
There are two ways to trigger a game win state. If the user guesses the game word in its entirety, or if there are no underscores left in the `HIDDEN_WORD` variable. Both of these outcomes will call the `game_win_trigger()` function.

```
elif guess == GAME_WORD:
            HIDDEN_WORD = GAME_WORD
            game_win_trigger()
```
```
if '_' not in HIDDEN_WORD:
        game_win_trigger()
```

The `game_win_trigger()` function will assign the variables `GAME_WIN` and `GAME_OVER` to `True`, set a variable called `END_TIME` (used to calculate the score later on) and call the `game_display()` function.

```
def game_win_trigger():
    END_TIME = time.time()
    GAME_WIN = True
    GAME_OVER = True
    game_display(WIN_HEADER)
```

When the game screen is reprinted with the variable `GAME_WIN` as `True` two functions are called, first `calculate_score()` and then `update_scoreboard()`.

### Calcuating the Score
The `calculate_score()` function has three parts. First, the variable `SECONDS` is assigned the value of `END_TIME` subtracted from `START_TIME` to get the seconds taken to win the game. The python [math](https://docs.python.org/3/library/math.html) module is used to round down the value to a whole number.

```
SECONDS = math.floor(END_TIME - START_TIME)
```

Then the score is calculated using an algorithm that takes into account the length of the word, the number of player lives remaining and the time taken. This is then rounded up using the `math.ceil()` function.

```
SCORE = math.ceil((len(GAME_WORD) * 500) + (PLAYER_LIVES * 1000) / SECONDS)
```

The values assigned have very little thought and design put into them, but basically, this makes longer words solved with a low number of incorrect guesses in a short time score higher than short words solved with many incorrect guesses in a longer amount of time. As the game time progresses the effect of the time taken has a diminishing effect on the final score.

### End of the Game
Just for user feedback purposes, once the scoreboard has been updated the user is presented with a simple screen thanking them for playing, this includes the name they provided previously and the option to return to the main menu.

```
def end_screen():
    print(f'Thank you for playing {NAME}!')
    print('Your name and score have been uploaded.')
```

[Back to top 🔺](#hangman---python-terminal-game)

## Common Features
To stop me from repeating myself in this documentation and my code, several features have been present throughout the program.

### Printing Center Aligned Text
I created a function called `cprint()` to center align text. This was to help with line length issues. 

```
def cprint(text):
    terminal_width = 80
    print(text.center(terminal_width))
```

### "Loading" Text
I have loading text where necessary to give the user feedback when the following takes place:
- Loading either scoreboard
- Updating the scoreboard
- Retrieving the word list after category selection

All three of these require information to be pushed to or pulled from the spreadsheet and can take a few seconds. The loading text reassures the user something is happening.

### Clearing the Terminal
A function called `clear_terminal()` is called throughout to clear the screen before printing what is required at that point in the program. This was essential to make sure all user input is on the same line - a key part of my design. This was a suggestion from [Stack Overflow](https://stackoverflow.com/questions/2084508/clear-terminal-in-python).

```
def clear_terminal():
	os.system('cls' if os.name == 'nt' else 'clear')
```

[Back to top 🔺](#hangman---python-terminal-game)

## Unimplemented Features

### Guessing Whole Words Incorrectly
Currently, the game will not penalise the user if they guess incorrectly when inputting an incorrect whole word. This will just show as an invalid guess. I would like the game to be able to distinguish between an invalid guess and an incorrect whole word guess. This would require a more complex if-else statement that I did not have the time to implement.

### More User Data
This would provide no extra functionality but there are many more values and details I can upload to the spreadsheet upon scoreboard entry. Things like the order of letters guessed, the time between guesses, whole words guessed and so on. All would benefit me as the designer from a gameplay perspective. This feedback could lead to improvements in word choice or difficulty. To implement this, new variables would have to be created throughout the program, and then each appended to the spreadsheet in the same way as the user's name and score currently are.

### Further Word and Category Expansion
And of course more simple things such as a larger word list and category choice. This is simple to implement but would add no extra value to this program as a portfolio project. 

[Back to top 🔺](#hangman---python-terminal-game)