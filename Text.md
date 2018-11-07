Rの練習 181106 Ryosuke TAJIMA 
==============  
# R言語の基本的な動作
```R  
x<-1
y<-5
z<-rbind(x,y) #行で結合
v<-cbind(x,y) #列で結合
a<-c(1,3,5,7,9) #数字列
b<-c(2,4,6,8,10) #数字列
c<-rbind(a,b) #行で結合
d<-cbind(a,b) #列で結合
e<-a+b #行列要素の足し算
f<-a*b #行列要素の掛け算
```  

# データの取り扱い  
  
## データの読み込み  
  
### ディレクトリの設定  
```R  
getwd()  
```  
### ディレクトリの変更  
-  File > Change Directory ...  
  
###  データの読み込み
```R  
d1<-read.table("Test1.txt", header=T)
d1  
head(d1)
```  
  
## 読み込んだデータのチェック
### アウトライン
```R  
nrow(d1) # total row number  
ncol(d1) # total column number  
dim(d1) # row and column numbers  
```  
  
### 列のクラスを調べる
```R  
class(d1$Stage)  
class(d1$Rep)  
class(d1$SDW)  
```  
  
## データの一部を抽出する  
```R  
A<-d1[1,1]
A<-d1[10,1]  
A<-d1[1:3,4:7]  
A<-d1[3,]  
A<-d1[,8]  
A<-d1[1:4,]  
A<-d1[,1:4]  
```  
  
```R  
Stage<-d1$Stage  
Stage<-d1$stage
```  
  
## データの一部を切り出す  
### 基本的な使い方  
```R  
sd5<-subset(d1,d1$Stage=="5week")  
sd8<-subset(d1,d1$Stage=="8week")  
```  
  
##データが多い場合はdplyrを使うと早い  
```R  
library(dplyr)
data5week <-d1 %>%
    dplyr::filter(d1$Stage == "5week")
```  

##データの換算値の付け足しも簡単  
```R  
data2 <-d1 %>%
    dplyr::mutate(
        SRratio=d1$SDW/d1$RDW,
        SP=d1$SDW*d1$SPC/100*1000,
        SRL=d1$RLD/d1$RDW
        )
```  
  
# データのアウトラインを図で見る  
## 相関図  
```R  
plot(d1[,5:8])  
plot(d1[,6], d1[,7]) # Selected  
plot(d1$SDW, d1$RDW) # Selected  
#Selected two plot  
par(mfrow=c(1,2)) # c(row, column)  
plot(d1[,6], d1[,7])  
plot(d1$SDW, d1$RDW)  
```  

## corrplotを使って  
```R  
cor_num<-cor(d1[,5:8])
library(corrplot)
corrplot.mixed(cor_num,lower="number",upper='ellipse')
```  
  
## 箱ひげ図
```R  
boxplot(d1$SDW)  
boxplot(sd5$SDW, sd8$SDW)  
boxplot(sd5$SDW~sd5$Limitation)  
boxplot(sd5$SDW~sd5$Inoculation)  
```  
  
# 統計解析  
## Fisher 1966  
- If the design of an experiment is faulty, any method of interpretation which makes it out to be decisive must be faulty, too.  
  
## Fisherの3原則  
- 反復：実験ごとのばらつきを考慮するため同条件で複数回実験する  
- 局所管理：反復(ブロック)内では解析する要因以外の制御可能な要因は一定にする  
- 無作為化：制御できない要因はランダムになるようにする(系統誤差をなくす)  

## 分散分析  
### 一元配置の分散分析  
```R  
result<-aov(SDW~Limitation, sd5)  
summary(result)  
```  
  
### 二元配置の分散分析  
```R  
result<-aov(SDW~Limitation+Inoculation+Limitation*Inoculation, sd5)  
summary(result)  
result<-aov(SDW~Limitation*Inoculation, sd5) #short version  
summary(result)  
```  
  
### 乱塊法
```R  
result<-aov(SDW~Rep+Limitation*Inoculation, sd5)
summary(result)  
```  
  
### 分割区法
```R  
result<-aov(SDW~Rep+Limitation+Error(Rep/Limitation)+Inoculation+Limitation*Inoculation, sd5)   
summary(result)  
```  
  
### 二方分割法
```R  
result<-aov(SDW~Rep+Limitation+Inoculation+Error(Rep/(Limitation*Inoculation))+Limitation*Inoculation, sd5)   
summary(result)  
```  
  
## 多重比較
### TukeyHSD  
```R  
d2<-read.table("Test2.txt", header=T)
result<-aov(SDW~Treatment, d2)  
summary(result)  
TukeyHSD(result)  
```  
  
### multcomp利用，Tukey  
```R  
result<-aov(SDW~Treatment, d2)  
Tukey<-glht(result, linfct=mcp(Treatment="Tukey"))  
summary(Tukey)  
cld(Tukey, level = 0.05, decreasing = TRUE) # attach alphabet  
```  
  
### multcomp利用，Dunnett  
```R  
result<-aov(SDW~Treatment, d2)  
Dunnett<-glht(result, linfct=mcp(Treatment="Dunnett"))  
summary(Dunnett)  
```  
  
## 相関  
```R  
cor(sd5$SDW, sd5$SPC)  
allcor<-cor(sd5[,5:8])
allcor
```  
  
## 回帰分析
```R  
result<-lm(SDW~SPC, sd5)  
summary(result)  
```  
  
## 重回帰分析
### 多重共線性のチェック
```R  
allcor  
```  
  
### 分析
```R  
result<-lm(SDW~SPC+RLD, sd5)  
summary(result)  
```  

---  
### おしまい．お疲れさまでした．