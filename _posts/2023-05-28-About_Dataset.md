---
published: true

title:  "About_Dataset"

categories: 
  - Create_DataSet

---


# DataSet
KaAI동아리에서 만든 Dataset은 Lidar,Camera, Can, GPS, Radar,Eyetracker 이렇게 6가지의 센서들을 Syncronization한 Dataset이다. 

다른 대다수의 DataSet(예를들어 Kitti, Waymo 등)은 차량 외부의 정보들(Camera, Lidar, GPS 등)만 포함한 DataSet이 대다수이지만 우리의 DataSet은 차량내부 즉 운전자의 시선추적이 가능한 Gazepoint와 운전자의 이미지까지 포함된 DataSet이다.

이러한 DataSet을 만든 과정과 어떤 Synchronization 알고리즘을 사용하였는지에 대한 설명을 진행 할 것이다.

##  각 센서 설명

Lidar(Livox Horizon)
<br>
![image](https://github.com/johook/Data-Synchronization/assets/116954375/2ac008b7-39c9-4aa0-9be4-b23a483ef69c)
<br>

|재원|Livox Horizon| 
|---|---|
|HFOV|81.7°|
|VFOV|25.1°|
|Point rate|240,000 pts/s|
|Detection range|260 m|

<br><br>
**Camera(FLIR)**
모델명: BFS-PGE-19S4C-C
<br>
![image](https://github.com/johook/Data-Synchronization/assets/116954375/79fb60e0-1285-4144-a23b-4fc43e4cc231)
<br>

|재원|Livox Horizon| 
|---|---|
|camera resolution|1632 * 1248|
|average frame|60fps → 30fps(4)|
|sensor name|Sony IMX430|
|sensor type|CMOS|
|sensor format|1/1,7”|
|reading method|global shutter|
|pixel size|4.5 µm|
|ADC|12-bit|

<br><br><br>
**EyeTracker(pupil)**
<br>
![image](https://github.com/johook/Data-Synchronization/assets/116954375/ac8b4970-c372-40b5-a219-882528005816)
<br><br><br>
**Radar(Mobileye)**

Mobileye를 통해 받을수있는 정보들
[MobilEye (1) (2).xlsx](https://github.com/johook/Data-Synchronization/files/11584032/MobilEye.1.2.xlsx)

**CAN(kvaser)**
<br><br>
![image](https://github.com/johook/Data-Synchronization/assets/116954375/bf8e2de7-3b63-4bbc-beba-2c28b5e67115)
<br><br><br>CAN을 통해 받을 수 있는 정보들 

[CAN 데이터 설명 (NIRO).xlsx](https://github.com/johook/Data-Synchronization/files/11584038/CAN.NIRO.xlsx)
<br><br><br>

**GPS(Xsens)**
제품명: MTI-G-710
![Untitled](https://github.com/johook/Data-Synchronization/assets/116954375/130d0fa4-7316-447c-a8a7-827f37573ad8)
xsens를 통해 받을 수 있는 정보들 
[xsens_topiclist.ods](https://github.com/johook/Data-Synchronization/files/11584042/xsens_topiclist.ods)


