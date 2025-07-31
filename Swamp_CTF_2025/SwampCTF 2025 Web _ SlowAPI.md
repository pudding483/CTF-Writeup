##### tags: `SwampCTF 2025`
# SwampCTF 2025 Web : SlowAPI

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

- ![image](https://hackmd.io/_uploads/S1sHJHBTJg.png)

## <span class="red">解題過程</span>

1. **首先會看到一個 API 的設定介面，右上角有一個 secret flag 點進去看看，會發現權限不足，但他給了一行提示：嘗試讓 API 相信我是 server-side middleware process**

- ![image](https://hackmd.io/_uploads/Bk6VNVL6kl.png)
- ![image](https://hackmd.io/_uploads/Hy95EVIpJl.png)

&emsp;

2. **先去登入介面看看，會發現功能還沒完善，加上剛剛的那行提示，猜測這題應該是跟 SSRF 有關，順便檢查一下網站原始碼，只有發現在設定一些物件**

- ![image](https://hackmd.io/_uploads/BJT-BVLpkl.png)
- ![image](https://hackmd.io/_uploads/rkV_SVLTkg.png)

&emsp;

3. **接著我用 burp 去檢查一下，發現了 host 與 IP 位址在取 flag 時的特徵，而且這題沒有明顯的 api 可以直接做操作，所以沒辦法改 URL 直接做操作**

- ![image](https://hackmd.io/_uploads/r1e8FVLT1x.png)

&emsp;

4. **根據題目一開始給的訊息，這題是用 ```Next.js``` 撰寫的，加上 step1 的提示，於是我去找 ```Next.js``` 有沒有這類型的漏洞資訊可以看，結果發現了 ```CVE-2025-29927```**

- **https://www.cnblogs.com/CVE-Lemon/p/18789173**

&emsp;

5. **利用 ```CVE-2025-29927``` 所提到的在 request 中添加 
```x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware``` 
就可以達到 SSRF 的目的，但是如果 ```Host = backend : 8000``` server 會讀不懂，所以必須將 Host 設定為根目錄(```chals.swampctf.com:43611```) 來送 request**

- ![image](https://hackmd.io/_uploads/ryZEluUpkg.png)

&emsp;

6. **成功獲得旗子**

- ![image](https://hackmd.io/_uploads/SkjtydI61l.png)
- flag🚩：swampCTF{b3w4r3_th3_m1ddl3w4r3}