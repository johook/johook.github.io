---
published: true

title:  "Lidar_Merge"

categories: 
  - Create_DataSet

---

# Lidar Merge

기존의 Livox Hub로 받아와서 6개의 라이다를 한 개로 합치는 과정은 문제가 있었다.
![Untitled](https://github.com/johook/Data-Synchronization/assets/116954375/fb3465e8-447e-4a74-9b95-f62dbc2e733e)

Rviz상에서 decay time을 0으로 했을때 Lidar들이 동시에 정보를 받아오는 것이 아니라 번갈아가며 빠른 속도로 받아온다는 것이였다.

decay time을 조정하면 동시에 나오지만 이는 visual상에서만 나오는 것이였고 bag를 했을때는 원하는 방식으로 나오지 않았다.

Hub의 문제라고 생각하여 hub로 합치는 과정을 없애고 Lidar 6개의 토픽을 모두 받아 그 pcd들을 하나로 합치기로 하였다.

코드는 다음과 같다.

주석을 확인하여 내가 원하는 경로로 설정해두면된다.

```python
import open3d as o3d
import numpy as np
import glob, pcl

point_cloud1 = []
str1 = "00000"

for k in range(0, 704): #for문에서 범위는 내가 가지고있는 pcd파일수+1을 한다. 그 수를 
#넘게 되면 똑같은 파일이 계속 생성되어 혼동을 야기할수있다.
    x = 0
    path = "/home/kaai/imgodhaeng/*/" + str1 + str(k) + ".pcd" #pcd파일이 존재하는 경로를
#지정해준다. str1이 00000이고 str(k)는 0부터 진행된다.
    for i in glob.glob(path):  # * means inner directory
        pcd = o3d.io.read_point_cloud(i) #내가 지정해준 경로에 pcd파일을 읽는다.
        a = np.array(pcd.points) #거기서 pointcloud의 array를 뽑아낸다.
        if x == 0:
            p3_load = a
            print("초기화")
        else:
            p3_load = np.concatenate((p3_load, a), axis = 0) #뽑아낸 array들을 모두 합친다.
            print("진행중")
        x += 1
    if k >= 10: #파일수가 1000개 이하인 경우가 많았기에 000000부터 001000까지 str1을 조정해주는 부분이다.
        str1 = "000"
    elif k >= 100:
        str1 = "00"
    elif k >= 1000:
        str1 = "0"

~~~~
    pcdfile = o3d.geometry.PointCloud()
    pcdfile.points = o3d.utility.Vector3dVector(p3_load)
    #o3d.visualization.draw_geometries([pcdfile])
    o3d.io.write_point_cloud("./final/" + str(k) +".pcd", pcdfile)
		#합친 뒤에는 저장하는 부분이 필요하다. 경로를 지정해주면 그 지정폴더에 합쳐진 pcd가 존재한다.
#o3d.visualization.draw_geometries([pcdfile])

```

python으로 구현하였으며 open3d라는 패키지를 사용하였다.

패키지가 설치되어있지않으면 잘 구동되지않는다. 구동 전 open3d 패키지를 다운받도록하자.

```python
pip install open3d or pip3 install open3d

```
<br><br>
[Open3D – A Modern Library for 3D Data Processing](http://www.open3d.org/)

open3d는 3d의 데이터들을 다룰떄 사용하는 development software이라고 생각하면 된다.

코드를 사용하는 방법은 ubuntu에서 python파일을 실행해주는 방법과 동일하다.

```python
python open3d1.py or python3 open3d1.py

```
<br><br>
_**이 코드를 이용할 때 중요한 점은 livox들이 서로 calibration이 되어있어야 한다는 점이다. calibration이 되어있지 않으면 pcd를 합쳐도 서로 다른 좌표계이기 때문에 이상한 pcd가 나올 것이다.**_

process가 끝나면 pcl_viewer로 잘 합쳐졌는지 확인해본다.

```python
pcl_viewer 000000.pcd

```

<img src="https://github.com/johook/Data-Synchronization/assets/116954375/5cc14673-cb86-4ce4-ac11-b5df2bf6c0a6" alt="Untitled (16)" width="400" height="400">  <img src="https://github.com/johook/Data-Synchronization/assets/116954375/3573e9dd-6b28-4662-ab0b-f378c1bda9c1" alt="Untitled (17)" width="400" height="400">
