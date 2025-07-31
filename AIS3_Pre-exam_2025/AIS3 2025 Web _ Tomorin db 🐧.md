##### tags: `AIS3_Pre-exam_2025`
# AIS3 2025 Web : Tomorin db 🐧

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

- ![image](https://hackmd.io/_uploads/rJqJah0-ee.png)

## <span class="red">解題過程</span>

1. **前端能看到有 flag ，但是被設定成重新導向，所以目標是請求 flag ，但是要繞過重新導向。嘗試目錄爆破，最後成功用 
`curl http://chals1.ais3.org:30000/flag%2f.` 讀到 flag 檔 
(`%2f` 是 `/` 做 URL Encode)**

- ![image](https://hackmd.io/_uploads/rJBGanRWgg.png)
- ![image](https://hackmd.io/_uploads/Sy6P02CZxe.png)
- flag🚩 : `AIS3{G01ang_H2v3_a_c0O1_way!!!_Us3ing_C0NN3ct_M3Th07_L0l@T0m0r1n_1s_cute_D0_yo7_L0ve_t0MoRIN?}`