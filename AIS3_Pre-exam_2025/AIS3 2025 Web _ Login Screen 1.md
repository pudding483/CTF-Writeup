##### tags: `AIS3_Pre-exam_2025`
# AIS3 2025 Web : Login Screen 1

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

- ![image](https://hackmd.io/_uploads/SynBbpCWll.png)

## <span class="red">解題過程</span>

1. **進到網頁後會看到一個登入頁面，照著提示中的小字繼續登入，就會看到 2FA 的頁面，繼續按照提示就會看到需要 admin (權限) 才能觀察**

- ![image](https://hackmd.io/_uploads/Hy2ASpCZle.png)
- ![image](https://hackmd.io/_uploads/SyrCYJJfxe.png)
- ![image](https://hackmd.io/_uploads/HyVU9kyGxe.png)

&emsp;

2. **所以我就試試看直接用**
    ```
    Username = admin
    Password = admin
    ``` 
    **能不能登入，想法是用 burp 再次觀察一下整個過程的流量，結果就在我使用 admin/admin 登入後，由於一直卡在 admin 帳號的 2FA 驗證階段所以我按上一頁時，就在 burp 發現了成功以 admin 身分進入後瀏覽到 flag 的 response**

- ![image](https://hackmd.io/_uploads/HyF9UTRZge.png)

- flag🚩 : `AIS3{1.Es55y_SQL_1nJ3ct10n_w1th_2fa_IuABDADGeP0}`

## <span class="red">正解應該是？</span>

1. **利用 union select 的 SQL Injection 技巧，payload = 
`' UNION SELECT id, username, password, code FROM Users WHERE username = 'admin' --` 這樣就能以 admin 身分成功登入**

- ![image](https://hackmd.io/_uploads/Sk1on8Jzxx.png)

