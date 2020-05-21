# Project 1: Implementing RISC-V Assembler  
## 主要目標:  
此程式編寫目的為模擬 assembler將組合語言(RISC-V)轉成 mechine code的實際情況。  

## 程式運行方式:    

**該程式執行軟體為Visual 2019。**   
請將程式碼複製至上述軟體中，再編譯執行。  

## 簡要使用說明:  
- Input: 使用鍵盤輸入一段組合語言，輸入完畢後請按下2次enter鍵，即可執行轉換；其中，每段指令皆須做換行分隔 (可執行之基本指令為 RV32I系列+ label) ；label 後若不續接程式碼，請直接換行。   
> 輸入範例  
![avatar](https://upload.cc/i1/2020/05/21/xAVMCU.jpg)    

- Output: 螢幕顯示該段組合語言轉換後，各行指令對應之 mechine code，各行 code間以換行作分隔，其中又以 vartical bar作分隔，以供方便閱讀。   
> 輸出範例   
![avatar](https://upload.cc/i1/2020/05/21/oRfuqL.jpg)   

## 程式結構說明:  

### 基本原理:  
找出各 type規律並且以其為基準，將各指令對應正確之分類，並以其分類給予正確的 mechine code。  

![avatar](https://upload.cc/i1/2020/05/21/l6H1Bb.jpg)   

### 引用函式庫說明:  
```cpp
#include<iostream> //負責輸出/輸入
```
```cpp
#include<cstring>  //負責字串處理
```
```cpp
#include<vector> //container
```
```cpp
#include <sstream> //用以將 string轉換成 int  
```   

### Global變數說明:  
```cpp
vector<string> inputall, label;
//inputall: 用以存放每行輸入之指令資訊，待後續處理  
//label: 存放label名稱，以利後續branch發生時尋找目標位置  
```
```cpp
vector<int>lbnum; //存各label之下行指令的位置(程式內之行數)  
``` 

### 詳細程式碼說明  

```cpp
int main()
{
	string input;
	int p,pc;
	getline(cin, input);
	int cou = 0;
	bool push = true;

```  
> input取各行 cin的指令。  
> p為一暫存值，為找尋 string中某字元的位置。  
> cou紀錄當前的讀取行數。  
> push 決定該字串是否保留。  
 
```cpp
for (int i = 0; i < inputall.size(); ++i) {
		p = inputall[i].find(':', 0);
		if (p != inputall[i].npos) {

			label.push_back(inputall[i].substr(0, p));
			lbnum.push_back(i);
			if (p + 1 < inputall[i].length()) {
				inputall[i] = inputall[i].substr(p + 1, inputall[i].length());
				if (inputall[i][0] == ' ')
					inputall[i] = inputall[i].substr(1, inputall[i].length());
			}
		}
	}
```  
> 尋找 label，處理如 label後續接指令的狀況，並將 label及其下一指令位置存入各陣列中。  
> 處理每行輸入之字串，首先屏除註解，留下指令。  
> 如非上述狀況，就將該行指令存入inputall，待後續作處理。  