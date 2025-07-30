##### tags:`picoCTF_2025`

# picoCTF Web : n0s4n1ty 1

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

- ![image](https://hackmd.io/_uploads/B1-PrTCoke.png)
- ![image](https://hackmd.io/_uploads/SkBDraRikl.png)

## <span class="red">**解題過程**</span>

1. **根據題目，猜測可能是要上傳 php 檔進行 command injection ，所以選擇預先設計好的 payload 上傳，並點擊 Upload Profile 的按鈕**

- ![image](https://hackmd.io/_uploads/rkz3wa0oJg.png)
- ![image](https://hackmd.io/_uploads/HkuquTCiJe.png)

&emsp;

2. **上傳後就能夠直接使用像是 ```ls -al``` 查看一下 webshell 的檔案內容，會發現多了一個 echo 。修改一下 payload 並重新上傳，**

- ![image](https://hackmd.io/_uploads/ryt8-00s1g.png)
- ![image](https://hackmd.io/_uploads/ryHpZRAike.png)
- ![image](https://hackmd.io/_uploads/BkkefAAsJx.png)

&emsp;

3. **接著修改一下 shell command ，像是 ```ls /```**

- ![image](https://hackmd.io/_uploads/rJKoX0Rokx.png)

&emsp;

4. **題目說藏在 root 底下，所以 ```ls /root``` ，但會發現全部都是空白的。而這部分是因為沒有使用超級使用者(我是因為在 ```ls -al``` 時看到讀寫權限都是 root 才聯想到的)，加上超級使用者就能看到 flag.txt ，接著 cat 出來就行了**

- ![image](https://hackmd.io/_uploads/rykQcCCike.png)
- ![image](https://hackmd.io/_uploads/ByL6KCRoyg.png)
- ![image](https://hackmd.io/_uploads/BJmMtRRs1g.png)
