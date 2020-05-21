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

```cpp
		for (int i = 0; i < inputall.size(); ++i) {
		if (inputall[i].find("ui", 0) != inputall[i].npos) {
			uType(inputall[i],i);
		}
		else if (inputall[i][0] == 'l' || inputall[i].find("i", 0) != inputall[i].npos || inputall[i].find("jalr", 0) != inputall[i].npos) {
			iType(inputall[i],i);
		}
		else if (inputall[i][0] == 's' && inputall[i].find(" ", 0) == 2) {
			sType(inputall[i],i);
		}
		else if (inputall[i][0] == 'b') {
			bType(inputall[i],i);
		}
		else if (inputall[i].find("jal", 0) != inputall[i].npos) {
			ujType(inputall[i],i);
		}
		else {
			rType(inputall[i],i);
		}
	}
}
```
> 依照各指令字元出現之規律，找到其對應 type  
> 去呼叫相對之 function做處理  

```cpp
void rType(string input, int pc)
{
	string fun7, rs2, rs1, fun3, rd, op,str[4];
	int p;
	p = input.find(" ", 0);
	str[0] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	for(int i=1;i<4;i++){
		p = input.find(",", 0);
		str[i] = input.substr(0, p);
		input = input.substr(p + 1, input.length());
	}
	if (str[0] == "sub" || str[0] == "sra")
		fun7 = "0100000";
	else
		fun7 = "0000000";
	if (str[0] == "add" || str[0] == "sub")
		fun3 = "000";
	else if (str[0] == "sll")
		fun3 = "001";
	else if (str[0] == "slt")
		fun3 = "010";
	else if (str[0] == "sltu")
		fun3 = "011";
	else if (str[0] == "xor")
		fun3 = "100";
	else if (str[0] == "srl" || str[0] == "sra")
		fun3 = "101";
	else if (str[0] == "or")
		fun3 = "110";
	else
		fun3 = "111";
	
	op = "0110011";
	rs2 = toBin(str[3],5);
	rs1 = toBin(str[2],5);
	rd= toBin(str[1],5);
	cout << fun7 << " | " << rs2 << " | " << rs1 << " | " << fun3 << " | " << rd << " | " << op << endl;
}
``` 
> 處理字串將其切割為 6個部分存入 str陣列中。      
> fun3、fun7和 op code是依照表格直接賦予其值。   
> rs2、 rs1和 rd則是將數字的部分取出，去呼叫 toBin這個 function將數字轉為 binary。  
> 最後依" fun7 | rs2| rs1 | fun3 | rd | op "的格式輸出。  
 ```cpp
void iType(string input, int pc)
{
	string imm, rs1, fun3, rd, op, str[4];
	int p;
	p = input.find(" ", 0);
	str[0] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	if (str[0][0] == 'l') {
		p = input.find(",", 0);
		str[1] = input.substr(0, p);
		if(input[p+1]!=' ')
			input = input.substr(p + 1, input.length());
		else
			input = input.substr(p + 2, input.length());
		p = input.find("(", 0);
		str[3] = input.substr(0, p);
		input = input.substr(p + 1, input.length());
		str[2] = input.substr(0, input.length()-1);	
	}
	else {
		for (int i = 1; i < 4; i++) {
			p = input.find(",", 0);
			str[i] = input.substr(0, p);
			input = input.substr(p + 1, input.length());
		}
	}
	if (str[0] == "addi" || str[0] == "jalr"|| str[0] == "lb")
		fun3 = "000";
	else if (str[0] == "lh"|| str[0]=="slli")
		fun3 = "001";
	else if(str[0] == "slti"||str[0]=="lw")
		fun3 = "010";
	else if (str[0] == "sltiu")
		fun3 = "011";
	else if (str[0] == "xori"||str[0]=="lbu")
		fun3 = "100";
	else if (str[0] == "lhu"||str[0]=="srai"||str[0]=="srli")
		fun3 = "101";
	else if (str[0] == "ori")
		fun3 = "110";
	else if (str[0] == "andi")
		fun3 = "111";
	if (str[0] == "jalr")
		op = "1100111";
	else if (str[0][0] == 'l')
		op = "0000011";
	else
		op = "0010011";
	rs1 = toBin(str[2], 5);
	rd = toBin(str[1], 5);
	imm = toBin(str[3], 12);
	cout << imm << " | " << rs1 << " | " << fun3 << " | " << rd << " | " << op << endl;
}
```
> 分為 2種狀況處理，其中一種為 load指令語法與其他略有不同。   
> 處理字串將其切割為 5個部分存入 str陣列中。       
> fun3和 op code是依照表格直接賦予其值。     
> imm、 rs1和 rd則是將數字的部分取出，去呼叫 toBin這個 function將數字轉為 binary。     
> 最後依" imm | rs1| fun3 | rd | op "的格式輸出。   
```cpp
void sType(string input, int pc)
{
	string temp,immf,immb, rs2, rs1, fun3, op, str[4];
	int p;
	p = input.find(" ", 0);
	str[0] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	p = input.find(",", 0);
	str[1] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	p = input.find("(", 0);
	str[2] = input.substr(0, p);
	if (str[2][0] == ' ')
		str[2] = str[2].substr(1, str[2].length());
	input = input.substr(p + 1, input.length());
	str[3] = input.substr(0, input.length()-1);
	if (str[0] == "sb")
		fun3 = "000";
	else if (str[0] == "sh")
		fun3 = "001";
	else if (str[0] == "sw")
		fun3 = "010";
	else
		fun3 = "011";
	op = "0100011";
	rs2 = toBin(str[1], 5);
	rs1 = toBin(str[3], 5);
	temp = toBin(str[2], 12);
	immf = temp.substr(0, 7);
	immb = temp.substr(7, 11);
	cout << immf << " | " << rs2 << " | " << rs1 << " | " << fun3 << " | " << immb << " | " << op << endl;
}
```
> 處理字串將其切割為 6個部分存入 str陣列中。    
> fun3和 op code是依照表格直接賦予其值。    
> immf、immb、rs1和 rs2則是將數字的部分取出，去呼叫 toBin這個 function將數字轉為 binary。    
> 最後依" immf | rs2 | rs1 | fun3  | immb | op "的格式輸出。   

