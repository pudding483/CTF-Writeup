##### tags: `HackTheBox-note`
# HTB Web : Trial by Fire (Very Easy)

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


## <span class="red">前言(可略)</span>

- As you ascend the treacherous slopes of the Flame Peaks, the scorching heat and shifting volcanic terrain test your endurance with every step. Rivers of molten lava carve fiery paths through the mountains, illuminating the night with an eerie crimson glow. The air is thick with ash, and the distant rumble of the earth warns of the danger that lies ahead. At the heart of this infernal landscape, a colossal Fire Drake awaits—a guardian of flame and fury, determined to judge those who dare trespass. With eyes like embers and scales hardened by centuries of heat, the Fire Drake does not attack blindly. Instead, it weaves illusions of fear, manifesting your deepest doubts and past failures. To reach the Emberstone, the legendary artifact hidden beyond its lair, you must prove your resilience, defying both the drake’s scorching onslaught and the mental trials it conjures. Stand firm, outwit its trickery, and strike with precision—only those with unyielding courage and strategic mastery will endure the Trial by Fire and claim their place among the legends of Eldoria.

- **當你攀登炙熱險峻的烈焰峰時，灼熱的高溫與不斷變化的火山地形在每一步都考驗著你的耐力。熔岩河流在群山間劃出炽熱的道路，將夜空映照成詭異的猩紅色。空氣中瀰漫著濃厚的灰燼，而大地深處傳來的低沉轟鳴，警示著前方潛伏的危險。**

- **在這片炙熱的地獄中心，一頭巨大的火焰巨龍鎮守於此——它是烈焰與憤怒的守護者，決心審判任何膽敢闖入者。它的雙眼如燃燒的餘燼，覆蓋全身的鱗片經歷了數百年的高溫鍛鍊，堅不可摧。然而，這頭火焰巨龍並不會盲目攻擊，它擅長編織恐懼的幻象，使你的內心深處最黑暗的懷疑與過往的失敗具現化。**

- **若想抵達隱藏於其巢穴深處的傳說寶物——「灰燼之石」，你必須展現堅韌不拔的意志，抵禦巨龍的灼熱攻擊與心靈試煉。堅守信念，識破其詭計，並精準出擊——唯有擁有無畏勇氣與卓越智慧之人，才能通過烈焰試煉，在 Eldoria 的傳說中留下不朽的名字。**

## <span class="red">題目(架構)</span>

- ![image](https://hackmd.io/_uploads/rkAEFeonJx.png)

## <span class="red">解題過程</span>

1. **題目給了一包檔案要我們自己在 local 建，所以直接跑寫好的 ```build-docker.sh``` 。跑完後應該會看到 docker desktop 有新的 images，接著下 ```docker run -d -p 1337:1337 --name web_challenge web_trial_by_fire``` 就能在 containers 看到靶機了。然後在 URL 搜尋 ```http://localhost:1337``` 應該就能看到題目了**

- ![image](https://hackmd.io/_uploads/Sk8_qei3kx.png)
- ![image](https://hackmd.io/_uploads/rk_6qeihJl.png)
- ![image](https://hackmd.io/_uploads/B1BHsgi2yg.png)

&emsp;

2. **題目敘述翻譯如下，照慣例打開 DevTools 看一下，並沒有太多線索，只有發現<span class="red">挑戰者名稱有輸入長度限制 30</span> ，以及 49 很關鍵**

    ```
    在燃燒的河流與炙熱的烈焰之地，火焰巨龍守護著「灰燼之石」。無數人曾渴望獲得它的力量，卻無一人成功。

    傳說中存在著古老的模板卷軸——神秘的文書，若被巧妙利用，便能扭轉命運。隱藏的符文或許能改變一切。

    你能解讀這些符文嗎？也許 49 才是關鍵。
    ```

- ![image](https://hackmd.io/_uploads/SyhM2es3ye.png)

&emsp;

3. **進到畫面中可以發現要 ~~勇者鬥惡龍了~~ ，右上角還可以打開精心設計的音樂。一樣打開 DevTools 檢查，發現有一堆按鈕才知道要縮放一下頁面，一下子就清楚怎麼玩了**

- ![image](https://hackmd.io/_uploads/SkGw6gjh1x.png)
- ![image](https://hackmd.io/_uploads/rJO4ClihJx.png)
- ![image](https://hackmd.io/_uploads/HkUZCls3yl.png)

&emsp;

4. **有鑑於勇者和惡龍之間的實力差距懸殊，所以我必須想(找)辦(漏)法(洞)，在檢查網頁原始碼時發現了一個隱藏按鈕，並於底下的程式碼中寫了該如何移除 hidden 這個屬性的邏輯。**

- ![image](https://hackmd.io/_uploads/B1Y6--s2ye.png)

&emsp;

5. **第一個 if 判斷式會先判斷現在按下的按鍵是不是 konamiCode array 中第一個值，是的話會將 index 加 1 (用於繼續判斷) ，如果 konamiIndex 值符合 array 長度(全部按鍵都是對的)，那就會將 ```.leet``` 的 hidden 屬性移除**

- ![image](https://hackmd.io/_uploads/H1gVEWj2kx.png)
- ![image](https://hackmd.io/_uploads/HyAbN-snJx.png)

&emsp;

6. **按照題目要求，按下鍵盤上的 PGUP (上鍵)就能成功出現捕捉按鈕了，按下後會跳出訊息，要我們以 ```{{ url_for.__globals__ }}``` 的方式獲得祕寶，搭配一開始提示的訊息( 49 很關鍵)，應該是要我們以 SSTI 的方式獲得秘密(因為通常會拿 ```{{7*7}}``` 試試看是不是可以 SSTI ，如果輸出 49 就有機會可以)**

- ![image](https://hackmd.io/_uploads/rkoiBZs21l.png)
- ![image](https://hackmd.io/_uploads/rkQyIWshye.png)

&emsp;

7. **那就直接打開 DevTools 的 console 進行 SSTI ，會發現很多類別都是未定義(像是 ```globals()``` 和 ```__builtins__``` 。於是我去翻一下題目給的 .py 檔， ```routes.py``` 中可以看到在 post 到 /begin 時的邏輯時，會將 ```warrior_name``` 丟到 template 進行渲染 (render)**

- ![image](https://hackmd.io/_uploads/B1XW7zs2Jx.png)
- ![image](https://hackmd.io/_uploads/rJmcPJ3nkl.png)

&emsp;

8. **所以這裡我想要試試看用 burp 送一個假的 post 看看能不能以此達到 SSTI 。找到 /begin 送的 request ，將其丟到 repeater，接著嘗試用 warrior_name 進行 SSTI ，嘗試看看剛剛跳出的提示 ```{{ url_for.__globals__ }}``` ，會發現 response 跟在網頁上看到的沒有差別**

- ![image](https://hackmd.io/_uploads/rJanO1n3yl.png)
- ![image](https://hackmd.io/_uploads/r1_Xjxhh1x.png)

&emsp;

9. **所以現在的目標是注入點，回想起之前的整個過程，有能夠讓伺服器接收到使用者輸入的地方只有一開始的地方(也就是提供 49 是關鍵那段話的頁面)。然後再想到會出現你的使用者名稱的地方。第一個是對戰過程中，第二個則是戰敗的結算畫面**

- ![image](https://hackmd.io/_uploads/ryHmpehnyx.png)
- ![image](https://hackmd.io/_uploads/B1oA2ehn1l.png)

&emsp;

10. **回到最初的起點，這次按照過關提示將名稱改為 ```{{url_for.__globals__}}``` ，重新跑一次剛剛的過程**

    (1) **首先是戰鬥畫面，並沒有照我預想的出現所有 ```url_for.__globals__``` 底下的類別**

    (2) **接著我故意落，果然在戰損頁面看到所有 ```url_for.__globals__``` 底下的類別**

- ![image](https://hackmd.io/_uploads/ryFOaxh31x.png)
- ![image](https://hackmd.io/_uploads/HklQ0lhnyg.png)
- ![image](https://hackmd.io/_uploads/H1jBkZ2nkl.png)

&emsp;

11. **在這之中，我有看到了像是 ```__import__``` 這個內建函式，所以應該可以利用一下這個函式來達到 RCE 。但是這裡需要注意在最一開始時，<span class="red">挑戰者名稱有輸入長度限制 30</span> (step2) ，所以不能從初始挑戰者名稱那邊進行 SSTI ，我的想法是利用一下在 code 中 session 的設計**

- ![image](https://hackmd.io/_uploads/SyZfeb3nyg.png)
- ![image](https://hackmd.io/_uploads/rkp-Ez3h1g.png)

&emsp;

12. **將整串 session 丟至 cyberchef 並用 base64 解雜湊，可以發現在前半段是用 warrior 來命名的(與 code 相符)。這裡要做的就是設計完 payload 後，用 base64 進行雜湊再加上原先後半部分的程式碼應該就能成功 RCE 了
這裡先把 payload 設定為 : 
```{{url_for.__globals__.__import__('os').popen('ls').read()}}```
然後用 base64 雜湊後再加上原本後半部分 session**

- ![image](https://hackmd.io/_uploads/SJsBEzn2Je.png)
- ![image](https://hackmd.io/_uploads/BJkYIGhhyg.png)
- ![image](https://hackmd.io/_uploads/HJzHwM3nJg.png)

&emsp;

13. **將 browser 的 session 設定為剛剛合併的結果，但是失敗了，根據 code 出現 Unknown Warrior 代表內部出錯了(出錯了會使用預設值 Unknown Warrior )**

- ![image](https://hackmd.io/_uploads/HkF2Dfn3yx.png)

&emsp;

14. **這邊改回用 burp ，思路是：**
    (1) **先用 burp 寄一次較長的 payload 繞過輸入限制獲得 session
(此步驟在 /begin 這邊寄送 request)**
    (2) **再利用獲得的 session 去存取 ```/battle-report```**

- ![image](https://hackmd.io/_uploads/r1vO4Qhn1l.png)
- ![image](https://hackmd.io/_uploads/rJ-9Nmn21x.png)

&emsp;

15. **還是失敗了，會發現只要送超過長度限制的 payload 最終都會導致 Internal Server Error 。但是經過不斷嘗試後我才想到，如果不能對 warrior_name 下手(會影響 session) ，但我還是可以對 Battle Statistics 的數值這邊下手阿！ 重新命名 warrior_name (方便等一下觀察)並使用 burp 來寄送 payload 試試看**

- ![image](https://hackmd.io/_uploads/HyrzLQan1e.png)

&emsp;

16. **所以我先把 damage_result 的值更改為 ```{{url_for.__globals__}}``` 這段 payload 先試試看能不能得到 ```url_for.__globals__``` 底下的所有類別，結果是可以的。但我發現確定不能使用
```{{url_for.__globals__.__import__('os').popen}}```
這樣的 payload**

- ![image](https://hackmd.io/_uploads/ryAtvXpnJe.png)
- ![image](https://hackmd.io/_uploads/B1j1Ymahkx.png)

&emsp;

17. **最終透過題目給的 ```config.py``` 檔中引入了 os 給了我靈感，我從 config 底下成功 import os ，並找到了 ```flag.txt```
```{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}```**

- ![image](https://hackmd.io/_uploads/SJ-WeHThyg.png)
- ![image](https://hackmd.io/_uploads/ryksb4a2Je.png)

&emsp;

18. **最終獲取 flag 的 payload 是
```{{config.__class__.__init__.__globals__['os'].popen('cat%20f*').read()}}```
(因為 burp 無法直接辨識空格，所以要用 url encode 的結果來代替)
以及成功在題目機上打出 flag 的截圖**

- ![image](https://hackmd.io/_uploads/BkFvGEpn1l.png)
- ![image](https://hackmd.io/_uploads/r1rMENT21l.png)

&emsp;

## <span class="red">心得</span>

1. **這麼看來，回到 step14 ，這個方法或許有機會成功(有可能這裡出錯是因為我的 payload 不對)，為了驗證一下，將成功的 payload 按照 step14 的步驟做一遍。而事實證明這個方法是可行的**

    (1) **首先是將 warrior_name 改為 payload :** **```{{config.__class__.__init__.__globals__['os'].popen('cat%20f*').read()}}```**

    (2) **再利用獲得的 session 去存取 ```/battle-report```**

- ![image](https://hackmd.io/_uploads/HJtvuEp3kg.png)
- ![image](https://hackmd.io/_uploads/Bk8ddNa3kg.png)
- ![image](https://hackmd.io/_uploads/SJWR_NThkg.png)

&emsp;

2. **基於實驗的精神，我也把 step12 的這個方法用正確的 payload 再做一次試試看可不可行。這次就真的不能這樣做了，結論是 cookie 後半段帶的值應該是有經過處理，不會是一樣的**

    (1) **payload 是** **```{{config.__class__.__init__.__globals__['os'].popen('cat%20f*').read()}}```**
    
    (2) **然後合併並更換 session 的值，同時將網頁重新整理**

- ![image](https://hackmd.io/_uploads/rJCosEp21g.png)
- ![image](https://hackmd.io/_uploads/HJYE3Ep21g.png)
- ![image](https://hackmd.io/_uploads/BJU0oNp3kx.png)
- ![image](https://hackmd.io/_uploads/BJXxn4ahJg.png)
- ![image](https://hackmd.io/_uploads/r1EZ2E62yl.png)
