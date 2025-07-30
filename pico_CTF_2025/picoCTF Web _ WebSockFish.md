##### tags:`picoCTF_2025`

# picoCTF Web : WebSockFish

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

- ![image](https://hackmd.io/_uploads/ByS_dWB3Je.png)
- ![image](https://hackmd.io/_uploads/Syk5dbB2Jl.png)
- ![image](https://hackmd.io/_uploads/r1v2ubr3yg.png)
- ![image](https://hackmd.io/_uploads/rk9It-B2kl.png)
- ![image](https://hackmd.io/_uploads/H1dBFWS31e.png)

## <span class="red">**解題過程**</span>

1. **根據題目以及我直接跟 AI 下一把，直接贏是不太實際的(出題者將 AI 思考深度設定為 10)，所以必須要找漏洞想辦法「作弊」**

- ![image](https://hackmd.io/_uploads/rJrG6-H3ye.png)

&emsp;

2. **首先看到了一些不太懂的名詞，所以我做了一下 research ，像是**

- **```stockfish.postMessage("uci");``` 中的 uci：Universal Chess Interface （通用西洋棋介面），目的是讓 GUI（如這個網頁程式）可以與國際象棋引擎（如 Stockfish）溝通。**
- **FEN = Forsyth-Edwards Notation ，標準化的國際象棋棋盤表示法**

&emsp;

3. **接續前面的維，根據 ```stockfish.onmessage()``` 的語法，我發現是可以直接操控棋子走向的，像是**

- ![image](https://hackmd.io/_uploads/B1xoXzrhyx.png)

&emsp;

4. **那我就可以設計一下，故意讓走入一些西洋棋的開局陷阱，以此來快速獲勝 (可以一次下完指令再更新棋盤就好，這裡方便讓讀者了解當前局面)**

- ![image](https://hackmd.io/_uploads/H1phBMr21g.png)
- ![image](https://hackmd.io/_uploads/rJr68MH3yl.png)

&emsp;

5. **最後 checkmate (殺棋)的時候自己動手下，但是<span class="red">沒有 flag</span> 。所以猜測這題應該是要利用 send message 之類的跟 server 互動，或是利用 websocket 來拿到 flag**

- ![image](https://hackmd.io/_uploads/H1N5DzSnJg.png)

&emsp;

6. **於是我把 URL 丟到 burp suite 開的瀏覽器上進行監聽**

- ![image](https://hackmd.io/_uploads/HJHMqVBhkl.png)
- ![image](https://hackmd.io/_uploads/SykhFEr2Jg.png)

&emsp;

7. **可以發現 client 會寄送 websocket 給 server ，內容是 eval 值(也就是評分)，而 server 會回傳相對應的回覆給 client ，所以這裡我就故意寄送 -99999 (這代表輸了，因為當電腦對已經死棋的評分會變成負無限大的分，所以這裡用 -99999) ，成功獲得 flag**

- ![image](https://hackmd.io/_uploads/BJK-h4Bhyx.png)
- ![image](https://hackmd.io/_uploads/B1br3VH31g.png)
