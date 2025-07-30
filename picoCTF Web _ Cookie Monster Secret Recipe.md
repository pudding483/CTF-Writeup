##### tags:`picoCTF_2025`

# picoCTF Web : Cookie Monster Secret Recipe

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

- ![image](https://hackmd.io/_uploads/rJ_VQcCokx.png)
- ![image](https://hackmd.io/_uploads/SJluuqCsyg.png)

## <span class="red">**解題過程**</span>

1. **根據題目所述， flag 與 cookie 有關聯，所以使用 DevTools 檢查一下網頁的 cookie ，會發現什麼都沒有**

- ![image](https://hackmd.io/_uploads/Hy8ZY90oke.png)

&emsp;

2. **所以這時候我就去看了一下網頁原始碼，發現題目使用的是 login.php 的檔案，所以我在 URL 重新導向一次 login.php 再去檢查 cookie 時就能夠發現有 cookie 了**

- ![image](https://hackmd.io/_uploads/rJndq50s1e.png)
- ![image](https://hackmd.io/_uploads/B1kocqCs1e.png)

&emsp;

3. **得到 cookie 後會發現 cookie 怪怪的，結尾是 ```%3D%3D``` ， ```%3D%3D``` 在經過 URL decode 之後是 ```==``` ，於是我猜測 cookie 可能是經過 base64 雜湊後的結果**

- ![image](https://hackmd.io/_uploads/ByOrjqAsJg.png)

&emsp;

4. **驗證一下，將整串 cookie 丟到 cyberchef 先經過 URL decode 再經過 base64 解雜湊就能獲得 flag 了**

- ![image](https://hackmd.io/_uploads/ry71ncCjyx.png)

