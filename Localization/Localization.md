<br />
*본 문서에서는 글로벌 스포츠 개발실에서의 다국어 번역 구현 방법 및 적용 사례를 기술함.  
프로젝트마다 쓰는 플러그인, 언어 등 상황이 다를 수 있으므로, 각 상황에 맞게 적용 필요*  

<br />
# 개요

로컬라이징을 하기 위해 게임 내 모든 문자열을 그 문화권에 맞는 언어로 유저에게 보여주어야 함.  
번역은 해외 법인, 번역 API 또는 번역기 사용 등 각 프로젝트에 알맞는 방법이 필요하고, 아래는   
각 방법에 의해 번역된 데이터를 처리하는 방법임.

<br />
# 목차

1. 글로벌스포츠 개발실 프로젝트 현황
2. 설명을 위한 용어 정의
3. 구현 사례  
	3.1. 테이블 구성  
	3.2. 테이블 로딩 및 사용  
4. 기타(예외 사항들, 또는 구현 중 예상치 못한 사례)
5. 예시 코드 및 유니티패키지(유니티 5.3 기준)  

<br />
# 1. 글로벌스포츠 개발실 프로젝트 현황

- Unity, NGUI 사용(Unity 사용 여부, Unity 버전, NGUI 사용 여부 및 NGUI 버전은 큰 상관이 없음)
- [테이블을 xml로 관리하고, bytes로 변환해서 사용(게임 데이터, 번역 테이블 통합)](http://183.110.18.219/Developers/idea/issues/16)    
- 15개의 언어 지원 필요
- 각 언어에 대한 번역은 각 해외 법인을 통해 받음.

<br />
# 2. 설명을 위한 용어 정의


[이미지 준비중]
![localizationExample](/uploads/99d9bd6e35e9943a68d1c6e875700fba/localizationExample.PNG)

*<사진 1 - '오늘의 스핀'> 게임 내 한 화면으로, 설명을 위해 UI의 위치를 약간 수정 하였음*  
<br />

### 어떤 게임이라도, 번역을 목적으로 문자열을 나눈다면 다음 3가지 종류로 분류할 수 있고,  이를 각각 *UI String, System String, TableData String* 으로 칭해서 설명함.  
<br />

#### 1. *System String*  
내부적인 시스템(게임 로직)에 의해서 유저에게 보여주어야 할 문자열이 달라지는 값들.    
> <사진 1>의 (1)에서, 코인이 부족했기 때문에 해당 문자열을 출력하지만,  
> 코인이 충분하다면, 내부 시스템(로직)에 의해 스핀의 결과("무엇을 얻었습니다") 등이 출력되어야 할 것임.  
> 이와 같이, 게임 내부 로직에 따라 보여주어야 할 결과 값이 달라질 수 있는 문자열을 System String 이라 칭함.

#### 2. *UI String*  
UI와 함께 고정되어 유저에게 나타나고, 가변 가능성이 굉장히 적은 문자열.    
> <사진1>의 (2)에서'오늘의 스핀'이라는 문자열은 해당 컨텐츠의 이름이 다른 이름으로 바뀌지 않는 한, 불변의 문자열임.    
> 해당 컨텐츠에 대한 설명도, 수정 되지 않는다면 유저에게 보여주어야 할 문자열이 바뀌지 않음.  
> 이러한 문자열들을 UI String 이라 칭함.

#### 3. *TableData String*  
게임 데이터를 관리하는 테이블의 내용 중, 번역이 필요한 항목들  
> <사진 1>의 (3) 아이템들은 '오늘의 스핀'이라는 컨텐츠의   보상 리스트로, DB 또는 로컬 DB 등에 저장되어 사용될 부분임.  
> 오늘의 스핀 관련 테이블에는 '20레벨 감독뽑기권', '1' / '슈퍼스타 선수팩', '1-3' ... 등으로 보상이 정해져 있을 것이며,  
> 이 데이터 중, '아이템 이름'은 각 언어에 따라 번역되어야 할 부분임. 이를 TableData String이라 칭함.


<br />
# 3. 구현 사례  
## 3.1. 테이블 구성  
### 3.1.1. 테이블 구성 모습

#### 1. [*System String*](http://183.110.18.219/Developers/idea/issues/17#1-system-string)  

[이미지 준비중]
![LocalizationSystemEX](/uploads/f1df66ea99de62d6419256bb9ffa6480/LocalizationSystemEX.PNG)  

*<사진2 - 'LocalizationSystem.xml'> [*System String*](http://183.110.18.219/Developers/idea/issues/17#1-system-string)을 사용하기 위해 구성한 테이블*  
> 각 로직에 따라 인덱스를 호출할 수 있게 인덱스와 언어별 번역 결과 값을 컬럼으로 테이블을 구성.   
> ex : (인덱스(int), 언어1 번역값, 언어2 번역값, ...)  
> <사진1>의 좌측상단에 있는 팝업에서는 <사진2>에 존재하는 인덱스 4197을 호출하여,   
> "코인이 부족하여 스핀을 이용할 수 없습니다." 값을 얻고 사용함.  
<br />   


#### 2. [*UI String*](http://183.110.18.219/Developers/idea/issues/17#2-ui-string)  

[이미지 준비중]
![LocalizationUIEX](/uploads/473c595b6ad1cf3a300764f4ed2d132d/LocalizationUIEX.PNG)   

*<사진3 - 'LocalizationUI.xml'> [*UI String*](http://183.110.18.219/Developers/idea/issues/17#2-ui-string)을 사용하기 위해 구성한 테이블*  

> 초기 구성 상황에서는 [*System String*](http://183.110.18.219/Developers/idea/issues/17#1-system-string)과 같이, 인덱스와 결과값으로 구성되어 있었으나,  
> 유니티 에디터에서 편집하기 용이하게 하기 위해, 씬의 의미와 세부적인 의미를 갖는 글자조합으로 키를 변경  
> ex : (인덱스(int - 사용안함), 인덱스(문자열), 언어1 번역값, 언어2 번역값, ...)  
> <사진1> 우측 상단에 보이는 "오늘의 스핀"이라는 타이틀은 "오늘의스핀1_오늘의스핀"이라는 키의 결과값이고,  
> 이는 <사진3>의 316번 값("오늘의 스핀1_오늘의스핀")에 매칭됨.  
<br />  

#### 3. [*TableData String*](http://183.110.18.219/Developers/idea/issues/17#3-tabledata-string)   

[이미지 준비중]
![LocalizationForTableDataEX](/uploads/02c2967a0f3d3473314a42af72ec01c4/LocalizationForTableDataEX.PNG)  

*<사진4 - 'TodaySpin.xml'> 오늘의 스핀이라는 컨텐츠의 정보가 존재하는 테이블*  

> 이 테이블에는 해당 컨텐츠의 보상 아이템이 존재함. 클라 또는 서버에서 이 테이블의 정보를 이용해 내부 로직을 전개하기도 함.  
> 컨텐츠의 게임 데이터이기때문에, 번역이 필요한 값도 존재하고, 번역이 필요치 않은 값도 존재하는데,   
> 이를 효율적으로 사용하기 위해, [*TableList*](http://183.110.18.219/Developers/idea/issues/17#etc-tablelist)를 아래에서 설명함.

[이미지 준비중]
![LocalizationForTableDataEX2](/uploads/7a0c3f4f3c131f6e8ec40057b97ec1d1/LocalizationForTableDataEX2.PNG)

*<사진 5 - 'LocalizationForTableData.xml'> 여러 게임 데이터가 존재하는 테이블의 내용들 중, 번역이 필요한 값을 인덱스로 하여, 각 언어별 번역 결과 값을 저장해놓는 테이블*

> <사진5>의 13752번을 예시로 들면, TodaySpin.xml(또는 bytes)에서 Svr_TodaySpin 시트에 있는 Name 컬럼 값을 번역할 예정인데,   
> 키가 '고급 레드 뽑기권' 이고, 한국어로는 '고급 레드 뽑기권' / 영어로는 Primium Red Ticket이 되는 식임.  
> ex : (인덱스(int - 사용안함), 테이블명, 시트명, 컬럼명, 해당컬럼의 번역 키 값, 언어1 번역값, 언어2 번역값, ...)

#### etc. TableList  

[이미지 준비중]
![LocalizationForTableListEX](/uploads/7d80eb910ecb352c64d63f8dbcb71375/LocalizationForTableListEX.PNG)  
*<사진6 - 'LocalizationForTableList.xml'> 바로 위의 [*TableData String*](http://183.110.18.219/Developers/idea/issues/17#3-tabledata-string-1)에서 미리 참고하기 위해 만든 테이블*
> <사진4>에서 모든 컬럼을 번역할 필요는 없음. Item Code나 Count_min, max 등 게임 데이터에만 사용되는 값들은 번역 필요가 없고, 아이템 이름과 같이 유저에게 보여줄 수 있는 값은 번역이 필요한 값임.  
> 번역의 필요 여부에 따라 필요한 테이블, 필요한 시트, 필요한 컬럼들을 미리 정의해두고, 최초 게임 실행시 로딩할 때, 이 테이블을 가장 먼저 참고하여 나머지 테이블들을 로딩하게 함.   

### 3.1.2. 테이블 구성 이유

- 작업 편의
 - UI는 에디터에서 씬이나 프리팹에 부착
 - 로직 달라지는 텍스트만 코드에 추가  
 - 텍스트의 종류에 따라 기획자가 테이블에서 찾기 편함  
- 게임 UI에 적합
 - 이미지 뿐만 아니라 텍스트도 User Interface 인데, 프리팹이나 씬에 고정되어 있음
 - 로직은 System으로 런타임 로딩  
 - 이 종류를 벗어나는 UI는 존재할 수 없음(더이상의 예외는 생각하지 않아도 됨)
- UI.xml의 경우 키가 string이라 int인 경우보다 속도는 느리겠지만, 작업할때 "1234"가 적혀있는 경우와 "오늘의스핀1_오늘의스핀" 이 적혀있는 경우에 알 수 있는 의미부터가 다름.
- 게임 데이터 테이블의 경우, 굳이 번역으로의 확장성을 고려하지 않고, 게임 데이터만으로 테이블을 구성해도 됨
 - TableData, TableList를 사용하기 때문

## 3.2. 테이블 로딩 및 사용  

[이미지 준비중]
![localizationFlow](/uploads/4e82aa2ccbdfaf1502238320fb7ba29d/localizationFlow.PNG)  
*<사진 7 - '테이블 로딩, 사용'>*  

(ㄱ) 
- 번역이 필요한 테이블들은 로딩하자마자 번역될 수 있게 기억해놓아야 함.  
[TableList](http://183.110.18.219/Developers/idea/issues/17#etc-tablelist)를 통해 번역이 필요한 테이블/시트/컬럼을 알아두고, 이 정보를 이용해서  <사진 5>의 LocalizationForTableData.xml를 로딩해놓음.  
<br />

(ㄴ) 
- [*System String*](http://183.110.18.219/Developers/idea/issues/17#1-system-string) : [LocalizationSystem.xml](http://183.110.18.219/Developers/idea/issues/17#1-system-string-1)을 로딩 후, 인덱스(int)와 해당 클라이언트 언어에 맞는 컬럼 결과 값들을 System Map(Dictionary)에 넣어놓음.
- [*UI String*](http://183.110.18.219/Developers/idea/issues/17#2-ui-string) : [LocalizationUI.xml](http://183.110.18.219/Developers/idea/issues/17#2-ui-string)을 로딩 후, 인덱스(string)와 해당 클라이언트 언어에 맞는 컬럼 결과 값들을 UI Map(Dictionary)에 넣어놓음.  
UI를 사용하는 씬이나 프리팹에는 [LocalizationUI.cs](http://183.110.18.219/snippets/10)를 UILabel에 컴포넌트로 부착시켜, Monobehaviour.Start()에서 바로 호출되게 해놓음.
- 이를 제외한 게임 데이터 테이블 : (ㄱ)에서 기억해놓았던 '번역이 필요한 컬럼'들만 번역하고, 나머지는 번역하지 않은 상태로 로딩함.   
<br />


(ㄷ)  
-  씬이나 프리팹에 고정되어 있는 기존 키값이, (ㄴ)에서 로딩해놓은 UI map을 이용해서 결과값으로 변환됨.
<br />


(ㄹ)  
- (ㄴ)에서 로딩해놓은 System Map 에서 키만 찾아서 런타임에 사용함.  

<br />
# 4. 기타(예외 사항들, 또는 구현 중 예상치 못한 사례)  
- 우리 프로젝트에서는 테이블명_Startup의 이름을 가진 테이블들이 존재하는데,   
이는 애셋번들을 다운받지 않은 초기 상태에서 번역문을 볼 수 있게 apk에 포함되는 리소스임.  
- 애셋번들로 만들지 않고, 기본 디렉토리에 변환후 저장.  

- 번역 테이블을 외국 법인에 맡기는데 일부 국가에서 줄바꿈이 다르게 나와서 "\u2028", "\u2029" 등을 " "로 대체.  
	
- 일본어 등에서 format string 미동작(전각문자).

		"\uff5b" -> "\u007b" ( { )   


		"\uff5d" -> "\u007d" ( } )    


		"\uff1a" -> "\u003a" ( : )   

- 아랍어의 경우에 다른언어와 섞였을 때는, 양방향 언어이기 때문에 주 언어를 판별. are_if 컬럼을 참고해서 아랍어 변환을 할지 결정.

	- 외국 법인에서 번역이 올 때마다 글자를 보고 판단하거나, 구글 스프레드 시트에서 함수 사용.

		- *=DETECTLANGUAGE(입력 텍스트_또는_범위)*

	- [NBidi](https://github.com/eyaldar/NBidi)를 이용해서 재조립한 후, 후처리로 NGUI 역 줄바꿈, 컬러태그 등을 계산.

	- 기본 우측 정렬.
<br />


<br />
# 5. 예시 코드 및 유니티패키지(유니티 5.3 기준)    
준비중....  
<br />
