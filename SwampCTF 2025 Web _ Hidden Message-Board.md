##### tags: `SwampCTF 2025`
# SwampCTF 2025 Web : Hidden Message-Board

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

- ![image](https://hackmd.io/_uploads/rJZ48Qrakx.png)

## <span class="red">解題過程</span>

1. **打開題目會發現一個叫做 HackerChat 的頁面，檢查一下網頁原始碼會發現有個 flagstuff ，並且在後面說明要將他移除**

- ![image](https://hackmd.io/_uploads/ryg3j8mr61l.png)
- ![image](https://hackmd.io/_uploads/SkZqvQBa1g.png)

&emsp;

2. **按照要求，我用 
```document.getElementById("flagstuff").remove();``` 
將 flagstuff 移除，能夠看到聊天室出現了剛剛看到的 code**

- ![image](https://hackmd.io/_uploads/rkaSnQrpJl.png)

&emsp;

3. **目前看起來經過剛剛的操作並沒有讓我獲得能夠利用的資訊，所以我去 sources 看看有沒有線索，找到了 ```app.js``` 以及一些獲得 flag 的觸發邏輯。根據程式碼我必須要把 ```printFlagSetup``` 的 ```code``` 屬性改成 ```G1v3M3Th3Fl@g!!!!``` ， flag 就會在聊天室顯示了**

- ![image](https://hackmd.io/_uploads/H1wt2mHpkg.png)
- ![image](https://hackmd.io/_uploads/BymyamSpJe.png)
- ![image](https://hackmd.io/_uploads/SyBo0Er6Jl.png)

&emsp;

4. **於是我直接在 Elements 將 code 賦值為 ```G1v3M3Th3Fl@g!!!!``` ，接著在聊天室隨便打個字就會跳出 flag 了**

- ![image](https://hackmd.io/_uploads/BJfy04H6ye.png)
- ![image](https://hackmd.io/_uploads/S1ZwR4Sake.png)
- flag🚩：swampCTF{Cr0ss_S1t3_Scr1pt1ng_0r_XSS_c4n_ch4ng3_w3bs1t3s}

## <span class="red">賽後查看他人解法</span>

1. **也可以直接在聊天室輸入：**
    ```html= 
    <img onError=document.getElementById("flagstuff").setAttribute("code","G1v3M3Th3Fl@g!!!!");
    src='any-invalid-url.com'>
    ```
    **這樣就會直接更改 flagstuff 的 Attribute 了**