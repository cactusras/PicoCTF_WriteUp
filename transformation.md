# transformation: Unicode Encoding Trick

## **題目分析**
題目給出的 Python 編碼方式如下：
```python
enc = ''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
```
並提供加密後的字串：
```
灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥ㄴㅡて㝽
```

這意味著，原始 `flag` 被分成兩個一組，每組合併為一個 **16-bit Unicode 字元**。

## **解碼過程**
要還原 `flag`，我們需要將每個 Unicode 字元拆回兩個 ASCII 字元。

### **步驟 1：觀察模式**
每個加密後的字元是 Unicode，範圍通常超出標準 ASCII (`0x00 - 0x7F`)。

考慮 `chr((ord(flag[i]) << 8) + ord(flag[i + 1]))` 的操作：
- `ord(flag[i]) << 8` 讓 `flag[i]` 變成高 8 位
- `+ ord(flag[i+1])` 將 `flag[i+1]` 加入低 8 位
- 最後 `chr(...)` 轉換成 Unicode

這告訴我們，我們可以使用 `ord()` 來還原：
```python
dec = ''.join([chr(ord(c) >> 8) + chr(ord(c) & 0xFF) for c in enc])
```

### **步驟 2：解碼**
完整的 Python 解碼腳本如下：
```python
enc = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥ㄴㅡて㝽"
dec = ''.join([chr(ord(c) >> 8) + chr(ord(c) & 0xFF) for c in enc])
print(dec)  # 恢復原始 flag
```
這將輸出原始 `flag`。

# CTF Write-up: Unicode Encoding Trick

## **變形混淆手法及判讀方式比較**

| 方法 | 主要特徵 | 解密方式 |
|------|---------|---------|
| Unicode 拼接 (`(A << 8) + B`) | 兩個 ASCII 字元合併為一個 Unicode | `ord(c) >> 8` 及 `ord(c) & 0xFF` 解析 |
| Base64 | `=` 結尾，可讀字母數字 | `base64.b64decode()` |
| ROT13 | 單詞結構沒變，但字母不同 | `codecs.decode(..., 'rot_13')` |
| XOR | 字元亂碼但長度相同 | `ord(c) ^ key` 嘗試 key |
| URL | `%XX` 形式 | `urllib.parse.unquote()` |
| Hex | 只有 `0-9` 和 `a-f` | `bytes.fromhex().decode()` |
| Unicode `\uXXXX` | `\u0066` 格式 | `decode("unicode_escape")` |
| Brainfuck | `+ - < >` 大量出現 | 需要 Brainfuck 解碼器 |
| Python `eval()` | `chr()+chr()` 拼接 | `eval()` |
| 變數替換 | `chr()` + 數字變數 | 手動計算 `chr()` |
| Morse Code | `.` 和 `-` 組合 | `Morse().decode()` |


## **結論**
本題運用了 **Unicode 混淆**（每兩個 ASCII 字元合併為一個 Unicode 字元）來隱藏 `flag`，但透過 `ord(c) >> 8` 和 `ord(c) & 0xFF` 即可還原。

在 CTF 挑戰中，常見的變形手法還包括 **Base64、ROT13、XOR** 等，遇到可疑的字串時，可以依據長度、可讀性、字元範圍來判斷可能的加密方式。

