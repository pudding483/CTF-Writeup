##### tags: `SwampCTF 2025`
# SwampCTF 2025 Web : Editor

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

- ![image](https://hackmd.io/_uploads/SyfoMj8pJl.png)

## <span class="red">解題過程</span>

1. **進入後可以看到這是一個自定義的 html ，根據題目的註解，應該是希望我們利用 XSS 的技巧存取到根目錄的 ```flag.txt```**

- ![image](https://hackmd.io/_uploads/HJyYXj8pkg.png)
- ![image](https://hackmd.io/_uploads/BkVLEjL6kx.png)

&emsp;

2. **有很多種方式可以顯示，先嘗試看看最簡單的 payload：
```<iframe src="/flag.txt"></iframe>```
結果成功獲得 flag**

- ![image](https://hackmd.io/_uploads/HyBEDs86kx.png)
- flag🚩：swampCTF{c55_qu3r135_n07_j5}

&emsp;

## <span class="red">補充</span>

1. **在沒有 CSP 的情況下才能夠使用 ```<iframe src="/flag.txt"></iframe>```**

2. **還能夠使用像是：**
    
    (1) **```<object data = /flag.txt></object>``` (Content-Type 必須要是 html) 才能用**
    
    - ![image](https://hackmd.io/_uploads/ryhY5oIpkl.png)
    
    (2) **```<a href = '/flag.txt'>6</a>```**
    
    (3) **使用 style class 如下：**
    ```html=
    <style class='custom-user-css'>
        @font-face {
            font-family: 'flag';
            src: url('http://chals.swampctf.com:47821/flag.txt');
        }
        body {
            font-family: 'flag', sans-serif;
            content: "Flag is being loaded as a font (check server logs)";
        }</style>
    ```
    
    (4) **...待補充**