-期中作业
=========
# (数据选取为绵阳市某时段人流情况)
# 摘要
# 一、数据处理
## 数据准备
```
library(RgoogleMaps)
library(googleVis)
library(ggplot2)
library(maptools)
library(maps)
library(mapproj)
library(mapdata)
# 导入数据
mydata<-read.table(header=T,file="df3.csv",sep=",")
# 获取起点坐标并使用k-means算法聚类
origdata<-mydata[,5:6]
origdata.kmeans<-kmeans(origdata,3)
lng <- mydata[,5]
lat <- mydata[,6]
# 获取终点坐标并使用k-means算法聚类
descdata<-mydata[,7:8]
descdata.kmeans<-kmeans(descdata,3)
lng_e<-mydata[,7]
lat_e<-mydata[,8]
# 设置背景
MapBackground(lat,lng, destfile ="bckg",maptype ="terrain")
doubs.map <- ReadMapTile(destfile = "bckg")
# 绘制地图

```
## 起点聚类
```
# 绘制起点坐标点
PlotOnStaticMap(doubs.map,lat,lng,cex = 1.0,add=TRUE,col = origdata.kmeans$cluster,pch = 19)
```
![起点聚类.png](https://github.com/shengunxiansen/Test/raw/master/起点聚类.png)

## 终点聚类
```
# 绘制终点坐标
PlotOnStaticMap(doubs.map,lat_e,lng_e,cex = 1.0,col = descdata.kmeans$cluster,pch = 19)
```
![终点聚类.png](https://github.com/shengunxiansen/Test/raw/master/终点聚类.png)

## OD线聚类
```
library(leaflet)
library(maps)
mydata <- read.csv("df3.csv",header = T)
ViewData<-data.frame(mydata)
#对线的中心点位置进行Kmeans聚类
CenterOfLineLng<-(ViewData$Lng_e+ViewData$Lng)/2
CenterOfLineLat<-(ViewData$Lat_e+ViewData$Lat)/2
CenterLo<-cbind(CenterOfLineLng,CenterOfLineLat)
KOfCenter<-kmeans(CenterLo,5)
ViewData<-data.frame(ViewData,KOfCenter$cluster)
#给每个类设置线条颜色
pt <- cut(ViewData$KOfCenter.cluster,breaks = c(0,1,2,3,4,5),labels = c("#00FF00", "#FF0000", "#FFFF00","#0000FF","#000000"))
ViewData$State2 <- pt
#画地图
map<-leaflet()
map<- fitBounds(map,104.61,31.42,104.78,31.52)
map<-addProviderTiles(map,"OpenStreetMap.Mapnik")
#循环完成在地图上线的描绘
for (i in 1:nrow(ViewData)) {
  map <- addPolylines(map,lng=c(ViewData[i,'Lng'],ViewData[i,'Lng_e']), lat = c(ViewData[i,'Lat'],ViewData[i,'Lat_e']), 
                      color = as.character(ViewData[i, c('State2')])
  )
}
map<-addCircleMarkers(map,lng = ViewData$Lng_e,lat = ViewData$Lat_e,color = 'red',radius = ViewData$count/5)
map
```
![OD线聚类.png](https://github.com/shengunxiansen/SJWJQZZY/raw/patch-2/OD线聚类.png)

# 二、数据样本
本次数据分析使用的数据如下的395条数据。
    kml_e	kml_s	Lng	Lat	Lng_e	Lat_e	count
终点编号	起点编号	起点经度	起点纬度	终点经度	终点纬度	轨迹条数
![数据样本.png](https://github.com/shengunxiansen/Test/raw/master/数据样本.png)

# 三、计算
```
#定义y1,y0,z1,z0简化计算过程
y1<-lng_e
y0<-lng
z1<-lat_e
z0<-lat
di<-(y1-y0)/(z1-z0)
newdata<-data.frame(mydata,di)
#将有缺省值的行数据删除。
newdata<-newdata[complete.cases(newdata[,10]),]
#实际工作中，数据集很少是完整的，许多情况下样本中都会包括若干缺失值NA，这在进行数据分析和挖掘时比较麻烦。
di<-na.omit(di) 
#根据距离进行k—means算法聚类
x.kmeans<-kmeans(di,3)
newdata <-na.omit(newdata)
newdata<-data.frame(newdata,x.kmeans$cluster)
zy<-(newdata$Lng_e+newdata$Lng)/2
zx<-(newdata$Lat_e+newdata$Lat)/2
#根据列进行合并
c<-cbind(zx,zy)
#根据每条连线的中心点进行k—means算法聚类
x2.kmeans<-kmeans(c,3)
newdata<-data.frame(newdata,x2.kmeans$cluster)
#将聚类结果分为三类
my.data1<-subset(newdata, x2.kmeans.cluster== '1')
my.data2<-subset(newdata, x2.kmeans.cluster== '2')
my.data3<-subset(newdata, x2.kmeans.cluster== '3')
```
