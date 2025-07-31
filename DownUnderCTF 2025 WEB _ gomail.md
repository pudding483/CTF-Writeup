##### tags: `DownUnderCTF_2025`
# DownUnderCTF 2025 WEB : gomail

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

- ![image](https://hackmd.io/_uploads/S1Ar6O5Ile.png)

## <span class="red">解題過程 (After Event)</span>

1. **根據題目看起來這題是要拿到 `MC Fat Monke` 的 mail (裡面有 flag) ， Instance 看起來沒有布置任何的 html ，所以直接進行 Code Review 。本題結構如下，是用 golang 寫的**

- ![image](https://hackmd.io/_uploads/ByS2CUj8eg.png)

- **本題架構：**
    ```text=
    gomail
      └── app
           ├── session
           │      ├── claims_test.go
           │      ├── claims.go
           │      ├── session_test.go
           │      └── session.go
           ├── go.mod
           ├── go.sum
           ├── handlers.go
           ├── main.go
           ├── middleware.go
           └── util.go
    ```

&emsp;

2. **本身對 golang 並不了解，但可以從 `main.go` 看出要互動的路由端點 (Endpoint) 主要有兩個，分別是 `/login` 和 `/emails` 。詳細 code 的內容會跳過，重點放在 exploit 的 code section**

- ![image](https://hackmd.io/_uploads/BkyoXDs8el.png)

&emsp;

3. **首先 `LoginHandler` 在發送 `POST request` 後會執行，內容如下。大意上就是會將拿到的 JSON 讀成寫好的 `SessionClaims` 結構 (`BindJSON()` 會自動從 `Content-Type: application/json` 的 body 解析 JSON) ，利用 `ConstantTimeCompare()` 拿輸入的密碼和隨機產生的 `64bytes(=512bits)` 做比較(如下圖)，避免因為執行時間差異 `(side-channel attack)` 而洩漏密碼**

- ![image](https://hackmd.io/_uploads/BJ7XP7pLll.png)
    
- ![image](https://hackmd.io/_uploads/S1e2p_viLex.png)

&emsp;

4. **接續 step3 ，比對完之後如果密碼正確就會將 `isAdmin` 設定為 true ，反之則將 `request` 中 `Email` 這個欄位的 value 強行轉換為 `guestEmail` 。在這之後就會包裝一個 token 並放在 `response` 內，用來做之後 `GET /emails` 時的身分證明，到這裡 `r.POST("/login", LoginHandler)` 的行為就做完了**

    ```go=
    func LoginHandler(c *gin.Context) {
        ...

        if usrPass != nil {
            if subtle.ConstantTimeCompare([]byte(lr.Password), usrPass) == 1 {
                isAdmin = true
            } else {
                lr.Email = guestEmail
            }
        }
        sH, exists := c.Get("sessionHandler")
        if !exists {
            c.JSON(http.StatusInternalServerError, gin.H{
                "error": "could not get session handler",
            })
            return
        }

        token, err := sH.(session.Session).Encode(lr.Email, isAdmin)

        if err != nil {
            c.JSON(http.StatusBadRequest, gin.H{
                "error": err,
            })
            return
        }
        c.JSON(http.StatusOK, gin.H{
            "token": token,
        })
    }
    ```

&emsp;

5. **接下來簡單說明 `r.GET("/emails", SessionMiddleware(), GetEmailsHandler)` 做了什麼。首先，在 `GET()` 中會發現他多放了一個 `SessionMiddleware()` 這個參數，而這是 `middleware` 中定義的一個針對請求的預處理，包括請求的 header 中有沒有 `X-Auth-Token` (也就是剛才的 token) ， session 欄位中的比對有沒有正確 ( `session.go` 中有定義 `.Decode()` 的檢查方法)**

- ![image](https://hackmd.io/_uploads/H1AwuXaUeg.png)

&emsp;

6. **step5 預處理完之後接著就會送到 `GetEmailHandler` 進行處理，這裡內容相對簡單一點，會分別檢查 `sessionClaims` (內含 `Email, Expiry, isAdmin`) 以及 `isAdmin` 的值，根據這個值回傳對應身分的信件。這些是大概的流程，實作細節(像是檢查方法之類的)會在介紹 pwned script 時一起解釋**

- ![image](https://hackmd.io/_uploads/Skjrm9i8gg.png)

&emsp;

7. **先講一下比賽當時我做過哪些調查，當時丟網路上查相關攻擊技巧時，看到有人寫了有關於<span class="red">用字串比對 `(==)` 取代恆定時間比較 `(crypto/subtle.ConstantTimeCompare)`</span> 造成的<span class="red">旁通道攻擊 `(CVE-2023-32691)`</span> ，當時一直再找相關漏洞利用結果回頭一看才發現，作者用的是對的!!好吧，但是又學到一招， try harder**

- ![image](https://hackmd.io/_uploads/BJxYS3oo8lg.png)

&emsp;

8. **接著我又有一計，當我看到 token 是由 `sessionClaims` 和 `sign (簽章)` 組成，所以當時我就想要<span class="red">嘗試偽造 token</span> ，我只改 `sessionClaims` ，讓 `Email` 和 `isAdmin` 欄位都變成我要的值難道不行嗎？於是我就偽造了 `sessionClaims` 的部分，再加上原本的 sig 。結果試了半天都沒有成功，回去檢查 code 在 `SessionMiddleware()` 中用了 `.Decode` ，而這個 function 在 `session.go` 中有定義會檢查 `HMAC hash` 值有沒有相同，也就是說因為<span class="red">原本的 `sign` 與我修改過後經過 `HMAC hash` 的值不同</span>，所以最終導致報錯**

- ![image](https://hackmd.io/_uploads/r1omK6jUeg.png)

    ```python=
    url = "https://web-gomail-3f344244ceb2.2025.ductf.net/"

    s = requests.Session()


    """ Based on handlers.go
    type loginReq struct {
        Email    string `json:"email" binding:"required,email"`
        Password string `json:"password" binding:"required"`
    }
    """

    json_data = {
        "Email" : "mc-fat@monke.zip",
        "Password" : "123" # doesn't matter
    }

    login_resp = s.post(url = url + "/login", json = json_data)

    # print("Login response in JSON:\n")
    # print(login_resp.json())
    token = login_resp.json()["token"]
    print("\n[*] Token is :", token)


    # analyze token
    body_b64, sig_b64 = token.split(".")
    compressed = base64.urlsafe_b64decode(body_b64 + "==")
    print("\n[*] body with base64 :", body_b64)
    print("\n[*] signature with base64 :", sig_b64 )

    # unzip gzip
    buf = BytesIO(compressed)
    with gzip.GzipFile(fileobj=buf, mode='rb') as f:
        raw = f.read()

    # analyze structure, email (2 bytes) + email (content) + expiry (8 bytes) + isAdmin (1 Bytes)
    # H = uint16
    email_len = struct.unpack("<H", raw[:2])[0]
    email = raw[2:2 + email_len].decode()
    # Q = uint64
    expiry = struct.unpack("<Q", raw[2 + email_len:2 + email_len + 8])[0]
    isAdmin_byte = raw[-1:]

    print(f"\n[*] email = {email}")
    print(f"[*] expiry = {expiry}")
    print(f"[*] isAdmin_byte = {isAdmin_byte}")

    new_isAdmin = b"t"
    new_raw = raw[:-1] + new_isAdmin

    # compress gzip
    buf = BytesIO()
    with gzip.GzipFile(fileobj=buf, mode='wb') as f:
        f.write(new_raw)
    new_compressed = buf.getvalue()

    # base64 encode, use origin sig_b64
    new_token = base64.urlsafe_b64encode(new_compressed).rstrip(b"=").decode() /
                + "." + sig_b64

    
    resp = s.get(url + "/emails", headers = {
        "X-Auth-Token": new_token
    })
    print("[*] /emails response:")
    print(resp.text)
    ```

- ![image](https://hackmd.io/_uploads/BkG9gRoUxx.png)

- ![image](https://hackmd.io/_uploads/SJZ1WCjUll.png)


- ![image](https://hackmd.io/_uploads/BJi3vnsIex.png)

&emsp;

9. **那麼這題應該怎麼打呢？這題的關鍵在於 `claims.go` 中， `Email` 的值是如何儲存的。如圖，他會<span class="red">將傳入值的長度直接丟給</span> `WriteLength()` ，然後讓 `el` 根據 `uint16` 的傳入值開一個相同大小的 `byte array` 來存放資料。也就是說，這題的關鍵在於<span class="red">利用 `uint16` 的大小上限</span>來做到 overflow 直接覆寫 `Email` 的值，最後成功產出 admin 身分的 token**

- ![image](https://hackmd.io/_uploads/Bkv_u0sLxe.png)

- ![image](https://hackmd.io/_uploads/S1L8GeaLxx.png)
    - ![image](https://hackmd.io/_uploads/Sy4YGxT8ee.png)

&emsp;

10. **payload 具體要塞甚麼呢？首先總長度一定要大於 uint16 的大小，所以至少要 `65536` 起跳 (`uint16` 範圍是 `0 ~ 65535`) ，接著多塞入的 bytes 是會被 `session.go` 的 `.Encode` 中呼叫的 function 讀入並且存起來的。 payload 如下：**

- **如果塞入內容總共 65536 bytes ，在 `claims.go` 的 `WriteLength` 函式就會如下：**

    - **`el := uint16(65536)` = `el := uint16(1) (overflow)`**
    
- **也就是說，針對 email 的部份目標帳號 `mc-fat@monke.zip` 總共 16 bytes ，就一共需要注入 `65536 + 16 = 65552` bytes ，這樣子在 `Claim.go` 的 `Serialize` 中 Email 欄位就會是 16 bytes**
    
    - **另外針對 `expiry` 和 `isAdmin` ，下面的結構可以很明顯看出，當我設計了一個錯的 `EmailLen` 時，後續的 bytes 會接續被影響(代表也可以被覆蓋)。同時 `expiry` 的檢查方式如下，也就是說只要 `expiry` 比 `當前時間` 還要大的時間都可以通過檢查(不會檢查這個時間是否離現在太遠)**
        - **結構: `[EmailLen:2][Email][Expiry:8][IsAdmin:1]`**
        
        - **檢查時會經過 `LittleEndian (低到高)` ${\rightarrow}$ `uint64` ${\rightarrow}$ `int16` 最後才是與 `time.Now().Unix()` 比較**
        
        - ![image](https://hackmd.io/_uploads/H17YGNT8le.png)
        
        - **如果今天 `expiry` 欄位是 ```b`t` * 8``` ：**
            - **`b"tttttttt" = [0x74, 0x74, 0x74, 0x74, 0x74, 0x74, 0x74, 0x74]`**
            - ![image](https://hackmd.io/_uploads/HybeB-A8ex.png)
            - ![image](https://hackmd.io/_uploads/SkXkBZCIlg.png)


- ![image](https://hackmd.io/_uploads/SJarVQaIlx.png)

- ![image](https://hackmd.io/_uploads/SkahxWCUlg.png)

- **flag🚩 :`DUCTF{g0v3rFloW_2_mY_eM41L5!}`**

- ![image](https://hackmd.io/_uploads/Hklrp8j8eg.png)

