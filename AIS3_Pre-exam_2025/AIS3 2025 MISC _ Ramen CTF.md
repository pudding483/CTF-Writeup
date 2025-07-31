##### tags: `AIS3_Pre-exam_2025`
# AIS3 2025 MISC : Ramen CTF
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

- ![image](https://hackmd.io/_uploads/r1i8BgJfxg.png)

## <span class="red">解題過程</span>

1. **這題把拿到的 zip 檔解壓縮後會拿到圖片，但是圖片很明顯被遮住了。一開始是想透過移除「猴子 emoji 」的方式，但後來發現不太現實，所以轉而從發票下手**

- ![image](https://hackmd.io/_uploads/rkjqHxkMee.png)

&emsp;

2. **圖片右邊有一張發票，而發票左邊的 QRcode 是完整的，所以掃描看看。掃描 QRcode 之後可以獲得一串數字加上一些中文字，針對發票的格式可以得出一些有用的資訊，其中就是「賣方統編」**

- ![image](https://hackmd.io/_uploads/S1_7Ie1Gle.png)
- ![image](https://hackmd.io/_uploads/r1y5seyMgg.png)
- ![image](https://hackmd.io/_uploads/r1ucUe1fge.png)

&emsp;

3. **透過財政部稅務入口網(依營業人統一編號查詢結果)可以查到這家拉麵店(複製地址到 Google map 查詢)，然後透過最後的蝦拉可以猜測是蝦拉麵**

- **財務部稅務入口網查詢連結：https://www.etax.nat.gov.tw/etwmain/etw113w1/ban/query**

- ![image](https://hackmd.io/_uploads/Sya7wxyGxe.png)

- flag🚩 : `AIS3{樂山溫泉拉麵:蝦拉麵}`