##### tags: `SwampCTF 2025`
# SwampCTF 2025 Web : Serialies

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

- ![image](https://hackmd.io/_uploads/S1daBQHTyg.png)
- ![image](https://hackmd.io/_uploads/SyhAx7STkl.png)

## <span class="red">解題過程</span>

1. **根據我查看題目給的原始碼，在 ```PersonController.java``` 中我發現了有 ```RequestMapping``` 到 ```/api/person``` 這個路徑，所以我直接在 URL 加上 ```/api/person``` 這個路徑，就能夠發現許多不應該能看見的隱密資訊**

- ![image](https://hackmd.io/_uploads/By3EQXBTyx.png)
- ![image](https://hackmd.io/_uploads/H1yZNQBTkl.png)

&emsp;

2. **而我在這邊直接 ctrl+f 搜尋 swamp 就找到了 flag**

- ![image](https://hackmd.io/_uploads/HJpNVXBpJl.png)
- flag🚩：swampCTF{f1l3_r34d_4nd_d3s3r14l1z3_pwn4g3_x7q9z2r5v8}