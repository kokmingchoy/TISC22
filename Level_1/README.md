# Level 1 - Slay the Dragon

![Screenshot from 2022-08-27 00-10-06](https://user-images.githubusercontent.com/82754379/187013226-81ea2d70-dcbb-45e8-afbb-be6d3039315d.png)

![Screenshot from 2022-08-28 00-34-23](https://user-images.githubusercontent.com/82754379/187040042-df4617c2-3e28-4159-a4e4-e5cb5ba30f22.png)

The RPG game is a text-based client-server game with a "sword and sorcery" theme.
The basic premise is that you have to fight with a "boss" creature and reduce its HP (Hit Points) below 0 before it does the same to you.
Your HP starts at 10 points and may be restored midway through a boss fight by taking healing potions.
You can deal more damage if you purchase a sword.
Both Potions and Sword have to be purchased with Gold, which is obtained through mining.
Unfortunately, whenever mining is undertaken, there is a (20%) chance of dying through an encounter with a "creeper".

Fighting the first boss creature - Slime - is fairly trivial as it has a HP of 5 and if you had bought a Sword, you can finish it off in 2 rounds of attack.

Fighting the second boss creature - Wolf (HP 30 / Attack 3) - is a bit more challenging but still doable as long as you have enough Potions.

![Screenshot from 2022-08-28 00-35-57](https://user-images.githubusercontent.com/82754379/187040069-e3ff46bb-4841-497c-b81d-8c9cb67462ab.png)

The final boss creature - Dragon (HP 100 / Attack 50) - is impossible to beat through normal gameplay because after your first round of attack or use of Potion, its attack will instantly kill you (since you only have a maximum HP of 10).

<br>

## Exploring the Code

Since this was a client-server based game, it would be instructive to know what is being communicated over the network between the game client and the game server.

Looking at the source file **client/networking/netclient.py** I noted that there was a parameter "verbose" to the constructor function of class "NetClient" which is set to "False" by default. I changed this value to "True", ran the game, and learned the following:

- The client sends Commands to the server which denote the actions the player is taking, e.g. "ATTACK", "HEAL", "BATTLE", "RUN". The list of Commands are enumerated in the source file **core/models/command.py** .
- Whenever the client sends a "VALIDATE" command, the server will respond with JSON like the following:
```json
{"hp": 10, "max_hp": 10, "gold": 5, "sword": 0, "potion": 0}
```
- All values transmitted between the client and server are Base64-encoded

<br>

## Hits and Misses

The game server keeps track of the internal state of the player and boss statistics (i.e. the HP and attack points) and I could not find a way to substantially increase my max HP or attack.

I did find a way to easily mine an unlimited amount of Gold (which can later be used to purchase as many Potions as I want), by modifying the source file **client/event/workevent.py** as follows:
- Taking out the check for the random encounter with the "creeper", which would lead to instant death
- Taking out the screen display that delays the completion of the act of mining for Gold

I then focused on how the game server determines if the player had succeeded in killing the boss creature.
I found in **server/service/battleservice.py** that the game server has a record of all the Commands issued during a boss fight and acts on those Commands in sequence to derive the internal state of the player and boss statistics.

What was interesting was that in the function **__compute_battle_outcome()**, the player's attack on the boss is calculated first _before_ the boss' attack on the player is calculated. This means that as long as I could reduce the boss' HP to zero **in a single attack**, it does not matter that my max HP is 10 and the final boss' attack was 50, it would still be considered a win for the player.

The key to that is that the game server records the Commands the client sends to it using the function **self.history.log_commands_from_str()** which allows _multiple_ Commands to be recorded at the same time.

Exploiting this, instead of just sending _one_ ATTACK command per round of attack, which at most deals 3 points of damage to the boss (if I had a Sword), I could deal as much damage as I liked by sending _multiple_ ATTACK commands per round of attack.

The following were the changes needed to be made to the source files on the client side:

**core/models/command.py**
```python
class Command(enum):
    ATTACK = "ATTACK"
    BIG_ATTACK = "ATTACK " * 100    # New Command added
    BATTLE = "BATTLE"
    VIEW_STATS = "VIEW_STATS"
    HEAL = "HEAL"
    BOSS_ATTACK = "BOSS_ATTACK"
    RUN = "RUN"
    VALIDATE = "VALIDATE"
    BUY_SWORD = "BUY_SWORD"
    BUY_POTION = "BUY_POTION"
    BACK = "BACK"
    WORK = "WORK"
    EXIT = "EXIT"
```

**client/event/battleevent.py**
```python
    def run(self):
        self.boss: Boss = self.client.fetch_next_boss()

        while True:
            self.__display()

            match self.__get_battle_command():
                case Command.ATTACK:
                    self.__attack_boss()
                    break   # Added this break and commented out the original 2 lines below
                    # if self.boss.is_dead:
                    #     break
                    
                    # ... omitted for brevity ...

```

**client/event/battleevent.py**
```python
    def __attack_boss(self):
        ### Original line below commented out
        ### self.client.send_command(Command.ATTACK)
        
        ### Send multiple ATTACK commands
        self.client.send_command(Command.BIG_ATTACK)
        self.boss.receive_attack_from(self.player)
```

Running the game with the above changes, I could dispatch each of the 3 boss creatures with a single round of attack and win the game.

![Screenshot from 2022-08-28 00-24-08](https://user-images.githubusercontent.com/82754379/187042261-3b4bcf1c-26c7-4610-b399-a57054a35298.png)

<br>

:triangular_flag_on_post: **Level 1 flag: `TISC{L3T5_M33T_4G41N_1N_500_Y34R5_96eef57b46a6db572c08eef5f1924bc3}`**
