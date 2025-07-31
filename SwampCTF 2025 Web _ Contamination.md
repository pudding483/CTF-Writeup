##### tags: `SwampCTF 2025`
# SwampCTF 2025 Web : Contamination

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

- ![image](https://hackmd.io/_uploads/r17ENdLTJx.png)

## <span class="red">解題過程 (After Event)</span>

1. **進到 browser 後，會發現只有 Unauthorized ，網頁原始碼和 DevTools 的 Sources 也沒有內容可以查看。去檢查 source code ，關鍵應該是如何繞過 proxy ，也就是 ```server.rb``` 中的 ```if request.params['action'] == 'getInfo'``` 
這行的目的是過濾除了 ```action = getInfo``` 以外的其他請求，但我們想要拿到 flag 就必須要讓 ```backend.py``` 的 ```param = "getFlag"``` ，但是 param 又是根據```request.args.get('action')``` ，所以必須想辦法繞過 ```server.rb``` 的 if 判斷式**

- **```server.rb```**
- ![image](https://hackmd.io/_uploads/rJS4EtD6kg.png)
- **```backend.py```**
- ![image](https://hackmd.io/_uploads/ryvyEFva1x.png)

&emsp;

2. **這時就可以利用 ruby 和 python 在處理 multi Query String 的 feature 來繞過。如果 request URL 內，有 same key with different values 的情況出現：**

    (1) **在 rack(ruby) 中 => <span class="red">後面的 value 會蓋掉前面的 value</span>**
    
    (2) **在 flask(python) 中 => 只會<span class="red">讀取第一個 key 的 value</span>**
    
    - **也就是說，如果 query string 是 ```action=getFlag&action=getInfo``` ，
那麼在 proxy 中 「action 這個 key」 的 value 就會被讀成 getInfo 而非 getFlag 。於是 payload 的 url 的部分寫成：**

    ```python=
    url = ".../api?action=getFlag&action=getInfo"
    
    body = 
    {
    ...
    }
    ```

- **也就是說，我們已經成功繞過 proxy 對於 action 的過濾，但是還有一個問題沒有解決：
<span class="red">後端對 json 的處理邏輯，並以此拿到 flag</span>**

&emsp;

3. **那就是 request body 的部分，我們的目標是讓 ```backend.py``` 觸發 ```request.get_json()``` 的讀取錯誤，從而導致 ```env_vars``` 的資料外洩(拿到 flag )。
但是 proxy 的部分也有對 request body (json) 解析做處理，<span class="red">如果 ```JSON.parse()``` 噴錯</span>，就會<span class="red">直接在 proxy 的處理階段 return error</span> 而不會呼叫 super(env) 將 json 資料傳遞到後端。
因此，目標是設計一個 json ，讓 proxy 能夠解析但是 backend 會噴錯**

- **```backend.py``` 重點**
- ![image](https://hackmd.io/_uploads/rJ-8iasakg.png)
- **```server.rb``` 重點**
- ![image](https://hackmd.io/_uploads/BJJOq6iaJx.png)

&emsp;

4. **所以 payload 就這樣誕生了。先講 ```JSON.parse()``` 對於 json 的處理， ```JSON.parse()``` 在處理非法轉義字符時，會把反斜線吃掉，最後變成 ```{"test": 1}```。然而，在 python 中，如果 ```request.get_json()``` 的內容包含非法轉義字元(像是 payload 中的 ```\s```) ，在解析時就會直接噴錯**

    ```python=
    url = ".../api?action=getFlag&action=getInfo"
    
    body = '{"te\st": 1}'
    ```

- **Ref：https://bishopfox.com/blog/json-interoperability-vulnerabilities**

&emsp;

5. **最後將 payload 寫成一個完整的 python script ，要小心 request.header 和 request.data 的設定
(把 payload 寫在 python 上沒處理好就會報錯，因為 payload(req.body) 就是用來觸發錯誤)**

- ![image](https://hackmd.io/_uploads/rJDuE_Qygg.png)


6. **最後趁題目的靶機還有開放的時候把靶機的 flag 打出來(writeup 的 script 有小改)**

- ![image](https://hackmd.io/_uploads/B143R3jp1g.png)
- **本地架設的靶機：**
- ![image](https://hackmd.io/_uploads/Skc0VOXyxl.png)
- flag🚩 : swampCTF{1nt3r0p3r4b1l1ty_p4r4m_p0llut10n_x7q9z3882e}

## <span class="red">另外想法</span>

1. **驗證了一下，用 request.params 的寫法也可以
(這邊用的 payload 是一開始打題目用的，實際只需要 "," 後面的部份就能 exploit 了)**

- ![image](https://hackmd.io/_uploads/HJu2GRop1l.png)
- ![image](https://hackmd.io/_uploads/ryDgXCs6yx.png)
