# Level 2

![Screenshot from 2022-08-28 11-48-16](https://user-images.githubusercontent.com/82754379/187056513-7e83cb89-e43e-48bb-addf-c2ec3f2bf092.png)

Besides the whitepaper document that was provided there was also a login screen provided at **chal00bq3ouweqtzva9xcobep6spl5m75fucey.ctf.sg:56765** :

![Screenshot from 2022-08-28 11-54-33](https://user-images.githubusercontent.com/82754379/187056934-28210410-30d6-4cd8-92fb-ddd4b7c73954.png)


Without having read through the whitepaper document first I simply played around with the login screen.

- Simply hitting 'enter' at the "Challenge Me #01" prompt threw an *AssertionError* :
```python
Challenge Me #01 <--
Traceback (most recent call last):
  File "/home/leaky/main.py", line 63, in <module>
    assert len(input_vec) == 8
AssertionError
```

- Looks like the input needs to be 8 characters in length. Giving a random input of 8 characters threw a different *AssertionError* :
```python
Challenge Me #01 <-- 12345678
Traceback (most recent call last):
  File "/home/leaky/main.py", line 64, in <module>
    assert input_vec.count("1") + input_vec.count("0") == 8
AssertionError
```

- Looks like the input needs to just be "1"s and "0"s (in other words, a string representation of a binary number between "0000000" to "11111111"):
```python
Challenge Me #01 <-- 10000000
My Response --> 11101111
Challenge Me #02 <--
```

The counter in the prompt suggests the way to get pass the authentication would be to supply a series of correct challenge inputs to a series of responses.
