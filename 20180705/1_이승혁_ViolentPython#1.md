# Violent Python #1

### **Violent Python**

7 July 2018 이승혁

[해커의 언어, 치명적 파이썬](https://book.naver.com/bookdb/book_detail.nhn?bid=7145997)이라는 책을 보고 실습 한 내용입니다. 본 내용으로 들어가기 전에 메타데이터란 무엇인지 알아보도록 하겠습니다.

------

#### Metadata

**메타데이터**(metadata)는 데이터(data)에 대한 데이터이다. 이렇게 흔히들 간단히 정의하지만 엄격하게는, Karen Coyle에 의하면 "어떤 목적을 가지고 만들어진 데이터 (Constructed data with a purpose)"라고도 정의한다. 가령 도서관에서 사용하는 서지기술용으로 만든 것이 그 대표적인 예이다. 지금은 온톨로지의 등장과 함께 기계가 읽고 이해할 수 있는 (Machine Actionable)한 형태의 메타데이터가 많이 사용되고 있다.

데이터에 관한 구조화된 데이터로, 다른 데이터를 설명해 주는 데이터이다. 대량의 정보 가운데에서 찾고 있는 정보를 효율적으로 찾아내서 이용하기 위해 일정한 규칙에 따라 콘텐츠에 대하여 부여되는 데이터이다. 어떤 데이터 즉 구조화된 정보를 분석, 분류하고 부가적 정보를 추가하기 위해 그 데이터 뒤에 함께 따라가는 정보를 말한다.

이를테면, 디지털 카메라에서는 사진을 찍어 기록할 때마다 카메라 자체의 정보와 촬영 당시의 시간, 노출, 플래시 사용 여부, 해상도, 사진 크기 등의 사진 정보를 화상 데이터와 같이 저장하게 되어 있다. 이러한 데이터를 분석하여 이용하면 그 뒤에 사진을 적절하게 정리하거나 다시 가공할 때에 아주 유용하게 쓸 수 있는 정보가 된다. GPS기능을 사용하여 위치 정보까지 사진의 메타데이터에 입력할 수도 있는데, 이를 이용하면 사진이 어디에서 촬영되었는지를 쉽게 알 수 있고, 이로써 다시 다른 지역 정보를 검색하거나 같은 지역에서 찍은 다른 사진을 검색하게 하는 검색성을 향상시킬 수 있다.

메타데이터는 메타데이터가 부여될 때와 쓰일 때의 문맥 정보를 구조화시켜 그 활용도를 확대시키는 역할을 한다. 웹 2.0이나 온톨로지(Ontology)의 분야에서 구조화된 메타데이터는 매우 유용하다.

[출처](https://ko.wikipedia.org/wiki/%EB%A9%94%ED%83%80%EB%8D%B0%EC%9D%B4%ED%84%B0)



------

#### pyPdf

pyPdf란 PDF문서를 관리하는 유틸리티이며 문서 정보를 추출, 분할, 통합, 잘라내기, 암호화, 복호화 하는 기능을 제공합니다. 하지만 pyPdf는 파이썬의 기본 내장 모듈이 아니기 때문에 추가적으로 설치를 해야합니다. 설치 명령어는 다음과 같습니다.

> pip install pyPdf  

pyPdf가 설치되면 파이썬으로 PDF분석 프로그램을 만들 수 있게 됩니다.

##### Codes

```python
import pyPdf
import optparse
from pyPdf import PdfFileReader

def printMeta(filename):
	pdfFile = PdfFileReader(file(filename, 'rb'))
	docInfo = pdfFile.getDocumentInfo()
	print "[*] PDF MetaData For : " + str(filename)
	for metaItem in docInfo:
		print "[+] " + metaItem + ":" + docInfo[metaItem]

def main():
	parser = optparse.OptionParser('usage %prog -F <PDF file name>')
	parser.add_option("-F", dest = "filename", type = "string", help = "specify PDF file name.")
	(options, args) = parser.parse_args()
	fileName = options.filename
	if fileName == None:
		print parser.usage
		exit(0)
	else:
		printMeta(fileName)

if __name__ == "__main__":
	main()
```

> .getDocumentInfo() : 메타데이터 요소의 설명과 값이 포함된 튜플의 배열을 반환합니다.



#### Result

```
❯ python pypdf.py -F Hello_SuNiNaTaS.pdf
[*] PDF MetaData For : Hello_SuNiNaTaS.pdf
[+] /CreationDate:D:20160526050544+09'00'
[+] /Author:capcorps
[+] /Producer:Hancom PDF 1.3.0.480
[+] /Creator:Hancom PDF 1.3.0.480
[+] /PDFVersion:1.4
[+] /ModDate:D:20160526070004+09'00'
[+] /Title:find the key
```

파일을 분석 해보면 만든 날짜, 수정한 날짜, 만든 사람, 쓰인 프로그램, 제목 등을 알 수 있습니다.

[ PDF 파일 출처(로그인 필요)](http://suninatas.com/Part_one/web31/web31.asp)



------

#### Exif

**교환 이미지 파일 형식** (Exif; EXchangable Image File format)은 디지털 카메라에서 이용되는 이미지 파일 포맷이다. 이 데이터는 JPEG, TIFF 6.0과 RIFF, WAV 파일 포맷에서 이용되며 사진에 대한 정보를 포함하는 메타데이터를 추가한다. Exif는 JPEG 2000, PNG나 GIF 파일에서는 지원하지 않는다.

Exif 메타데이터는 카메라 제조사, 카메라 모델, 회전 방향, 날짜와 시간, 색 공간, 초점 거리, 플래시, ISO 속도, 조리개, 셔터 속도, gps 등의 정보를 제공한다.

[출처](https://ko.wikipedia.org/wiki/%EA%B5%90%ED%99%98_%EC%9D%B4%EB%AF%B8%EC%A7%80_%ED%8C%8C%EC%9D%BC_%ED%98%95%EC%8B%9D)



------

#### Exiftool

Exiftool은 Exif 태그를 분석하기 위한 툴로 Exif 태그를 분석하여 카메라 모델의 이름, 위도와 경도 등 다양한 정보를 수집할 수 있는 툴입니다.

[Download](https://www.sno.phy.queensu.ca/~phil/exiftool/)

##### Example

<img width="537" alt="screen shot 2018-07-07 at 8 35 05 pm" src="https://user-images.githubusercontent.com/40850499/42410487-51f08baa-8225-11e8-9932-a77f0325f40c.png">

위 예시는 편의상 뒷부분을 생략 한 것으로 실제 결과는 다소 길 수 있습니다.



------

### **suninatas.com**

이번에 풀었던 워게임 사이트 입니다. [Link](http://suninatas.com/)

#### Question #15(Forensics #1)

15번 문제는 파일에 숨어있는 인증 키를 찾는 문제입니다.

다음과 같은 간단한 방법으로 풀 수 있었습니다.

```
❯ exiftool diary.mp3
...
Title                           : ´ÙÀÌ¾î¸®
Album                           : ´ÙÀÌ¾î¸®
Year                            : 2011
Track                           : 1
User Defined URL                : http://ihoneydew.com/
Picture MIME Type               : image/jpeg
Picture Type                    : Front Cover
Picture Description             :
Picture                         : (Binary data 13289 bytes, use -b option to extract)
Comment (kor)                   : kpop.321.cn
Genre                           : kpop.321.cn
Conductor                       : ####################
Artist                          : ³ªºñ
Comment                         : kpop.321.cn
Audio Bitrate                   : 234 kbps
Date/Time Original              : 2011
Duration                        : 0:03:31 (approx)
```

일단 Exiftool을 사용해서 열어보면 Conductor 부분에 바로  인증 키[^#15 key]가 노출되어있는 것을 볼 수 있습니다.

------

#### Question #18(Forensics #2)

18번 문제는 주어진 숫자들이 무엇을 뜻하는지 알아내는 문제입니다.

아스키코드로 추정되어 프로그램을 짜서 풀었습니다.



##### Codes

```python
import pybase64

inp = "86 71 57 107 89 88 107 103 97 88 77 103 89 83 66 110 98 50 57 107 73 71 82 104 101 83 52 103 86 71 104 108 73 69 70 49 100 71 104 76 90 88 107 103 97 88 77 103 86 109 86 121 101 86 90 108 99 110 108 85 98 50 53 110 86 71 57 117 90 48 100 49 99 109 107 104"

inp = map(int, inp.split())
length = len(inp)
mystr = ""

for i in range(length):
	mystr += chr(inp[i])

print mystr
print ""

mystr = pybase64.b64decode(mystr)
print mystr
```



##### Results

> **Result #1 Convert ord to chr** 
>
> VG9kYXkgaXMgYSBnb29kIGRheS4gVGhlIEF1dGhLZXkgaXMgVmVyeVZlcnlUb25nVG9uZ0d1cmkh

일단 변환을 하니 BASE64로 추정되는 암호문이 나오게 되어 pybase64모듈을 이용해 복호화 해보았습니다.



> **Result #2 Decryption**
>
> ##### ##### ## # #### ###. ### ####### ## ##################### #####################

복호화 후의 결과입니다. 평문[^#18 key]이 나오는 것을 볼 수 있습니다.

------

#### Question #19(Forensics #3)

2진수로 이루어진 숫자들이 주어집니다. 프로그램을 만들어 2진수를 알아보기 쉽게 바꿔보았습니다.

##### Codes

```python
inp = "010011100101011001000011010101000100011001000100010101100010000001001011010001100010000001001010010011000100010101011010010001010101001001001011010100100100101000100000010100100100010101010101001000000100101101000110010101010101001001010000001000000101101001001010001000000101001000100000010110000100011001000110010101010010000001010101010100100101000000100000010100100100010101010101001000000101001001001100010010110101100101000010010101100101000000100000010110100100101000100000010001110100001101010010010110100101010101010100010010110101011101011010010010100100110101010110010010010101000001011001010100100100100101010101"
length = len(inp)
print length
mylist = []
mystr = ""

for i in range(length):
	if i % 8 == 0 and i != 0:
		mylist.append(int(inp[i - 8 : i], 2))
	elif i == length - 1:
		mylist.append(int(inp[i - 7 : i + 1], 2))


print len(mylist)

for j in range(26):
	cur = ""
	print ""
	for i in range(len(mylist)):
		if mylist[i] == 32:
			cur += " "
		else:
			cur += chr((mylist[i] - 65 + j) % 26 + 65)

	print "-" * 50
	print "#%d : %s" % (j + 1, cur)
	print "-" * 50
```



##### Results

```
------------------------------------------------------------------------------------
#1 : NVCTFDV KF JLEZERKRJ REU KFURP ZJ R XFFU URP REU RLKYBVP ZJ GCRZUTKWZJMVIPYRIU
------------------------------------------------------------------------------------
...
------------------------------------------------------------------------------------
#10 : ####### ## ######### ### ##### ## # #### ### ### ####### ## ##################
------------------------------------------------------------------------------------
```

이진수들을 문자로 바꿔보니 1번 문자열이 나오게 되었고 형태를 보니 시저 암호일 것 같아 시저 암호 복호화 코드를 추가하여 10번 문자열(평문)[^#19 key]을 얻어내었습니다. 

------

#### Question #21(Forensics #4)

첨부된 사진을 다운받아서 헥스 에디터로 열어보았습니다. 일반 사진 파일에 비해 파일의 내용이 긴 것이 확인되었고, 헤더 부분이 한 개 이상 존재했기 때문에 Exif 부분을 기준으로 파일을 분리했습니다. 분리할 수 있는 파일의 갯수는 더 많았지만 두 장을 분리하는 것으로 인증 키를 알아낼 수 있었기 때문에 두 장 까지만 분리했습니다.

##### Hex Editor

<img width="458" alt="screen shot 2018-07-05 at 5 35 13 am" src="https://user-images.githubusercontent.com/40850499/42410516-dcc107dc-8225-11e8-82d4-c3a8d2393108.png">
<img width="459" alt="screen shot 2018-07-05 at 5 35 49 am" src="https://user-images.githubusercontent.com/40850499/42410517-dcebd0a2-8225-11e8-88b4-d042748e2a4d.png">
<img width="459" alt="screen shot 2018-07-05 at 5 35 59 am" src="https://user-images.githubusercontent.com/40850499/42410518-dd130f64-8225-11e8-97ef-7fc10b35cdaa.png">

##### Original File

![monitor](https://user-images.githubusercontent.com/40850499/42410430-58572716-8224-11e8-801c-fbbebdc172fe.jpg)

##### Seperated File #1

![m2](https://user-images.githubusercontent.com/40850499/42410441-8b443e3e-8224-11e8-8902-542dc920fd5a.jpg)

##### Seperated File #2 

![m3](https://user-images.githubusercontent.com/40850499/42410446-98950c08-8224-11e8-937a-ef73d4110d60.jpg)

이렇게 분리 한 결과들을 조합하여 키[^#21 key]를 찾을 수 있었습니다.



------

#### Question #26(Forensics #5)

[빈도분석법](http://crypto.interactive-maths.com/frequency-analysis-breaking-the-code.html)으로 복호화하는 문제이며 공백 문자와 특수 문자는 생략되어있습니다.

##### Trigraph

<img width="484" alt="screen shot 2018-07-05 at 5 43 38 am" src="https://user-images.githubusercontent.com/40850499/42410530-0566f070-8226-11e8-8d23-3831f7a1c95e.png">

시작은 가장 많이 나오는 Trigraph(같이 사용 된 세 알파벳)을 찾는 것입니다. 위 이미지와 같이 영어에서 가장 많이 나오는 trigraph는 the이지만 26번 문제의 암호문에서는 bpn이기 때문에 b는 t로, p는 h로, n은 e로 치환합니다. 

<img width="800" alt="screen shot 2018-07-05 at 5 46 42 am" src="https://user-images.githubusercontent.com/40850499/42410533-05d8e554-8226-11e8-9b9c-f4df73066884.png">

위 사진은 주어진 암호문의 Trigraph 빈도분석을 이용하여 문자들을 치환 한 모습입니다. 대문자는 수정되지 않은 알파벳, 소문자는 추정을 통해 치환 된 알파벳입니다. 

##### Replace Example

| 암호문                                                       | 추정                   | 알아낸 알파벳       |
| ------------------------------------------------------------ | :--------------------- | ------------------- |
| <img width="57" alt="screen shot 2018-07-05 at 5 49 05 am" src="https://user-images.githubusercontent.com/40850499/42410534-05ffb09e-8226-11e8-8a65-704f54b5b82a.png"> | **she has**            | **C  → a**          |
| <img width="55" alt="screen shot 2018-07-05 at 5 50 37 am" src="https://user-images.githubusercontent.com/40850499/42410535-06277a66-8226-11e8-9f58-e2f380985af2.png"> | **system**             | **K  → y, Q  → m**  |
| <img width="67" alt="screen shot 2018-07-05 at 5 53 02 am" src="https://user-images.githubusercontent.com/40850499/42410538-06b8223c-8226-11e8-8214-b3b9def1889b.png"> | **females**            | **O  → f**          |
| <img width="51" alt="screen shot 2018-07-05 at 5 52 13 am" src="https://user-images.githubusercontent.com/40850499/42410536-0651fd2c-8226-11e8-9ed0-162c4a87e2be.png"> | **names** or **games** | **D  → n **or **g** |
| <img width="29" alt="screen shot 2018-07-05 at 6 03 08 am" src="https://user-images.githubusercontent.com/40850499/42410540-0705d82e-8226-11e8-8e86-a90fe3abff3c.png"> | **first**              | **I  → r**          |
| <img width="52" alt="screen shot 2018-07-05 at 6 00 08 am" src="https://user-images.githubusercontent.com/40850499/42410539-06de5c90-8226-11e8-91f6-9260730b05b8.png"> | **highly**             | **Z  → i**          |
| <img width="68" alt="screen shot 2018-07-05 at 6 04 03 am" src="https://user-images.githubusercontent.com/40850499/42410541-072c47a2-8226-11e8-9f7a-4a554f4bd876.png"> | **her entire**         | **G  → n**          |
| <img width="54" alt="screen shot 2018-07-05 at 6 04 42 am" src="https://user-images.githubusercontent.com/40850499/42410542-075461f6-8226-11e8-8b1a-010ea28b913a.png"> | **career**             | **M  → c**          |

위 표와 같이 알파벳들을 추정을 통해 치환하여 복호화 할 수 있습니다.

##### Result

<img width="804" alt="screen shot 2018-07-07 at 9 03 47 pm" src="https://user-images.githubusercontent.com/40850499/42410721-74895c24-8229-11e8-9199-a54539750b5e.png">

Auth Key[^#26 key]

------

하단에는 Auth Key가 있습니다. 문제를 푸실 분은 주의하여 보시기 바랍니다.







[^#15 key]: GoodJobMetaTagSearch
[^#18 key]: Today is a good day. The AuthKey is VeryVeryTongTongGuri!
[^#19 key]: WELCOME TO SUNINATAS AND TODAY IS A GOOD DAY AND AUTHKEY IS PLAIDCTFISVERYHARD
[^#21 key]: H4CC3R_IN_MIDD33_4TT4CK
[^#26 key]: kimyuna

