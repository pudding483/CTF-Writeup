##### tags: `HTB Apocalypse CTF 2025` `HackTheBox-note`
# HTB Apocalypse 2025 Web : Whispers of the Moonbeam (Easy)

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

- In the heart of Valeria's bustling capital, the Moonbeam Tavern stands as a lively hub of whispers, wagers, and illicit dealings. Beneath the laughter of drunken patrons and the clinking of tankards, it is said that the tavern harbors more than just ale and merriment—it is a covert meeting ground for spies, thieves, and those loyal to Malakar's cause. The Fellowship has learned that within the hidden backrooms of the Moonbeam Tavern, a crucial piece of information is being traded—the location of the Shadow Veil Cartographer, an informant who possesses a long-lost map detailing Malakar’s stronghold defenses. If the fellowship is to stand any chance of breaching the Obsidian Citadel, they must obtain this map before it falls into enemy hands.

- **在 Valeria 熱鬧的市中心，月光酒館成為了耳語、賭注和非法交易的熱鬧中心。**
- **在醉醺醺的顧客笑聲和酒杯碰撞的聲音中，據說酒館隱藏的不僅是美酒和歡樂——它還是間諜、小偷以及忠於 Malakar 勢力的秘密聚會場所。**
- **兄弟會得知，在月光酒館的隱蔽後房中，一項關鍵的信息正在交易—— Shadow Veil Cartographer 的所在位置，這位情報員擁有一張失落已久、記載著 Malakar 堡壘防禦的地圖。**
- **如果兄弟會想要突破 Obsidian Citadel ，他們必須在敵人獲得這張地圖之前將其奪取。**

## <span class="red">題目</span>

- ![image](https://hackmd.io/_uploads/Hya-rHpnkg.png)
- ![image](https://hackmd.io/_uploads/Hk4mrrTh1g.png)

## <span class="red">解題過程</span>

1. **進入後先檢查一下網頁原始碼，發現了 ```/src/main.tsx``` 和 ```/@vite/client``` 並且查看一下，順便把 base64 的內容解雜湊**

- ![image](https://hackmd.io/_uploads/B1y8HBa21g.png)
- **```/src/main.tsx``` 主要部分:**
- ![image](https://hackmd.io/_uploads/SJmvfIp21e.png)
- ![image](https://hackmd.io/_uploads/S1nRHSThJl.png)

&emsp;

2. **稍微探索了一下：**
    (1) **在 gossip 的地方能發現 ```flag.txt``` 以及一些設定檔**
    (2) **在 observe 中可以看到 ```ps aux``` 的輸出結果**
    (3) **在 examine 可以看到自己的身分 (```whoami``` 的輸出結果)**

- ![image](https://hackmd.io/_uploads/SJzEYHahyx.png)
- ![image](https://hackmd.io/_uploads/HyrstS621x.png)

&emsp;

3. **首先在頁面左下角有看到 command Injection 的提示，所以按照提示先隨便用一些規定的語句，再用分號接上 payload (像是 ```ls```)，結果真的可以執行，並且 flag 就在當前目錄**

- ![image](https://hackmd.io/_uploads/SyrE_La2kl.png)
- ![image](https://hackmd.io/_uploads/H1KDv8p3ke.png)

&emsp;

4. **所以就能夠簡單的使用像是 ```examine; cat flag.txt``` 這樣的 payload 獲得 flag**

- ![image](https://hackmd.io/_uploads/rJebO86h1e.png)
