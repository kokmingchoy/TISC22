# Level 2

![Screenshot from 2022-08-28 11-48-16](https://user-images.githubusercontent.com/82754379/187056513-7e83cb89-e43e-48bb-addf-c2ec3f2bf092.png)

Besides the whitepaper document that was provided there was also a login screen provided at **chal00bq3ouweqtzva9xcobep6spl5m75fucey.ctf.sg:56765** :

![Screenshot from 2022-08-28 11-54-33](https://user-images.githubusercontent.com/82754379/187056934-28210410-30d6-4cd8-92fb-ddd4b7c73954.png)


The user/player gets to send 8 challenge inputs to the server (which responds accordingly after each challenge, based on the SECRET_KEY used in the challenge/response calculations), and subsequently the server gets to send 8 challenge values to the player, who must respond accordingly to prove he also knows the SECRET_KEY used in the calculations.

Since the SECRET_KEY is randomly-generated on each new session to the authentication server we could not use 2 live sessions to the server to play the responses of one session as the response inputs to the other session.

The supplied whitepaper document provides the source code implementation of the whole authentication (challenge/response) process.

In addition it shows the following:

![image](https://user-images.githubusercontent.com/82754379/187060586-1cb5cc98-cc96-41f0-a8f1-733e63fd25b9.png)


So, the SECRET_KEY may be represented by an 8-by-8 matrix of 1's and 0's.

The solution is surprisingly simple: Given the opportunity to provide 8 challenge inputs, we can provide as input an 8-by-8 Identity Matrix (where each column of 1's and 0's are the challenge inputs). The resultant responses will give us back the SECRET_KEY matrix (where each column of 1's and 0's is a response output from the server).

Once we have the SECRET_KEY in an 8-by-8 matrix, we can then use it to generate the appropriate response per challenge from the server, like so (running in Python REPL):

```python
>>> import numpy as np

>>> def strtovec(s, rows=8, cols=1):
...   return np.fromiter(list(s), dtype="int").reshape(rows, cols)

>>> SECRET_KEY = ([[0,0,0,0,1,1,1,0],
...   [1,0,1,0,0,1,0,1],
...   [0,0,0,1,1,0,1,0],
...   [0,0,1,0,1,1,1,1],
...   [0,1,0,1,0,0,0,1],
...   [1,0,1,0,1,0,1,0],
...   [1,0,0,1,0,1,0,0],
...   [0,1,1,1,1,1,0,0]])

>>> (SECRET_KEY @ strtovec("00111111")) & 1
array([[1],
       [1],
       [1],
       [1],
       [0],
       [1],
       [0],
       [0]])
>>> (SECRET_KEY @ strtovec("00001111")) & 1
array([[1],
       [0],
       [0],
       [0],
       [1],
       [0],
       [1],
       [0]])
>>> 

... and so on and so forth ...
```

Here's the complete transcript of a successful run to crack the authentication protocol:
```
=============
Challenge Me!
=============
Challenge Me #01 <-- 10000000
My Response --> 01000110
Challenge Me #02 <-- 01000000
My Response --> 00001001
Challenge Me #03 <-- 00100000
My Response --> 01010101
Challenge Me #04 <-- 00010000
My Response --> 00101011
Challenge Me #05 <-- 00001000
My Response --> 10110101
Challenge Me #06 <-- 00000100
My Response --> 11010011
Challenge Me #07 <-- 00000010
My Response --> 10110100
Challenge Me #08 <-- 00000001
My Response --> 01011000
==============
Challenge You!
==============
Challenge You #01 --> 00111111
Your Response <-- 11110100
Challenge You #02 --> 00001111
Your Response <-- 10001010
Challenge You #03 --> 11010000
Your Response <-- 01100100
Challenge You #04 --> 11000000
Your Response <-- 01001111
Challenge You #05 --> 00101001
Your Response <-- 10111000
Challenge You #06 --> 01100011
Your Response <-- 10110000
Challenge You #07 --> 00001001
Your Response <-- 11101101
Challenge You #08 --> 00010001
Your Response <-- 01110011
========================
All challenges passed :)
========================
=================================================================
Here is your flag: TISC{d0N7_R0lL_Ur_0wN_cRyp70_7a25ee4d777cc6e9}
=================================================================
```

<br>

:triangular_flag_on_post: **Level 2 flag: `TISC{d0N7_R0lL_Ur_0wN_cRyp70_7a25ee4d777cc6e9}`**
