##### tags:`TSC_CTF_2025`

# TSC CTF Web：Ave Mujica

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

- ![image](https://hackmd.io/_uploads/r1vgKMMDkl.png)
- ![image](https://hackmd.io/_uploads/HJHQFzzPJl.png)
- ![image](https://hackmd.io/_uploads/r1WdtzzP1l.png)
- ![image](https://hackmd.io/_uploads/H1zYKMGDJg.png)

&emsp;

## <span class="red">**解題過程 (After Event)**</span>

1. **查看網頁原始碼和網頁後會發現沒有太多能夠互動的地方，有機會進行攻擊的地方看起來是 ```/image?name=<file_name>.webq``` ，當時我在嘗試像是：**

    (1) **```/image?name=flag.webq```**

    (2) **```/image?name=Oblivionis_Original.webp```**

    (3) **```/../../../flag.txt```**

**等等的 path traversal 後都沒有結果後，我就沒再繼續嘗試了**

&emsp;

2. **而這題首先可以利用 path traversal 成功下載到 /etc/passwd 的內容(有些 server 如果你輸入的目錄結構超出上限，它就會視為根目錄)，代表題目確實存在<span class="red">任意檔案下載漏洞</span>**

- ![image](https://hackmd.io/_uploads/HJJuK5vvJg.png)
- ![image](https://hackmd.io/_uploads/B19Yo9wPJl.png)

&emsp;

3. **這時就要利用 /proc 這個目錄了，這個目錄主要<span class="blue">負責處理程序和執行緒的相關資訊</span> ，在 /proc/self/cmdline 中能夠獲取<span class="red">當前</span> process 的啟動參數，所以我們下載 /proc/self/cmdline 看一下內容**

- ![image](https://hackmd.io/_uploads/HyxtTqPDye.png)

&emsp;

4. **用 VS code 打開它會說 unsupported text encoding ，用線上 16 進制編輯器打開。可以看到題目會運行 bangdream 這個 python 檔 (從 bangdream:app 和 python 3.12 看出來)**

- ![image](https://hackmd.io/_uploads/ByJQ0cDvyg.png)
- ![image](https://hackmd.io/_uploads/H1ja-jPPyg.png)

&emsp;

5. **那接著就把 ```bangdream.py``` 下載下來(注意路徑是 ```/../bangdream.py```) ，打開後會發現有個 route 能夠直接拿到 flag ，那就直接把那條 path 放進 URL ，成功獲取 flag**

- ![image](https://hackmd.io/_uploads/r1BuesPDkx.png)
- ![image](https://hackmd.io/_uploads/HkGU7jvvyl.png)
- ![image](https://hackmd.io/_uploads/ryEjXjvDyx.png)

&emsp;

## <span class="red">**心得**</span>

1. **看了一下別人的 WriteUp ，也能夠使用 ```curl 'http://172.31.3.2:168/image?name=../../../../proc/self/environ' --output flag.txt```
 直接一刀殺進去，把結果丟到 flag.txt**
     - **```/proc/self/environ```：用來獲取當前 process 的環境變數**