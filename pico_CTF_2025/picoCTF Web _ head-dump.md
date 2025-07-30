##### tags:`picoCTF_2025`

# picoCTF Web : head-dump

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

- ![image](https://hackmd.io/_uploads/HJft25Ro1e.png)
- ![image](https://hackmd.io/_uploads/SkLp2cRi1l.png)

## <span class="red">**解題過程**</span>

1. **進入靶機後，點點看圖片, Likes 和 Comment 都沒有反應。根據題目所述，希望我們找到存有伺服器記憶體的 (log) file ，僅有 about us 和 Services 會重新導向到新的頁面**

- ![image](https://hackmd.io/_uploads/BJm3T9CiJg.png)
- ![image](https://hackmd.io/_uploads/HkOTT9CsJe.png)

&emsp;

2. **觀察後會發現， URL 的命名方式是依據功能來命名的且都是小寫。所以嘗試 exploit URL 看看。首先嘗試 /admin ，結果會出現如圖，無法 get /admin**

- ![image](https://hackmd.io/_uploads/rylNA5RiJl.png)

&emsp;

3. **這時我回到主頁重新檢查，發現了 #API Documentation 會連接到別的地方，所以點擊連結查看**

- ![image](https://hackmd.io/_uploads/HyrNeiCs1x.png)
- ![image](https://hackmd.io/_uploads/B138ej0oye.png)
- ![image](https://hackmd.io/_uploads/HyCwgi0o1x.png)

&emsp;

4. **成功找到目標 endpoint ，根據題目的名稱和目標 (memory) ，直接在根目錄底下接 heapdump ，就會出現一個 heapsnapshot**

- ![image](https://hackmd.io/_uploads/BJizboCokx.png)
- ![image](https://hackmd.io/_uploads/rkcubi0iyx.png)

&emsp;

5. **直接用 IDE 打開，我用的是 VScode ，會發現在第一列(橫的)有一個 node_types 叫做 hidden ，直接 ctrl+f 搜尋 hidden 看看**

- ![image](https://hackmd.io/_uploads/S1RNfo0i1e.png)
- ![image](https://hackmd.io/_uploads/rJ7RziAo1e.png)

&emsp;

6. **好像沒有有幫助的訊息，所以我搜尋了一下 heapdump 的分析工具，發現 chrome 的 DevTools 中有相關的工具可供使用，但是結果都是報錯。**

- **Document : https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots?hl=zh-tw**
- ![image](https://hackmd.io/_uploads/S1pxK2CiJl.png)

&emsp;

7. **所以我再一次使用 IDE 檢查 heapdump 的 file ，想到還沒有搜尋過 pico 這個關鍵詞，結果一搜尋就找到 flag 了**

- ![image](https://hackmd.io/_uploads/rJCIY20s1l.png)

&emsp;

## <span class="red">**額外心得**</span>

1. **我下載了 jq 去檢查 json 格式是否正確，發現在<span class="blue">第 276703 行</span>有出現錯誤，而這正好符合了 flag 所在的位置**

- ![image](https://hackmd.io/_uploads/B1aRt20okx.png)
- ![image](https://hackmd.io/_uploads/HyD353Cjkl.png)
