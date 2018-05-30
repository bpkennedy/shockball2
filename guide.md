Shockball Guide
======
<img width="150" height="150" align="right" src="https://raw.githubusercontent.com/bpkennedy/shockball2/master/public/img/shockballLogo.png"/>

> **<sub>No credits will be required or sent during the Alpha</sub>**
##### Table of Contents

* [Understanding Shockball](#understanding-the-game)
  * [Player Training](#player-training)
  * [Match Simulation](#match-simulation)
* [Playing the Shockball](#playing-the-game)
  * As Player:
    1. [How to sign up as a player](#how-to-sign-up-as-a-player)
    2. [How to train your player](#how-to-train-your-player)
    3. [What skills change when I train](#what-skills-change-when-i-train)
    4. [How to accept or reject a contract](#how-to-accept-or-reject-a-contract)
  * As Team Owner:
    1. [How to create a Team](#how-to-create-a-team)
    2. [How to offer a player a contract](#how-to-offer-a-player-a-contract)
    3. [How to set your team lineup](#how-to-set-your-team-lineup)
* [Common Problems and Questions](#common-problems-and-questions)
  * [The offer contract button is not there](#the-offer-contract-button-is-not-there)
  * [Cannot spend more than available team budget](#cannot-spend-more-than-available-team-budget)
  * [Must offer player a minimum bid equal to their market value](#must-offer-player-a-minimum-bid-equal-to-their-market-value)
  * [How are Player Ratings Determined](#how-are-player-ratings-determined)
  * [How Contracts Work](#how-contracts-work)
  * [How Team Lineups Work](#how-team-lineups-work)
    * [More on energy](#more-on-energy)
    * [Subs](#subs)
  * [What Stats Are Important](#what-stats-are-important)
    * [Position of Ball](#position-of-ball)
    * [Actions That Players Take](#actions-that-players-take)
  * [When Scheduled Server Jobs Happen](#when-scheduled-server-jobs-happen)

## Understanding the Game
Galactic Shockball is a sports manager simulation game that is played on a true-to-life timeline. This means that matches between teams are scheduled across weeks and months (in real-life time).
### Player training
Players can go select their **training** regimen. The selection box will save your changes as soon as you select one of the options. **Important** - the training will NOT run until the training job runs at **18:30 UTC**

Players can train to certain regimens based on any role they prefer:
* **General** training: No stat changes, but Morale/Leadership/Energy increase slightly.
* **Rest** training: All skills slight decay, but Morale/Energy significant increase and slight Leadership increase.
* **None Selected** training: All skills slight decay as well as Morale and Leadership.  Only slight Energy increase.

### Match Simulation
To describe how the match simulation engine works, I will walk through an example of a tick.  I will preface once that everything here is **current** and probably will evolve and change as the game is further developed.

Before the game starts, players on each team are sorted onto either the *field* or the *bench*. 4 players go onto the field and 4 stay on the bench. This depends on which players you set in the squad lineup.

The game is divided into 70 ticks which represent a minute of match time. On each tick here is the approximate breakdown of events:
* Each player does an *applyEffects* update where modifiers to their stats are applied based on their skills
  * If the player is on the Bench, they gain energy at a rate relational to their endurance skill.  Higher the skill, the faster they regenerate energy.
  * If the player is on the field, they lose energy relational to their endurance skill. This is a smaller amount of energy lost in comparison to the higher amount gained for Benched players.
  * If the player's energy dips below 10, they will be automatically *rotated* out.  The simulation will pick a player on the bench with the *same role*.  So if player Tholme is rotated out, then another **Center** on the bench should rotate in to take his place.

* Each player does a *think* update where they consider the state of the game/players. In future this evaluation of the state of the game/players will be skewed based on a skill of the players and may not always reflect the reality.  For example, player Lister may perceive that player Fred is a terrible shooter, so may decide to try blocking his pass instead. However, player Lister's perception may not be correct, and player Fred may be a great shooter, and thus player Fred may decide to shoot. Currently, the players perceptions are 100% what the reality is, but in the future you can expect this to change.

* Each player does an *action* update where they decide what action to do.
  * First, the player checks the state of play on the pitch - it is either "before kickoff" or "play on".
    * If "before kickoff", then every player will try a "tackle ball" action.
    * During "play_on", player further evaluates if his team or the opposing team has the ball, or if he himself has the ball.
    * If player **is ballhandler**, then he checks if he can score first. If he thinks he can score he will decide to shoot. Whether or not he can score is based on proximity to the opponent's goal. Usually within 1 space from the goal, he will think he can score. **Important Note**: this does not mean that he *actually* shoots.  It means he *decides* to.  Whether or not he successfuly takes the shot happens later in the `challenge` phase for this tick.
      * If the ballhandler thinks he can't score, he then decides whether he should pass or run the ball. The determination is made based on his own throwing and passing skills compared to his toughness and blocking skill.  Most of the time, he will prefer to do the thing he is stronger in. A pass is to a random person on the ballhandler's team.
    * If player is **not the ballhandler** and he is the **same team**, then player does nothing currently (to change in future).
    * If player is **not the ballhandler** and he is on the **opposing team**, then player considers the ballhandler and guesses the ballhandlers next action - whether it will be a run type action or throw type action.  The determination is made based on the ballhandler's throwing + passing skills compared to his toughness + blocking skills. Stronger throwing + passing will result in the player thinking the ballhandler will do a throw type action (shot or pass).
      * If player thinks ballhandler will do a throw type, he then determines whether that throw will be a shot or a pass.   * If the ballhandler's throwing is greater than his passing, then he'll guess the ballhandler is about to shoot and will perform a tryBlockShot.
        * If the ballhandler's passing is greater than is throwing, then he'll guess the ballhandler is about to pass and will perform a tryBlockPass.
      * If the player thinks the ballhandler will do a run type action, he'll perform a tryTacklePlayer.

* Now the [Challenge](#actions-that-players-take) phase of the tick evaluates all selected player actions. Each of the Challenge action types are iterated over and evaluated (see the list of them below).  The ballhandler will only have picked one of the actions (tryRun, tryScore, tryPass) and the match events are generated based on what the ballhandler ultimately tries to do and whether or not he is successful.  Of the opposing team players that attempted to stifle the ballhandler's chosen action, one will generally be picked to be the "opposing player" (as described in each of those challenge actions detailed below).  This is the player that is reported in the Match Event in the play by play control.
  * For the opposition players that "picked wrong", or picked an action that the ballhandler did not end up doing, their failed guesses are not recorded in Match Events for the play by play (although are/can be in the MatchLogs; the verbose logs).
  * For the other team players of the ballhandlers, they pick nothing to do during the tick (this is on the roadmap to change).


## Playing the Game
### How to sign up as a player
1. Go to https://shockball2.herokuapp.com and **Login** and then click **Allow**
2. If you see your avatar image then your registration/signup completed without issue.  You are ready to roll!
> The *Character Read* permission from combine allows the shockball application to get your combine character’s basic info for creating your name, profile image, and unique id.

### How to train your player
Each day, you may select a training regimen for your player. Every 24 hours your selected player performs their training for permanent skill changes (some increment, some decrement) depending on which regimen you select. Players must choose to train every day to get the most growth!

1. Go to your **Me** page (the default when you visit the website)
2. Select the **Training Regimen** drop-down and pick your preference.
3. You should see a onscreen confirmation of your selection.

### What skills change when I train
Here are the skill changes:
```javascript
if (player.regimen.value === 'Wing') {
    player.blocking = decrement(player.blocking, .25, player.blockingCap)
    player.throwing = increment(player.throwing, .5, player.throwingCap)
    player.passing = increment(player.passing, 1.5, player.passingCap)
    player.endurance = increment(player.endurance, .25, player.enduranceCap)
    player.toughness = decrement(player.toughness, .25, player.toughnessCap)
    player.vision = increment(player.vision, .25, player.visionCap)
    player.morale = increment(player.morale, 1)
    player.energy = decrement(player.energy, 5)
    player.leadership = increment(player.leadership, .25)
  } else if (player.regimen.value === 'Guard') {
    player.blocking = increment(player.blocking, 1.5, player.blockingCap)
    player.throwing = increment(player.throwing, .5, player.throwingCap)
    player.passing = decrement(player.passing, .25, player.passingCap)
    player.endurance = decrement(player.endurance, .25, player.enduranceCap)
    player.toughness = increment(player.toughness, .25, player.toughnessCap)
    player.vision = increment(player.vision, .25, player.visionCap)
    player.morale = increment(player.morale, 1)
    player.energy = decrement(player.energy, 5)
    player.leadership = increment(player.leadership, .25)
  } else if (player.regimen.value === 'Center') {
    player.blocking = decrement(player.blocking, .25, player.blockingCap)
    player.throwing = increment(player.throwing, 1.5, player.throwingCap)
    player.passing = increment(player.passing, .0, player.passingCap)
    player.endurance = increment(player.endurance, .75, player.enduranceCap)
    player.toughness = decrement(player.toughness, .25, player.toughnessCap)
    player.vision = increment(player.vision, .25, player.visionCap)
    player.morale = increment(player.morale, 1)
    player.energy = decrement(player.energy, 5)
    player.leadership = increment(player.leadership, .25)
  } else if (player.regimen.value === 'General') {
    // all skills unchanged with a slow energy recovery
    player.blocking = increment(player.blocking, .0, player.blockingCap)
    player.throwing = increment(player.throwing, .0, player.throwingCap)
    player.passing = increment(player.passing, .0, player.passingCap)
    player.endurance = increment(player.endurance, .0, player.enduranceCap)
    player.toughness = increment(player.toughness, .0, player.toughnessCap)
    player.vision = increment(player.vision, .0, player.visionCap)
    player.morale = increment(player.morale, 1)
    player.energy = increment(player.energy, 2)
    player.leadership = increment(player.leadership, .25)
  } else if (player.regimen.value === 'Rest') {
    // all skills suffer .25 decay with a fast energy recovery
    player.blocking = decrement(player.blocking, .25, player.blockingCap)
    player.throwing = decrement(player.throwing, .25, player.throwingCap)
    player.passing = decrement(player.passing, .25, player.passingCap)
    player.endurance = decrement(player.endurance, .25, player.enduranceCap)
    player.toughness = decrement(player.toughness, .25, player.toughnessCap)
    player.vision = decrement(player.vision, .25, player.visionCap)
    player.morale = increment(player.morale, 5)
    player.energy = increment(player.energy, 30)
    player.leadership = increment(player.leadership, .25)
  } else {
    // no training selected
    // all skills suffer .25 decay with a slow energy recovery
    player.blocking = decrement(player.blocking, .25, player.blockingCap)
    player.throwing = decrement(player.throwing, .25, player.throwingCap)
    player.passing = decrement(player.passing, .25, player.passingCap)
    player.endurance = decrement(player.endurance, .25, player.enduranceCap)
    player.toughness = decrement(player.toughness, .25, player.toughnessCap)
    player.vision = decrement(player.vision, .25, player.visionCap)
    player.morale = decrement(player.morale, .25)
    player.energy = increment(player.energy, 2)
    player.leadership = decrement(player.leadership, 1)
  }
```

### How to accept or reject a contract
In order to play on a team, a player must have an **active** contract with that team. Contracts are issued by the team’s owner and are for a single Season. Players receive a 20% signing bonus in their GSL account.  At the end of the season, players may “cash out” their account on request (i.e they will be sent the credits via combine).

1. Go to your **Office** page.
2. Find any contract(s) for your player in the **My Contract Offers** section.
3. If there is action waiting on you (like approval/rejection), you will see an **Approve/Reject** button in the contract row.  Click that.
4. Review the details of the contract and click **Accept** or **Reject**.
5. You should see an onscreen confirmation of your action.

### How to create a Team
A Shockball team requires the following:
* **team logo image URL**: a URL to a square pixel (example 200px by 200px or 1000px by 1000px)
* **team name**: which should consist of a planet or system prefix and then team name (example: Corellia Captains, Stensen Browbeaters)
* **team owner**: name of person who'll own and manage the team lineup and contracts, who must also exist as a Player in the league (must have logged in at least once)
* (OPTIONAL) **team stadium image URL**: a URL to a square pixel image
* (OPTIONAL) **team stadium name**: name of the team's home venue stadium 

1. Send your request to the [Discord Alpha channel](https://discord.gg/gxWphVs) with all that info. A team will be created for you and a starting account balance of 70Mil. <sub>Remember, this is non-real credits for Alpha...</sub>

### How to offer a player a contract
Teams can offer contracts to human players and NPC players. All players have a minimum market value based on their skills, and this minimum purchase price is enforced when offering a contract. While a human can accept or reject an offered contract, an NPC player will always accept the offer.

1. Go to the **Transfer Market** page and view all the players in the league. Players without a team appear as **Free Agents**.
2. Click on a player's name to be taken to that player's details page.
3. Find the blue button labeled **Offer Contract** and click it.
4. In the form's **Enter Purchase Price** field, enter the offer amount in whole numbers (Market Value is minimum you can offer).
5. In the form's **Pick Season** field, leave defaulted (will be enhanced in future for multiple seasons).
6. Click **Send Contract**
7. You should see an onscreen confirmation of your action.

At this point you can visit your **Office** page and view the team contracts. The states of a contract are:
* Pending - waiting for the Player to accept or reject
* Accepted - Player accepted, now waiting on league admin to approve
* Rejected - Player rejected your offer (currently there is no editing an existing contract offer - simply create a new one to renegotiate)
* Active - Player is now a member of the team, can be used in the lineup, and can participate in team matches.
> NPC players will automatically 'Accept' offered contracts, but they still require league admin approval before the contract is made active.

### How to set your team lineup
Teams can set a primary and secondary player for each position on the field (Left/Right Wing, Center, Guard). As a player gets tired, he/she will be rotated automatically with the other player in his role.

Example: Fipp is the primary Guard so he starts the game. When his energy gets below 20 percent, he is rotated out for Levon, the secondary Guard. And so forth.
> Currently players do not "swap" into other roles. Also, the substitute roles are not yet functional.

1. Go to the **Squad** page.
2. Scroll down to the game pitch map with images of players. Here you can click on a position and be presented with a list of your team players to select who should be in that position.
3. You should see a onscreen confirmation of your selection.

> *Bots* are there right now if your team is under the threshold of minimum players. In future, a minimum lineup (humans and/or npcs) will be required or forfeit the match. Bots are generated when the match is simulated, are random (with limits) in skill, and are destroyed after the match is over.

## Common Problems and Questions

### The offer contract button is not there
If the player is already a member of a team, then the option to offer them a contract will not be visible.

### Cannot spend more than available team budget
Your team has an account, and two representations of it on the **Office** page. You will find a **Potential Budget** and a **Available Budget**. The Potential Budget is the total amount of credits in your account at this moment. The Available Budget is the amount available minus any pending commitments (like contract offers that are pending). You receive this particular error when you are trying to offer a contract and the amount required will put you over what is left in your Available Budget.

### Must offer player a minimum bid equal to their market value
When offering a contract to a player, you are required to offer **at least the market value**. You may always offer more.

### How are player ratings determined
* Overall Rating = blocking + endurance + leadership + passing + throwing + toughness + vision / 7
* Wing Rating = throwing + passing + endurance + leadership / 4
* Center Rating = throwing + vision + endurance + leadership / 4
* Guard Rating = throwing + blocking + toughness + leadership / 4

### How contracts work
The Team Owner can find a list of players in the Transfer Market, click on their names, and from their profile page offer them a contract (via a Button on the profile page). They must pick an offer amount that is **at least** the minimum market value. The minimum market value is determined based on the players skill averages, and it updates each time they train.

When this offer is made, the amount for the contract is deducted from the **availableBalance** to prevent over-bidding beyond the team's budget.

The player may go to his/her **Office** page and approve or reject a contract offer. During this time both the Team Owner and Player will see the contract offer pending in the **Office** page.  An Accept will change the contract status to **awaiting admin** which requires a league admin to do the final "Approval" of the contract before it becomes **active**.  Once the contract is **active**, the player is a member of the team.

If the contract is **Rejected** the funds should be refunded to the team's **availableBudget**
### How team lineups work
The shockball game is two sides of 4 on the field at a given time.  Each side has a Center, a Right Wing, a Guard, and a Left Wing. Consider the shape of a Diamond - this is the arrangement of players on the screen in your **Squad** view.

Shockball players "rotate" in and out of the field based on their energy level (similar to a Hockey line change) - however, the players will change individually instead of as an entire line.  The energey threshold to change is `10`. When rotating, the player will only rotate with a player of same role. For example, a Left Wing will rotate with another Left Wing player and NOT a Center player. For this reason, in the lineup you see "stacks" of two for each role.  There is Center1 and below is Center2, then RightWing1 and RightWing2, then Guard1 and Guard2, then LeftWing1 and LeftWing2.  

All of the xx1 players will start the match for a Team's side ,and they will rotate out with the xx2 same role player when they `10` energy threshold is met.

#### More on energy
Players on the match field lose energy on every 'tick' of the match. A match has 70 'ticks' (or you can think of them as "Game Minutes"). The amount of energy a player loses is based on his/her Endurance stat.

Players on the "bench" (or rotated OUT) regain energy at a rate that is also based on their Endurance.

#### Subs
Currently subs do not participate in matches in any way. In future, as sub rules are implemented, they will have custom rules defined by the Team Owner that will be used to swap them in/out.

### What stats are important
First, the idea behind AI decision is based around this kind of flow:
* On every tick of the game, the Player takes in an understanding (or `perceivedWorldModel`) of the game.
* He/She uses this understanding to make decisions on what he/she will do:
  * If player is the ballhandler, player decides to take an action - the action could be a pass, a shot, or a run depending on the player's on stats. Currently, the player prefers to do the thing that he/she is strongest in.
  * If the player is not the ballhandler, the player decides to analyze what he/she thinks the ballhandler WILL do. The player makes this decision based on it's `perceivedWorldModel` - if not implemented this way at this very moment, it is stubbed out to be go this direction in future. If the Player believes the ballhandler will pass, the Player will attempt to do a blockPass action. If the player belives the ballhandler will try to shoot, the player attempts a blockShot action.

Now on every 'tick' of the game, all players analyze, think, and decide on a course of action - before success/fail is calculated. Then, after everyone on the field has decided, there is an evaluation of the `realWorldModel` against the `perceivedWorldModel` and the simulation calculates who wins out for each of their actions against the ballhandler. At the end of the 'tick' a single action will succeed and be written as a record. In other words, you won't have one Player succeed at a Tackle on playerZ and a different player succeed at a Score against an attempted block by playerY. Instead you would only have Player succeed at a Score against attempted block by playerY (**one event is the outcome**). The action follows the ballhandler.

Some positions prefer certain stats over others:
* Center: Throwing, Vision, Endurance
* Wing: Passing, Endurance, Throwing
* Guard: Blocking, Toughness, Throwing

#### Position of Ball
The position of the ball on the field is measured by linear proximity to the Home/Away goal. The home goal is at -5 and the away goal is at +5. Each team needs to carry the ball to within shooting range in order for a player to attempt a shot. The shooting range should also be based on their `perceivedWorldModel` - their own belief in their ability to score and thus choose to attempt a shot. Currently, the `perceivedWorldModel` is the same as reality, so Players will shoot if it is their stronger skill and they are within the range of about 1 from the opponent goal. 

>Note: this may not reflect the "reality" that they are too far away to shoot or were already within range a tick ago and didn't realize.

#### Actions that players take
Whether offense or defence, with/without ball, players can take different "actions" in the game, and on every given 'tick'. Here is a list and the stats that factor into win/loss of the action.

**Player analysis:**
* `analyzeCanScore` - scoring, if an opportunity arises, is every players number 1 priority. Currently this is based on where they are and how close the goal is. If it is within range for them and they think they can score, they will shoot.  If the player thinks he/she can't score, they will `analyzeCanPass`
* `analyzeCanPass` - thinks about player's won passing and throwing skills - if it is greater than own toughness + random roll then player will decide to pass, otherwise player will try to run with the ball.
* `analyzeNextAction` - player considers the ballhandler and decides if ballhanlder will try to throw or run. Player uses it's perception of if the ballhandler's throwing and passing is greater than the ballhandler's toughness plus a random roll.
* `analyzeMoreLikelyToShoot` - if player thinks ballhandler will 'throw', he/she now determines if the throw will be a shot or a pass. Based on player's perception of if the ballhandler's throwing is greater than their passing.

**Player challenges:**
* `resolveTackleBall` - here would be a struggle between whoever else on the field for control of the ball, or shot block, or pass interception/block, after winning the encounter, .possess is called.  For now the Toughness attribute determines who wins the Tackle.  For now we look at each player vying to tackle the ball, and add their Toughness and a random dice roll - highest wins.
* `resolvePlayerRun` - If ballhandler decides to run, for the time being he will always succeed unless a tackler(s) is trying to tackle him. If multiple tacklers are trying to tackle him, for time being one is chosen at random to be selected to actually challenge the ballhandler and then that tackler and the ballhandler compare their toughness + vision + random roll. Winner will prevail. For ballhandler that means he/she successfully runs, for opposing player that means he/she successfully tackles the player and gets the shockball.
* `resolvePlayerTryScore` - ballhandler is trying to score. If there are opposing players trying to block his/her shot then Guard has a higher probability to be the player selected to challenge the shot block, then Wing, then finally Center. Once selected, shooter compares shooting + vision + random roll against blocker's blocking + vision + random roll.
* `resolvePlayerTryPass` - same as `resolvePlayerTryScore` but ballhandler compares passing + vision + random roll to blocker's blocking + vision + random roll.
* `passForward` - player takes this final action after succeeding in the `resolvePlayerTryPass` and this will randomly select a fielded player on the same side to pass to. Ball moves forward one proximity position towards opposing goal.
* `runForward` - player carries ball forward one proximity position towards opposing goal.

### When scheduled server jobs happen
Things like the matches, player training, and contract cleanup are all server jobs and run at set schedules **once per day**:
* Train Players - This happens at **18:00 UTC** time (UTC is the same as GMT) - This is when all players in the league have their selected Training Regimens calculcated and their players are updated with ther new stats and new market value.
* Play Matches - This happens at **18:30 UTC** - This is when the fixtures (or matches) are simulated. The outcomes are saved back and you will see the fixture results in the app immediately after this finishes.
* Contract Cleanup - This happens at **4:30 UTC** - This is when handles switching active contracts to inactive (and vice versa) when a season changes. A season is a global configuration in the database that can be changed by administrators.
