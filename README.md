# Risk

Conquer all 42 territories in a turn-based console game for **3-6 human players**.
Players share one terminal, claim their starting territories, build armies, attack
their neighbors, and fortify their borders until one player controls the world.

## Start Playing

Install **JDK 11** and Git. Gradle is included through the wrapper; you do not need
to install it separately.

```sh
git clone https://github.com/AnantPoddar1004/risk-game.git
cd risk-game
./gradlew classes
java -cp build/classes/java/main:build/resources/main code.Main
```

On Windows, build with `gradlew.bat classes`, then run:

```powershell
java -cp "build/classes/java/main;build/resources/main" code.Main
```

The first prompt is `Enter number of players:`. Enter one answer per line and
press **Enter**. The direct Java command above connects the game to your terminal;
the Gradle `run` task does not currently forward keyboard input.

## Set Up the Board

### 1. Register Players

Enter an integer from `3` to `6`, then a name and an available color for each
player. Choose from `red`, `blue`, `green`, `yellow`, `black`, and `purple`.
Colors are case-insensitive, and each player must choose a different color.
Use distinct player names so elimination messages and turn tracking are unambiguous.

```text
Enter number of players: 3
Enter player 1 name: Alice
Enter color for Alice: red
Enter player 2 name: Bob
Enter color for Bob: blue
Enter player 3 name: Cara
Enter color for Cara: green
```

The game randomly chooses the starting player. Turns then follow registration
order, wrapping back to the first player.

### 2. Claim Territories

The game displays the current player, unclaimed territories grouped by continent,
and the territories that player already owns. At each claim prompt, enter **one
unclaimed territory name**:

```text
Enter territory to claim: Alaska
```

A successful claim places one Infantry there and advances to the next player.
Continue until all 42 territories are claimed. Unknown names or already claimed
territories are rejected, and the same player tries again.

Use the displayed territory names, including spaces: `Northwest Territory`,
`Western United States`, or `New Guinea`. Do not put quotes around names.

### 3. Place Your Remaining Starting Armies

Each player starts with the following total, including armies used to claim territories:

| Players | Starting armies per player |
| --- | --- |
| 3 | 35 |
| 4 | 30 |
| 5 | 25 |
| 6 | 20 |

Players take turns placing **one Infantry at a time** on territories they own:

```text
Enter territory to place army: Alaska
```

For this setup prompt, enter only the name, without an army count. Once everyone
has placed all starting armies, the game announces that setup is complete.

## Play a Turn

Every turn has three phases: **reinforce, attack, fortify**. Finish placing all
reinforcements before attacking. Attacking and fortifying are optional.

### 1. Reinforce

Your new armies are calculated automatically:

```text
territory armies = max(3, floor(owned territories / 3))
total reinforcements = territory armies + continent bonuses + card trade-in bonus
```

Owning 8 territories gives 3 armies; owning 12 gives 4. You receive a continent
bonus only if you own every territory in that continent:

| Continent | Territories | Bonus armies |
| --- | --- | --- |
| North America | 9 | 5 |
| South America | 4 | 2 |
| Europe | 7 | 5 |
| Africa | 6 | 3 |
| Asia | 12 | 7 |
| Australia | 4 | 2 |

#### Trade Cards When Prompted

If a trade is available, the game displays your cards with numbered indices:

```text
1: INFANTRY, 2: CAVALRY, 3: ARTILLERY
Enter card indices to trade in: 1 2 3
```

Enter exactly **three different indices**, separated by spaces. Indices start at
`1`, not `0`. A valid set contains three of the same non-wild type, one of each
non-wild type, or one Wild and two non-wild cards. At most one Wild is allowed.

Press **Enter on an empty line** to skip an optional trade. With five or more
cards, a trade is required and cannot be skipped. Invalid selections are rejected
and prompted again. Traded cards leave your hand and enter the discard pile.

Trade bonuses follow the number of sets traded by **all players combined**:

| Set traded | Bonus armies |
| --- | --- |
| 1st | 4 |
| 2nd | 6 |
| 3rd | 8 |
| 4th | 10 |
| 5th | 12 |
| 6th | 15 |
| 7th-14th | 20, 25, 30, 35, 40, 45, 50, 55 |

#### Place Reinforcements

The game displays your available pieces, for example `{INFANTRY=5}`. Placement
input must end with **three counts in this order**:

```text
<territory name> <infantry count> <cavalry count> <artillery count>
```

These are piece counts. Infantry is worth **1 army**, Cavalry **5 armies**, and
Artillery **10 armies**. They represent army quantities, not different combat abilities.

To place five armies as Infantry:

```text
Enter territory and armies to place: Alaska 5 0 0
```

To split five reinforcements across two owned territories, answer successive prompts:

```text
Enter territory and armies to place: Alaska 3 0 0
Enter territory and armies to place: Alberta 2 0 0
```

For a name containing spaces:

```text
Enter territory and armies to place: Western United States 3 0 0
```

Counts must be nonnegative, with at least one positive count. You must own the
territory and have enough available army value. Use Infantry-only input such as
`Alaska 5 0 0` for the simplest placement. The phase continues until your entire
available army value has been placed.

### 2. Attack

Enter `yes` to attack or `no` to stop attacking. `y` and `n` also work,
case-insensitively. If no legal attacks exist, the game skips this phase.

Choose a territory you own with at least **2 armies**, then an **adjacent enemy
territory**. Enter each name at its own prompt:

```text
Do you want to attack? (yes/no): yes
Enter attacking territory: Alaska
Enter defending territory: Northwest Territory
Enter number of dice for Alaska: 3
Enter number of dice for Northwest Territory: 1
```

This example requires at least 4 armies in Alaska and an enemy-owned Northwest
Territory with at least 1 army. The players sharing the terminal enter their
respective dice choices; these are the **number of dice**, not desired roll results.

| Side | Allowed dice count |
| --- | --- |
| Attacker | 1-3, never more than the attacking army count minus 1 |
| Defender | 1-2, never more than the defending army count |

The game rolls and sorts both sides' dice. Highest faces highest, and second-highest
faces second-highest if both sides have a second die. Each comparison removes one
army from the loser; **the defender wins ties**. The result shows both rolls,
losses, and remaining army counts.

Each attack resolves **one battle roll**. Answer `yes` again to continue fighting
the same territory or choose another legal attack. Answer `no` to end the phase.

#### Capture a Territory

When the defending army count reaches zero, enter the number of armies to move:

```text
Enter armies to move from Alaska to Northwest Territory: 3
```

Leave at least one army in the attacking territory. The minimum move is the
number of attacker dice used in the final battle, or all movable armies if fewer
are available. The maximum is the attacking territory's remaining army count minus 1.
Invalid movement is rejected until you enter a legal count.

The captured territory becomes yours. If this was the defender's last territory,
that player is eliminated and their cards transfer to you. Future turns skip
eliminated players.

If you captured at least one territory during this attack phase, you receive
**exactly one card** when the phase ends, regardless of how many territories you
captured. An empty draw pile is replenished by shuffling the discarded cards.

### 3. Fortify

Enter `no` to finish your turn immediately, or `yes` to make one fortification move:

```text
Do you want to fortify? (yes/no): yes
Enter source territory: Alaska
Enter destination territory: Alberta
Enter number of armies to move: 2
```

Both territories must be yours and connected by a path entirely through your
territories. They need not be directly adjacent. Move at least one army and leave
at least one behind at the source. The number is an **army value**, not three
piece counts. An invalid move prompts you to enter the source, destination, and
count again. One successful fortification ends this phase.

## Win the Game

Control **all 42 territories** to win. The game checks for victory after the full
turn completes, including the fortify phase, displays `<player name> won the game.`,
and stops. After taking the final territory, finish or skip fortification to see
the winner announced.

## Input Reference

| Prompt | Example answer |
| --- | --- |
| Number of players | `3` |
| Player name | `Alice` |
| Player color | `red` |
| Claim a territory | `Alaska` |
| Place one setup army | `Alaska` |
| Trade three cards | `1 2 3` |
| Skip an optional card trade | Empty line |
| Place reinforcements | `Alaska 3 0 0` |
| Attack or fortify? | `yes` / `no` or `y` / `n` |
| Attacking or defending territory | One territory name per prompt |
| Number of dice | `1`, `2`, or `3`, as allowed |
| Capture or fortify movement | A single integer, such as `3` |

## Current Limits

- Local human players only; there is no AI, graphical board, or network play.
- Games are not saved. **Ctrl+C** exits; launching again starts a new game.
- Enter an integer at the player-count prompt. Text or a blank line currently
  terminates startup instead of prompting again; out-of-range integers are rejected.
- Territory listings show ownership grouped by continent, but not army counts or
  a map. Battle results show the two participating territories' remaining armies.
  Keep track of placements and use a standard Risk map for planning adjacency.
- Card trading currently processes one set per turn and supports at most 14 sets
  across the game. A 15th trade raises an error even though discards can be reshuffled.
- Card territory-match bonuses are not implemented; card trades award the set bonus only.

## Development

Run unit and integration tests:

```sh
./gradlew test -x pitest
```

Run style checks or mutation testing separately:

```sh
./gradlew checkstyleMain checkstyleTest
./gradlew pitest
```

The `-x pitest` flag skips the mutation test task that otherwise runs after the
JUnit suite. Boundary value analyses are in `docs/bva/`, and class diagrams and
design notes are in `docs/design/`.
