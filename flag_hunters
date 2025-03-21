# **Flag_Hunters: Command injection - 利用 RETURN 0 取得 Flag**

## **題目分析**
題目給出了一段 Python 程式碼，這段程式會讀取 `flag.txt` 並將其插入到 `secret_intro`，然後播放一首關於 CTF 挑戰的歌曲。整個歌曲的結構由 `[VERSE]`、`[REFRAIN]`（副歌）、`RETURN`（跳轉指令）以及 `CROWD`（輸入點）所組成。

## **關鍵程式碼解析**
### **1. `refrain` 與 `refrain_return` 的作用**
在 `reader()` 函式中，程式會搜尋 `[REFRAIN]` 和 `RETURN` 的位置，並存入變數：
```python
for i in range(0, len(song_lines)):
    if song_lines[i] == startLabel:
        start = i + 1
    elif song_lines[i] == '[REFRAIN]':
        refrain = i + 1  # 記錄副歌開始的行號
    elif song_lines[i] == 'RETURN':
        refrain_return = i  # 記錄 RETURN 這行的行號
```
- `refrain`：存 `[REFRAIN]` 之後的行號，確保 `REFRAIN` 指令能正確跳轉到副歌。
- `refrain_return`：存 `RETURN` 指令的位置，使得 `RETURN` 能跳回正確的地方。

### **2. `RETURN` 指令的作用**
當 `reader()` 解析 `RETURN` 時，它會讓 `lip`（當前行號）跳轉到指定位置：
```python
elif re.match(r"RETURN [0-9]+", line):
    lip = int(line.split()[1])  # 讓 lip 跳轉到 RETURN 指定的行
```
這讓程式可以根據 `RETURN` 的參數回到不同的位置。

### **3. `CROWD` 指令的漏洞**
```python
elif re.match(r"CROWD.*", line):
    crowd = input('Crowd: ')
    song_lines[lip] = 'Crowd: ' + crowd
    lip += 1
```
當程式遇到 `CROWD` 時，它會詢問使用者輸入內容，並將該內容覆蓋 `CROWD` 這一行。**這裡是一個輸入點，我們可以輸入 `RETURN 0` 來操控程式流程！**

## **利用 RETURN 0 獲取 Flag**
當我們輸入 `RETURN 0` 給 `CROWD`，程式就會將該行替換為：
```
Crowd: RETURN 0
```
當程式再次讀取這一行時，它會解析 `RETURN 0`，並將 `lip` 設為 `0`，即跳回程式的最開頭，這時 `secret_intro` 會被輸出：
```
Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, <FLAG>
```
此時，`flag.txt` 的內容會被直接顯示在終端上，成功獲取 flag！🚩

## **總結**
### **攻擊流程**
1. 執行程式，當到 `[REFRAIN]` 會遇到 `CROWD`。
2. 在 `Crowd:` 輸入 `RETURN 0`。
3. 當程式讀取 `RETURN 0`，`lip` 被設為 `0`，程式跳回 `secret_intro`，輸出 `flag.txt` 內容。
