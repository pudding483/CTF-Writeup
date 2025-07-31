##### tags: `DEFCON Qual CTF 2025`
# DEFCON Qual CTF Basics : 🐱‍💻🌐

<style>
.red {
  color: red;
}
.blue {
  color: blue;
}
.purple {
  color: #7D3382;
}
.light_purple {
  color: #810cf5;
}
</style>

<span class=""></span>

## <span class="red">題目</span>

```$ export INPUT="____ ___ _______"  # some string goes here```
```"$ ${@#IV}"  ''p''"${@,,}"rintf  %s  "$(  Ax='  '"'"'E'"'"'"V"A"L" "$( ${@,}  P"R"I'"'"''"'"'\NTF %S  '"'"'23- C- TUC | MUS5DM | tupni$ OHCE ;ENOD ;LLUN/VED/> MUS5DM | tupni$ OHCE ;I$ PEELS OD ;)0001 1 QES($ NI I ROF'"'"'  ${*##.}|${@%;}  RE${@}V${*%%O1}  ${@%\`}   )" '$*  &&${@/-_/\{}p$'\162i'${*##E}n${*%%B*}tf  %s  "${Ax~~}"  $@ ;  ${!*}   )"  "$@" | b"a"sh  ${*//c}```

- ![image](https://hackmd.io/_uploads/r1UgHrw0Je.png)

## <span class="red">解題方法</span>

1. **先解混淆，解混淆後長成這樣：**

```
$ INPUT="____ ___ _______"
$ for i in $(seq 1 1000); do sleep $i; echo $INPUT | md5sum >/dev/null; done; echo $INPUT | md5sum | cut -c -32
```

&emsp;

2. **可以進行簡化，簡化後長成這樣：**

```
$ INPUT="____ ___ _______"
$ echo $INPUT | md5sum | cut -c -32
```

&emsp;

3. **接著依據格式 (4, 3, 7) 通靈出 ```$ INPUT```**

- **```$ INPUT = Hack the planet!```**

- ![image](https://hackmd.io/_uploads/HyXGLHv0Jg.png)

- **flag🚩 : `flag{af6f4f9d82898e44628ef8740a707db2}`**