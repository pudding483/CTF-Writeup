##### tags: `SwampCTF 2025`
# SwampCTF 2025 Web : Beginner Web

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

- ![image](https://hackmd.io/_uploads/rJ5kZ3UTJl.png)

## <span class="red">解題過程</span>

1. **首先進到題目後，頁面沒有太多訊息，一樣先查看網頁原始碼，發現 flag part1
flag part1：```w3b_```**

- ![image](https://hackmd.io/_uploads/SJ-q-3L61l.png)

&emsp;

2. **接著在最下面看到一個被註解的連結，嘗試在網路上搜尋看看，發現是真實存在的網站，所以應該要看一下 js 的部分，於是去 Sources 查看一下，在 main 找到了 flag2 和 flag3 相關的訊息**

- ![image](https://hackmd.io/_uploads/HJgQN3Iayg.png)
- ![image](https://hackmd.io/_uploads/SkRyIhUTyx.png)

&emsp;

3. **根據 code 的邏輯推斷，我應該要把 flag2 和 flag3 都用 AES 做 Decrypt ， 而 key 是 ```flagPart2_3``` ，但是：**
    
    (1) **這個 key 不是 16 bits**
    
    (2) **尚未得知 Initialization Vector**

&emsp;

4. **繼續找線索，在 delete() 前看到了這段 code ，有提供一個 github 的網址點進去查看一下。根據 PR 內容，當 ```sameSite property = Strict``` 時，我必須要讓 
```secure flag = false``` chrome 才會顯示 cookie** 

- ![image](https://hackmd.io/_uploads/B1Oas3L6kx.png)

&emsp;

5. **所以我在 Sources 底下的 ```main-....js``` 跑完 cookieService.set 的 function 後設了一個斷點 (一定要設斷點，這樣 browser 才能看懂 ```this.cookieService.set```) ，接著去 console 跑**
```javascript=
this.cookieService.set("flagPart2", $n.AES.decrypt(r, n).toString($n.enc.Utf8), {
expires: 7, path: "/", secure: false, sameSite: "Strict"
});
``` 
- **調整 cookie 的設定後，就能在 cookieService 看到 flag part2 了
flag part2：br0w53r5_4r3_**

- ![image](https://hackmd.io/_uploads/rkhGQ6UaJx.png)
- ![image](https://hackmd.io/_uploads/rkwS4aUTkg.png)

&emsp;

6. **到目前為止已經完成了 part1 和 part2 ，接下來完成 part3 就能獲得完整的 flag 。從程式碼可以看的出來， part3 和 header 有關，而且 part3 用了 fetch ，所以可以去 Network 的地方查看，進入 Network 後重新整理一下，找到 ```Type = fetch``` 點進去就能看到 flag part3
flag part3：c0mpl1c473d**

- ![image](https://hackmd.io/_uploads/rkYznpLakg.png)
- ![image](https://hackmd.io/_uploads/rJxkapIayl.png)
- ![image](https://hackmd.io/_uploads/BkQq2TUayx.png)

&emsp;

7. **最後把 flag part1 ~ part3 全部合起來，再加上題目給的 flag form 就會變成**
flag🚩：swampCTF{w3b_br0w53r5_4r3_c0mpl1c473d}