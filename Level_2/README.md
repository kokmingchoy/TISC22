# Level 2

![Screenshot from 2022-08-28 11-48-16](https://user-images.githubusercontent.com/82754379/187056513-7e83cb89-e43e-48bb-addf-c2ec3f2bf092.png)

Besides the whitepaper document that was provided there was also a login screen provided at **chal00bq3ouweqtzva9xcobep6spl5m75fucey.ctf.sg:56765** :

![Screenshot from 2022-08-28 11-54-33](https://user-images.githubusercontent.com/82754379/187056934-28210410-30d6-4cd8-92fb-ddd4b7c73954.png)


The whitepaper states that the input to the server must be supplied as a string representation of a binary number of 8 bits (i.e. from "00000000" to "11111111").

The user/player gets to send 8 challenge inputs to the server (which responds accordingly after each challenge, based on the SECRET_KEY used in the challenge/response calculations), and subsequently the server gets to send 8 challenge values to the player, who must respond accordingly to prove he also knows the SECRET_KEY used in the calculations.

Since the SECRET_KEY is randomly-generated on each new session to the authentication server we could not use 2 live sessions to the server to play the responses of one session as the response inputs to the other session.

