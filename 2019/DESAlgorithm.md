---
title: DES算法的JAVA实现（ECB模式）
urlname: DESAlgorithm
tags:
  - Algorithm
  - DES
  - ECB
  - Java
copyright: true
category: Code
abbrlink: 42217
date: 2018-10-21 17:45:36
---



## 一、算法原理概述

参考自老师的PPT

<!-- more --> 

### 加密过程

![1541244158363](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541244158363.png)

![1541244631502](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541244631502.png)

#### 初始置换IP

![1541244740744](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541244740744.png)

#### 迭代T

![1541245001052](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541245001052.png)

##### Feistel轮函数

![1541245200505](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541245200505.png)

##### 子秘钥生成

![1541245413226](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541245413226.png)

#### 逆置换IP-1

![1541245475171](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541245475171.png)



### 解密过程

![1541244245356](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1541244245356.png)





## 二、总体结构

```java
class DES{
    //构造函数
	public DES(StringBuffer text, StringBuffer key, int mode) throws Exception {}
	//PKCS5填充处理明文
	public StringBuffer dealText(final StringBuffer origin) {}
	//初始置换IP
	public void start() {
		//把明文或者密文转换成二进制字符串
		//分组加密或解密
        IP(...);
	}
	//IP置换
	public void IP(final String plaintextBinary) {
		//通过IP置换明文
        //把置换后的明文分为左右两块
        iterationT(...);
	}
	//迭代T
	public void iterationT(StringBuffer L, StringBuffer R) {
		//得到子秘钥
        getSbuKey(...);
		//16次迭代
        	feistel(...);
		//左右交换
		//逆置换
        //生成字符串明文或密文
        //END!
	}
	//Feistel轮函数
	public StringBuffer feistel(final StringBuffer R, final StringBuffer subKey ) {
		//E扩展
		//异或运算
		//S-Box
		//P置换
	}
	//子秘钥生成
	public StringBuffer[] getSubKey() {
		//把key转换为二进制
		//PC1置换
	    // LS循环16轮生成子密钥
            // 把左右两块合并
            // PC2置换
            // 根据PC2压缩C0D0，得到子密钥
	}
    //主函数
    public static void main(String[] args){start();}
}
```

## 三、模块分解

### PKCS5填充处理明文

```java
public StringBuffer dealText(final StringBuffer origin) {
		StringBuffer textBinary = new StringBuffer();
		//使用PKCS#5/PKCS7填充
		int max = group*8;
		int padding = max - origin.length();
        for (int i = 0; i < max; ++i) {
        	StringBuffer charBinary;
        	if(i >= origin.length()) {
        		charBinary = new StringBuffer(Integer.toBinaryString(padding));
        	}else
        		charBinary = new StringBuffer(Integer.toBinaryString(origin.charAt(i)));
            while (charBinary.length() < 8) {
            	charBinary.insert(0, 0);
            }
            textBinary.append(charBinary);
        }
        return textBinary;
	}
```

### 字符串转二进制

```java
	public StringBuffer string2Binary(final StringBuffer origin) {
		StringBuffer textBinary = new StringBuffer();
        for (int i = 0; i < origin.length(); ++i) {
        	StringBuffer charBinary = new StringBuffer(Integer.toBinaryString(origin.charAt(i)));
            while (charBinary.length() < 8) {
            	charBinary.insert(0, 0);
            }
            textBinary.append(charBinary);
        }
        return textBinary;
	}
```

### IP置换

```java
	public void IP(final String plaintextBinary) {
		//通过IP置换明文
        StringBuffer substitutePlaintext = new StringBuffer(); // 存储置换后的明文
        for (int i = 0; i < 64; ++i) {
            substitutePlaintext.append(plaintextBinary.charAt(IP[i] - 1));
        }
        //把置换后的明文分为左右两块
        StringBuffer L = new StringBuffer(substitutePlaintext.substring(0, 32));
        StringBuffer R = new StringBuffer(substitutePlaintext.substring(32));
        iterationT(L, R);
	}
```

### Feistel轮函数

```java
	public StringBuffer feistel(final StringBuffer R, final StringBuffer subKey ) {
		//E扩展
		StringBuffer RExtent = new StringBuffer(); // 存储扩展后的右边
        for (int i = 0; i < 48; ++i) {
            RExtent.append(R.charAt(E[i] - 1));
        }
		//异或运算
        for (int i = 0; i < 48; ++i) {
            RExtent.replace(i, i + 1, (RExtent.charAt(i) == subKey.charAt(i)) ? "0" :"1");
        }
		//S-Box
        StringBuffer SBoxString = new StringBuffer();
        for(int i = 0; i < 8; ++i) {
        	String SBoxInput = RExtent.substring(i * 6, (i + 1) * 6);
        	int row = Integer.parseInt(Character.toString(SBoxInput.charAt(0)) + SBoxInput.charAt(5), 2);
            int column = Integer.parseInt(SBoxInput.substring(1, 5), 2);
            
            StringBuffer SBoxOutput = new StringBuffer(Integer.toBinaryString(SBox[i][row * 16 + column]));
            while (SBoxOutput.length() < 4) {
                SBoxOutput.insert(0, 0);
            }
            SBoxString.append(SBoxOutput);
        }
		//P置换
        StringBuffer substituteSBoxString= new StringBuffer(); // 存储置换后的R
        for (int i = 0; i < 32; ++i) {
        	substituteSBoxString.append(SBoxString.charAt(P[i] - 1));
        }
        return substituteSBoxString;
	}
	
```

### 子密钥生成

```java
//子秘钥生成
public StringBuffer[] getSubKey() {
	//把key转换为二进制
	StringBuffer keyBinary = string2Binary(key);
	
	StringBuffer[] subKey = new StringBuffer[16]; // 存储子密钥
	//PC1置换
	StringBuffer C0 = new StringBuffer(); // 存储密钥左块
    StringBuffer D0 = new StringBuffer(); // 存储密钥右块
    for (int i = 0; i < 28; ++i) {
        C0.append(keyBinary.charAt(PC1[i] - 1));
        D0.append(keyBinary.charAt(PC1[i + 28] - 1));
    }
    
    // LS循环16轮生成子密钥
    for (int i = 0; i < 16; ++i) {
        // 循环左移
    	char mTemp;
        mTemp = C0.charAt(0);
        C0.deleteCharAt(0);
        C0.append(mTemp);
        mTemp = D0.charAt(0);
        D0.deleteCharAt(0);
        D0.append(mTemp);
        if(i != 0 && i != 1 && i != 8 && i != 15) {
        	mTemp = C0.charAt(0);
            C0.deleteCharAt(0);
            C0.append(mTemp);
            mTemp = D0.charAt(0);
            D0.deleteCharAt(0);
            D0.append(mTemp);
        }
        // 把左右两块合并
        StringBuffer C0D0 = new StringBuffer(C0.toString() + D0.toString());
        
        // PC2置换
        // 根据PC2压缩C0D0，得到子密钥
        StringBuffer C0D0Temp = new StringBuffer();
        for (int j = 0; j < 48; ++j) {
            C0D0Temp.append(C0D0.charAt(PC2[j] - 1));
        }
        subKey[i] = C0D0Temp;
    }
    return subKey;
}
```
### 十六次迭代

```java
//迭代T
	public void iterationT(StringBuffer L, StringBuffer R) {
		//得到子秘钥
		StringBuffer[] subKey = getSubKey();
		//这是解密的子秘钥
		if(encrypt_decrypt != 0) {
			StringBuffer[] temp = getSubKey();
			for (int i = 0; i < 16; ++i) {
                subKey[i] = temp[15 - i];
            }
		}
		StringBuffer Lbackup = new StringBuffer();
		StringBuffer Rbackup = new StringBuffer();
		Lbackup.replace(0, L.length(), L.toString());
		Rbackup.replace(0, R.length(), R.toString());
		//16次迭代
		for(int i = 0; i < 16; i++) {
			//不能直接用等于号!!!!!
			StringBuffer tempL = new StringBuffer(L);
			StringBuffer tempR = new StringBuffer(R);
			//字符串赋值
			L.replace(0, 32, R.toString());
			StringBuffer feistelString = feistel(tempR, subKey[i]);
			
			for(int j = 0; j < 32; j++) {
				R.replace(j, j + 1, (tempL.charAt(j) == feistelString.charAt(j)) ? "0" :"1");
			}
		}
		
		//左右交换
		StringBuffer RL = new StringBuffer(R.toString() + L.toString());
		//逆置换
		StringBuffer substituteRL = new StringBuffer(); // 存储置换后的LR
        for (int i = 0; i < 64; ++i) {
            substituteRL.append(RL.charAt(IPReverse[i] - 1));
        }
        //生成字符串明文或密文
        for (int i = 0; i < 8; ++i) {
            String temp = substituteRL.substring(i * 8, (i + 1) * 8);
            //加密
            if(encrypt_decrypt == 0) {
            	ciphertext.append((char) Integer.parseInt(temp, 2));
            	StringBuffer tempHex = new StringBuffer(Integer.toHexString(Integer.parseInt(temp, 2)));
            	while(tempHex.length() < 2) {
            		tempHex.insert(0, "0");
            	}
            	ciphertextHex.append(tempHex + " ");
            //解密
            }else {
            	plaintext.append((char) Integer.parseInt(temp, 2));
            }
            	
        }
        //END!
	}
```



## 四、数据结构

`StringBuffer plaintext`  明文字符串

> 理论上明文字符串长度可以任意，加密时八个字符(字节)一组，也就是64bit，不够8个字节一组使用`PKCS#5`标准填充
>
> 假设待加密数据长度为x，那么将会在后面padding的字节数目为8-(x%8)，每个`padding`的字节值是8-(x%8)。
>
> 特别地，当待加密数据长度x恰好是8的整数倍，也是要在后面多增加8个字节，每个字节是0x08。

`StringBuffer ciphertext`  密文字符串

> 密文字符串只能是8字节的整数倍，因为它是通过明文加密得到的，如果不是8的整数倍也就不存在明文了

`StringBuffer key` 密钥

> 这里的密钥是用字符串表示，不是十六进制！！！

`StringBuffer ciphertextHex` 密文的十六进制表示
`int group` 分组数(`DES`以`64bit`为一个分组，所以需要根据每组来加密相应的明文)
`int encrypt_decrypt` 算法模式：0表示加密，其余表示解密

`int[] IP`	IP置换表
`int[] IPReverse`  IP逆置换表

`int[] E` E扩展表
`int[] P`  P置换
`int[] PC1` PC1置换
`int[] PC2` PC2压缩置换
`int[][] SBox` 8个S-Box

## 五、运行结果

> 运行说明:
>
> 主函数分为两个部分：加密和解密
>
> **加密部分**输入明文，密钥和算法模式(0表示加密，其余表示解密)，输出密文
>
> **解密部分**输入密文（这里直接用上面生成的密文，好对比结果），密钥（与上面同样的密钥），算法模式，应该输出同样的明文

![1540126222637](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/DESAlgorithm/1540126222637.png)

**可以看到确实能够实现加密再解密得到原数据！**

## 六、源代码

```java

public class DES {
	private StringBuffer plaintext;
	private StringBuffer ciphertext;
	private StringBuffer key;
	private StringBuffer ciphertextHex;
	private int group;
	//0表示加密,其余表示解密
	private int encrypt_decrypt;
	//IP置换
	private static final int[] IP = { 
			58, 50, 42, 34, 26, 18, 10, 2, 
			60, 52, 44, 36, 28, 20, 12, 4, 
			62, 54, 46, 38, 30, 22, 14, 6, 
			64, 56, 48, 40, 32, 24, 16, 8, 
			57, 49, 41, 33, 25, 17, 9, 1, 
			59, 51, 43, 35, 27, 19, 11, 3, 
			61, 53, 45, 37, 29, 21, 13, 5, 
			63, 55, 47, 39, 31, 23, 15, 7
			}; 
	//IP逆置换
	private static final int[] IPReverse = { 
			40, 8, 48, 16, 56, 24, 64, 32, 
			39, 7, 47, 15, 55, 23, 63, 31, 
			38, 6, 46, 14, 54, 22, 62, 30, 
			37, 5, 45, 13, 53, 21, 61, 29, 
			36, 4, 44, 12, 52, 20, 60, 28, 
			35, 3, 43, 11, 51, 19, 59, 27, 
			34, 2, 42, 10, 50, 18, 58, 26, 
			33, 1, 41, 9, 49, 17, 57, 25 };
	// E位选择表(扩展置换表)
    private static final int[] E = { 
    		32, 1, 2, 3, 4, 5, 
    		4, 5, 6, 7, 8, 9, 
    		8, 9, 10, 11, 12, 13, 
    		12, 13, 14, 15, 16, 17, 
    		16, 17, 18, 19, 20, 21, 
    		20, 21, 22, 23, 24, 25, 
    		24, 25, 26, 27, 28, 29, 
    		28, 29, 30, 31, 32, 1 };
    //P换位表(单纯换位表)
    private static final int[] P = { 
    		16, 7, 20, 21, 
    		29, 12, 28, 17, 
    		1, 15, 23, 26, 
    		5, 18, 31, 10, 
    		2, 8, 24, 14, 
    		32, 27, 3, 9,
            19, 13, 30, 6, 
            22, 11, 4, 25 };
    //PC1
    private static final int[] PC1 = { 
    		57, 49, 41, 33, 25, 17, 9, 
    		1, 58, 50, 42, 34, 26, 18, 
    		10, 2, 59, 51, 43, 35, 27, 
    		19, 11, 3, 60, 52, 44, 36, 
    		63, 55, 47, 39, 31, 23, 15, 
    		7, 62, 54, 46, 38, 30, 22, 
    		14, 6, 61, 53, 45, 37, 29, 
    		21, 13,5, 28, 20, 12, 4 };
    // PC2
    private static final int[] PC2 = { 
    		14, 17, 11, 24, 1, 5, 
    		3, 28, 15, 6, 21, 10,
    		23, 19, 12, 4, 26, 8, 
    		16, 7, 27, 20, 13, 2,
            41, 52, 31, 37, 47, 55, 
            30, 40, 51, 45, 33, 48, 
            44, 49, 39, 56, 34, 53, 
            46, 42, 50, 36, 29, 32 };
    // SBox
    private static final int[][] SBox = {
            // S1
            { 14, 4, 13, 1, 2, 15, 11, 8, 3, 10, 6, 12, 5, 9, 0, 7, 
              0, 15, 7, 4, 14, 2, 13, 1, 10, 6, 12, 11, 9, 5, 3, 8, 
              4, 1, 14, 8, 13, 6, 2, 11, 15, 12, 9, 7, 3, 10, 5, 0, 
              15, 12, 8, 2, 4, 9, 1, 7, 5, 11, 3, 14, 10,0, 6, 13 },
            // S2
            { 15, 1, 8, 14, 6, 11, 3, 4, 9, 7, 2, 13, 12, 0, 5, 10, 
              3, 13, 4, 7, 15, 2, 8, 14, 12, 0, 1, 10, 6, 9, 11, 5, 
              0, 14, 7, 11, 10, 4, 13, 1, 5, 8, 12, 6, 9, 3, 2, 15, 
              13, 8, 10, 1, 3, 15, 4, 2, 11, 6, 7, 12, 0, 5, 14, 9 },
            // S3
            { 10, 0, 9, 14, 6, 3, 15, 5, 1, 13, 12, 7, 11, 4, 2, 8, 
              13, 7, 0, 9, 3, 4, 6, 10, 2, 8, 5, 14, 12, 11, 15,1, 
              13, 6, 4, 9, 8, 15, 3, 0, 11, 1, 2, 12, 5, 10, 14, 7, 
              1, 10, 13, 0, 6, 9, 8, 7, 4, 15, 14, 3, 11,5, 2, 12 },
            // S4
            { 7, 13, 14, 3, 0, 6, 9, 10, 1, 2, 8, 5, 11, 12, 4, 15, 
              13, 8, 11, 5, 6, 15, 0, 3, 4, 7, 2, 12, 1, 10, 14, 9, 
              10, 6, 9, 0, 12, 11, 7, 13, 15, 1, 3, 14, 5, 2, 8, 4,
              3, 15, 0, 6, 10, 1, 13, 8, 9, 4, 5, 11, 12,7, 2, 14 },
            // S5
            { 2, 12, 4, 1, 7, 10, 11, 6, 8, 5, 3, 15, 13, 0, 14, 9, 
             14, 11, 2, 12, 4, 7, 13, 1, 5, 0, 15, 10, 3, 9, 8,  6, 
             4, 2, 1, 11, 10, 13, 7, 8, 15, 9, 12, 5, 6, 3, 0, 14, 
             11, 8, 12, 7, 1, 14, 2, 13, 6, 15, 0, 9, 10, 4, 5, 3 },
            // S6
            { 12, 1, 10, 15, 9, 2, 6, 8, 0, 13, 3, 4, 14, 7, 5, 11, 
              10, 15, 4, 2, 7, 12, 9, 5, 6, 1, 13, 14, 0, 11, 3, 8, 
              9, 14, 15, 5, 2, 8, 12, 3, 7, 0, 4, 10, 1, 13, 11, 6, 
              4, 3, 2, 12, 9, 5, 15, 10, 11, 14, 1, 7, 6, 0, 8, 13 },
            // S7
            { 4, 11, 2, 14, 15, 0, 8, 13, 3, 12, 9, 7, 5, 10, 6, 1, 
              13, 0, 11, 7, 4, 9, 1, 10, 14, 3, 5, 12, 2, 15, 8,6, 
              1, 4, 11, 13, 12, 3, 7, 14, 10, 15, 6, 8, 0, 5, 9, 2, 
              6, 11, 13, 8, 1, 4, 10, 7, 9, 5, 0, 15, 14, 2, 3, 12 },
            // S8
            { 13, 2, 8, 4, 6, 15, 11, 1, 10, 9, 3, 14, 5, 0, 12, 7, 
              1, 15, 13, 8, 10, 3, 7, 4, 12, 5, 6, 11, 0, 14, 9, 2, 
              7, 11, 4, 1, 9, 12, 14, 2, 0, 6, 10, 13, 15, 3, 5, 8, 
              2, 1, 14, 7, 4, 10, 8, 13, 15, 12, 9, 0, 3, 5, 6, 11 } };
    
    //构造函数
	public DES(StringBuffer text, StringBuffer key, int mode) throws Exception {
		plaintext = new StringBuffer(mode == 0 ? text : "");
		ciphertext = new StringBuffer(mode != 0 ? text : "");
		ciphertextHex = new StringBuffer(mode != 0 ? text : "");
		group = mode == 0 ? (text.length()/8 + 1) : (text.length()/8);//分组
		if(ciphertext.length() % 8!= 0)
			throw new Exception("CipherText is not standard!");
		if(key.length() < 8)
			throw new Exception("Key is not standard!");
		this.key = key;
		encrypt_decrypt = mode;
	}
	//PKCS5填充处理明文
	public StringBuffer dealText(final StringBuffer origin) {
		StringBuffer textBinary = new StringBuffer();
		//使用PKCS#5/PKCS7填充
		int max = group*8;
		int padding = max - origin.length();
        for (int i = 0; i < max; ++i) {
        	StringBuffer charBinary;
        	if(i >= origin.length()) {
        		charBinary = new StringBuffer(Integer.toBinaryString(padding));
        	}else
        		charBinary = new StringBuffer(Integer.toBinaryString(origin.charAt(i)));
            while (charBinary.length() < 8) {
            	charBinary.insert(0, 0);
            }
            textBinary.append(charBinary);
        }
        return textBinary;
	}
	//字符串转二进制
	public StringBuffer string2Binary(final StringBuffer origin) {
		StringBuffer textBinary = new StringBuffer();
        for (int i = 0; i < origin.length(); ++i) {
        	StringBuffer charBinary = new StringBuffer(Integer.toBinaryString(origin.charAt(i)));
            while (charBinary.length() < 8) {
            	charBinary.insert(0, 0);
            }
            textBinary.append(charBinary);
        }
        return textBinary;
	}
	//初始置换IP
	public void start() {
		//把明文或者密文转换成二进制字符串
		StringBuffer plaintextBinary = new StringBuffer();
		if(encrypt_decrypt == 0)
			//使用PKCS#5/PKCS7填充
			plaintextBinary = dealText(plaintext);
		else {
			//解析字符密文
			plaintextBinary = string2Binary(ciphertext);
		}
		//分组加密或解密
		int loop = group;
        for(int i = 0; i < loop; i++) {
        	group--;
        	IP(plaintextBinary.subSequence(i*64, (i + 1)*64).toString());
        }
	}
	//IP置换
	public void IP(final String plaintextBinary) {
		//通过IP置换明文
        StringBuffer substitutePlaintext = new StringBuffer(); // 存储置换后的明文
        for (int i = 0; i < 64; ++i) {
            substitutePlaintext.append(plaintextBinary.charAt(IP[i] - 1));
        }
        //把置换后的明文分为左右两块
        StringBuffer L = new StringBuffer(substitutePlaintext.substring(0, 32));
        StringBuffer R = new StringBuffer(substitutePlaintext.substring(32));
        iterationT(L, R);
	}
	//迭代T
	public void iterationT(StringBuffer L, StringBuffer R) {
		//得到子秘钥
		StringBuffer[] subKey = getSubKey();
		//这是解密的子秘钥
		if(encrypt_decrypt != 0) {
			StringBuffer[] temp = getSubKey();
			for (int i = 0; i < 16; ++i) {
                subKey[i] = temp[15 - i];
            }
		}
		StringBuffer Lbackup = new StringBuffer();
		StringBuffer Rbackup = new StringBuffer();
		Lbackup.replace(0, L.length(), L.toString());
		Rbackup.replace(0, R.length(), R.toString());
		//16次迭代
		for(int i = 0; i < 16; i++) {
			//不能直接用等于号!!!!!
			StringBuffer tempL = new StringBuffer(L);
			StringBuffer tempR = new StringBuffer(R);
			//字符串赋值
			L.replace(0, 32, R.toString());
			StringBuffer feistelString = feistel(tempR, subKey[i]);
			
			for(int j = 0; j < 32; j++) {
				R.replace(j, j + 1, (tempL.charAt(j) == feistelString.charAt(j)) ? "0" :"1");
			}
		}
		
		//左右交换
		StringBuffer RL = new StringBuffer(R.toString() + L.toString());
		//逆置换
		StringBuffer substituteRL = new StringBuffer(); // 存储置换后的LR
        for (int i = 0; i < 64; ++i) {
            substituteRL.append(RL.charAt(IPReverse[i] - 1));
        }
        //删除多余字符
        if(group == 0) {
        	String temp = substituteRL.substring(56, 64);
        }
        //生成字符串明文或密文
        for (int i = 0; i < 8; ++i) {
            String temp = substituteRL.substring(i * 8, (i + 1) * 8);
            //加密
            if(encrypt_decrypt == 0) {
            	ciphertext.append((char) Integer.parseInt(temp, 2));
            	StringBuffer tempHex = new StringBuffer(Integer.toHexString(Integer.parseInt(temp, 2)));
            	while(tempHex.length() < 2) {
            		tempHex.insert(0, "0");
            	}
            	ciphertextHex.append(tempHex + " ");
            //解密
            }else {
            	if(group == 0 && 7 - i < Integer.parseInt(substituteRL.substring(56, 64), 2)) {
            		break;
            	}
            	plaintext.append((char) Integer.parseInt(temp, 2));
            }
            	
        }
        //END!
	}
	
	//Feistel轮函数
	public StringBuffer feistel(final StringBuffer R, final StringBuffer subKey ) {
		//E扩展
		StringBuffer RExtent = new StringBuffer(); // 存储扩展后的右边
        for (int i = 0; i < 48; ++i) {
            RExtent.append(R.charAt(E[i] - 1));
        }
		//异或运算
        for (int i = 0; i < 48; ++i) {
            RExtent.replace(i, i + 1, (RExtent.charAt(i) == subKey.charAt(i)) ? "0" :"1");
        }
		//S-Box
        StringBuffer SBoxString = new StringBuffer();
        for(int i = 0; i < 8; ++i) {
        	String SBoxInput = RExtent.substring(i * 6, (i + 1) * 6);
        	int row = Integer.parseInt(Character.toString(SBoxInput.charAt(0)) + SBoxInput.charAt(5), 2);
            int column = Integer.parseInt(SBoxInput.substring(1, 5), 2);
            
            StringBuffer SBoxOutput = new StringBuffer(Integer.toBinaryString(SBox[i][row * 16 + column]));
            while (SBoxOutput.length() < 4) {
                SBoxOutput.insert(0, 0);
            }
            SBoxString.append(SBoxOutput);
        }
		//P置换
        StringBuffer substituteSBoxString= new StringBuffer(); // 存储置换后的R
        for (int i = 0; i < 32; ++i) {
        	substituteSBoxString.append(SBoxString.charAt(P[i] - 1));
        }
        return substituteSBoxString;
	}
	
	//子秘钥生成
	public StringBuffer[] getSubKey() {
		//把key转换为二进制
		StringBuffer keyBinary = string2Binary(key);
		
		StringBuffer[] subKey = new StringBuffer[16]; // 存储子密钥
		//PC1置换
		StringBuffer C0 = new StringBuffer(); // 存储密钥左块
	    StringBuffer D0 = new StringBuffer(); // 存储密钥右块
	    for (int i = 0; i < 28; ++i) {
            C0.append(keyBinary.charAt(PC1[i] - 1));
            D0.append(keyBinary.charAt(PC1[i + 28] - 1));
        }
        
	    // LS循环16轮生成子密钥
        for (int i = 0; i < 16; ++i) {
            // 循环左移
        	char mTemp;
            mTemp = C0.charAt(0);
            C0.deleteCharAt(0);
            C0.append(mTemp);
            mTemp = D0.charAt(0);
            D0.deleteCharAt(0);
            D0.append(mTemp);
            if(i != 0 && i != 1 && i != 8 && i != 15) {
            	mTemp = C0.charAt(0);
                C0.deleteCharAt(0);
                C0.append(mTemp);
                mTemp = D0.charAt(0);
                D0.deleteCharAt(0);
                D0.append(mTemp);
            }
            // 把左右两块合并
            StringBuffer C0D0 = new StringBuffer(C0.toString() + D0.toString());
            
            // PC2置换
            // 根据PC2压缩C0D0，得到子密钥
            StringBuffer C0D0Temp = new StringBuffer();
            for (int j = 0; j < 48; ++j) {
                C0D0Temp.append(C0D0.charAt(PC2[j] - 1));
            }
            subKey[i] = C0D0Temp;
        }
        return subKey;
	}
	//返回密文
	public StringBuffer getCiphertext() {
		return ciphertext;
	}
	//返回密文十六进制
	public StringBuffer getCiphertextHex() {
		return ciphertextHex;
	}
	//返回明文
	public StringBuffer getPlaintext() {
		return plaintext;
	}
	//主函数
	public static void main(String[] args) throws Exception {
		//测试加密
		StringBuffer text = new StringBuffer("NiceToMeetYouDES");
		StringBuffer key = new StringBuffer("aaaaaaaa");
		int mode = 0;
		DES instance = new DES(text, key, mode);
		instance.start();
		System.out.println("DES-Algorithm\n");
		System.out.println("输入:" + text);
		System.out.println("模式:" + ((mode == 0) ? "加密" : "解密"));
		System.out.println("\n明文:" + instance.getPlaintext());
		System.out.println("密文:" + instance.getCiphertext());
		if(mode == 0)
			System.out.println("密文(十六进制):" + instance.getCiphertextHex());
		System.out.println("秘钥:" + key);
		System.out.println("");
		
		//测试解密
		text = instance.getCiphertext();
		key = new StringBuffer("````````");
		mode = 1;
		instance = new DES(text, key, mode);
		instance.start();
		System.out.println("输入:" + text);
		System.out.println("模式:" + ((mode == 0) ? "加密" : "解密"));
		System.out.println("\n明文:" + instance.getPlaintext());
		System.out.println("密文:" + instance.getCiphertext());
		if(mode == 0)
			System.out.println("密文(十六进制):" + instance.getCiphertextHex());
		System.out.println("秘钥:" + key);
		
	}
}

```



## 七、参考资料

[DES加密算法详解](https://blog.csdn.net/fenghao_5555/article/details/1589464)

[DES算法实现](https://blog.csdn.net/White_Idiot/article/details/67634872)

