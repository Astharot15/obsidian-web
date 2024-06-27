```shell-session
Astharot15@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator
Astharot15@htb[/htb]$ cd Bashfuscator
Astharot15@htb[/htb]$ pip3 install setuptools==65
Astharot15@htb[/htb]$ python3 setup.py install --user
```


```shell-session
./bashfuscator -c 'cat /etc/passwd'
```

Ofusca el código que se le introduce, en este caso cat /etc/passwd. Para poder especificar flags.

```shell-session
Astharot15@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1

[+] Mutators used: Token/ForCode
[+] Payload:
eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"
[+] Payload size: 104 characters
```
