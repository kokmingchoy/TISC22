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

## Exploring the Code

Since this was a client-server based game, it would be instructive to know what is being communicated over the network between the game client and the game server.

Looking at the source file _*client/networking/netclient.py*_ I noted that there was a parameter "verbose" to the constructor function of class "NetClient" which is set to "False" by default. I changed this value to "True", ran the game, and learned the following:

- The client sends Commands to the server which denote the actions the player is taking, e.g. "ATTACK", "HEAL", "BATTLE", "RUN". The list of Commands are enumerated in the source file _*core/models/command.py*_ .
- Whenever the client sends a "VALIDATE" command, the server will respond with JSON like the following:
```json
{"hp": 10, "max_hp": 10, "gold": 5, "sword": 0, "potion": 0}
```
- All values transmitted between the client and server are Base64-encoded

