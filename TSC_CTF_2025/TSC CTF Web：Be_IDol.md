##### tags:`TSC_CTF_2025`

# TSC CTF Web：Be_IDol

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

- ![image](https://hackmd.io/_uploads/rkuZfYDDJe.png)
- ![image](https://hackmd.io/_uploads/ByJQfKwvJe.png)

&emsp;

## <span class="red">**解題過程 (After Event)**</span>

1. **直接先看網頁原始碼，可以從中看到有一個明顯的 ```ez_login()``` 讓我們藉由給定的 cookie 登入，這邊要注意的是這邊不能在 DevTools 的 console 直接 ```call ez_login``` ，原因是裡面有寫到 <span class="light_purple">location.reload</span> ，這行 code 會讓你不斷重新載入導致跳轉失敗，所以這邊我們就在 console 直接下 
document.cookie = "PHPSESSID=secretbackdoor123";
，並且在原登入畫面重新整理看看狀況**

- ![image](https://hackmd.io/_uploads/rJzTmYPw1x.png)

- ![image](https://hackmd.io/_uploads/rkE0XKwwye.png)

- ![image](https://hackmd.io/_uploads/rk91EKDwyg.png)

&emsp;

2. **進入到 File Explorer 後有很多檔案可以下載，於是我先隨便下載一個下來檢查內容，會發現內容是無法檢視，把檔案丟到 linux 環境用 string 指令檢查 PDF 檔也會說只是一個包含 ID 的PDF 檔案。並且我嘗試了不只一個檔案，其中包括最後修改日期為 ```2025-01-01``` 的所有 PDF ，都沒有能夠利用的資訊，<span class="red">根據目前已知資訊我猜測重點應該不是現在出現的 PDF 檔</span>**

- ![image](https://hackmd.io/_uploads/H1w0EYDvkg.png)

- ![image](https://hackmd.io/_uploads/BJyG8KvD1x.png)

&emsp;

3. **那該怎麼辦呢?事實上如果有出現可以下載的按鈕，我猜測會出現一些跟 download 有關的路徑，方法是把滑鼠放到 download 按鈕上就會發現了路徑相關資訊，有了路徑就有了一些想法
(這一步時我也有檢查過網頁原始碼，當初事先檢查網頁原始碼時發現了這個路徑，後續跟同學討論時才知道也可以這樣找)**

- ![image](https://hackmd.io/_uploads/SyeWmwKPvke.png)

&emsp;

4. **比賽時我解到這裡就沒有發現了，原因是：**

    (1) **把 ```/download.php?file_id=10000``` 接在 /index.php 後面，發現網頁看起來會不斷刷新這些檔案，並且檔案都是沒有辦法載入(挑選部分進行測試)**

    (2) **把 ```/download.php?file_id=10000 OR 1=1 --``` 接在 /index.php 後面也沒有任何發現，認為無法利用 URL 進行 SQL Injection**

    (3) **把 ```/download.php?file_name=flag.php``` 接在 /index.php 後面，看看能不能直接抓到 flag ，也不行**

    (4) **把 ```/download.php?file_id``` 設定為 0 , 1 , 9999 , 11001 都沒有發現**

&emsp;

5. **賽後我才知道，這裡還必須嘗試直接把 ```/index.php``` 改成 ```/download.php?file_id=11001``` 就會找到一個奇怪的頁面，然後接著找，直到 ```file_id=12001``` 時 ，就能夠找到 System Command Interface ，底下也有提示可以使用 shell 指令**

- ![image](https://hackmd.io/_uploads/HycECFvDJe.png)

- ![image](https://hackmd.io/_uploads/B1IF0KDvkg.png)

&emsp;

6. **檢查當前目錄和根目錄都沒有 flag 相關字眼後，直接使用 ```find / -name flag```暴力搜尋 flag**

- ![image](https://hackmd.io/_uploads/rkHFy5wDyg.png)
- ![image](https://hackmd.io/_uploads/HJ49ycvD1e.png)

&emsp;

7. **會找到只有在 /opt/flag 底下沒有被鎖權限，那就直接 ```cat /opt/flag``` ，系統說他是一個目錄，那就找找看他底下的內容，找到 flag.txt 直接 cat 就能得到 flag 了**

- ![image](https://hackmd.io/_uploads/ry5meqPv1e.png)
- ![image](https://hackmd.io/_uploads/HJP6g9PPJe.png)
- ![image](https://hackmd.io/_uploads/SJvHZ9PP1g.png)
- ![image](https://hackmd.io/_uploads/HyTO-cDP1x.png)

&emsp;

## <span class="red">**心得**</span>

1. **題梗：IDol → IDOR**

2. **在找的過程中，也可以自己寫 python 的 payload 輔助查詢，像是：**

    ```python=
    import os
    import requests

    # 設定起始與結束 ID 範圍
    start_id = 11001
    end_id = 20000

    success_logs = "./succ_logs"


    if not os.path.exists(success_log_file):
        with open(success_log_file, "w") as f:
            pass


    for file_id in range(start_id, end_id + 1):
        url = f"http://172.31.0.2:8057/download.php?file_id={file_id}"

        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()  # 確保回應狀態正常

            with open(success_log_file, "a") as log: # 紀錄 file_id
                log.write(f"ID: {file_id} is available\n")

        except requests.RequestException as e:
            pass
            #print(f"Failed to check file_id {file_id}: {e}")
    ```
**用這種方式能夠快速紀錄哪些 ID 可能是我們要的**

&emsp;

3. **額外進入 ```index.php``` 的方法，首先在一開始跳轉到文件管理介面的方法中，不使用他給的方案(也就是不能使用 ```ez_login()``` 中給予的 cookie) ，直接在 URL 後方加上 ```/index.php``` ，兩種方式最大的差別就是 cookie 中 的 PHPSESSID 會不一樣，然後後續步驟與解題過程相同，一樣能獲得 flag**

- ![image](https://hackmd.io/_uploads/S1HpjKwDke.png)

&emsp;

