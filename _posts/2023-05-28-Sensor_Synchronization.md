---
published: true

title:  "Sensor_Synchronization"

categories: 
  - Create_DataSet

---

# Sensor Synchronization 알고리즘

## Approximate Time Policy

Ros에서 제공하는 Ros::Message_filters::sync::Approximate Time Policy를 사용한다.

![Untitled (13)](https://github.com/johook/Data-Synchronization/assets/116954375/b755044c-2847-434f-9d07-bbbb374f883c)
![Untitled (14)](https://github.com/johook/Data-Synchronization/assets/116954375/1173a309-2d6b-4f42-a4bb-54996e7cb0a4)

각 센서의 Topic Hz가 다르기 때문에, 이것을 맞춰주는 작업이 필요하고... 그 과정은 ROS 상에서 TimeSynchronizer filter를 통해서 메세지에 적용시킬 수 있다.

![Untitled (15)](https://github.com/johook/Data-Synchronization/assets/116954375/6c297d38-c32e-4100-8c04-ef28c3e67152)

== >요약 : ApproximateTime Policy 알고리즘을 사용하기 위해서는

**모든 메시지에 타임 스탬프를 확인할 수 있는 Header 필드가 필요**하다.

message_filters::sync_policies::ApproximateTime 를 사용하면, 메세지간의 Hz가 다르다고 하여도 adaptive algorithm을 사용하여 queue에 들어온 가장 마지막 메세지들을 연결해준다.

[Wiki_ApproximateTime](http://wiki.ros.org/message_filters/ApproximateTime)

**문제발생**

cpp코드로 진행하다보니까 여러개의 센서들을 동기화 할 때 Buffer overrun현상이 나타난다.

**문제해결**

각 센서들에서 보내는 Topic을 동기화 하는 Message_filter의 cpp 코드가 아닌, python 코드로 전환하여 시도하니 해결 되었다. ( **Buffer overrun 현상은 cpp의 코드에서 주로 발생**한다고 한다. )

[참고페이지]

[008. Buffer Overflow in C++ (1)](https://sh1r0hacker.tistory.com/151)

## Header header 추가과정

( Lidar / Camera / GPS / xsense 등 **대부분의 센서들이 header.stamp를 포함**하기 때문에,

**기존에 header msg를 포함하지 않던** Eye tracker의 gaze를 전송하는 pupil_listner.py 파일과 gazepoint.msg, 그리고 can data를 전송하는 [k7.listenrt.py](http://k7.listenrt.py) 파일과 can_std.msg 파일에 **Header header를 추가**하였다. )

-   우리의 세 센서 팀들은 모두 .msg파일에 Header header를 추가하였고, Header stamp로 전송하는 시간의 format이 UTC기준 + unix 시간 = ROS time 이 되므로,
    
    allow_headerless=True로 설정함으로써 다시금 **Header 메시지가 정확히 전송이 명확히 되지 않는 토픽에 한해 UTC시간을 사용할 수 있도록 할 수 있었다.**

## _Sync.py

ApproximateTime Policy 알고리즘을 사용해서 [sync.py](http://sync.py) 코드를 작성 하였다.

**문제발생**

너무 많은 hz가 다른 센서들의 topic이 하나의 코드로 들어가다 보니 잘 돌아가지 않았다.

**문제해결**

hz의 크기로 3 그룹을 만들었다.

1.  hz가 큰 can, mobileye
    
2.  hz가 작은 xsens
    
3.  hz가 중간인 나머지
    

따라서 hz가 큰 can, mobileye와 작은 xsens 그리고 중간인 Flir를 하나의 sync파일로 만들고

hz가 중간인 나머지 Topic들을 하나의 sync파일로 만들어 publish 하도록 했다.


## _sync.py

```python
#!/usr/bin/env python
#-*- coding:utf-8 -*-
import rospy
import message_filters
from std_msgs.msg import Header
from sensor_msgs.msg import Image,NavSatFix,PointCloud2
from tracker.msg import GazePoint
from kaaican.msg import can_std

#import cv2
#import numpy as np
#from cv_bridge import CvBridge,CvBridgeError
#import pcl
#import pcl_helper
#import pandas as pd

#pubflir1 = rospy.Publisher('flir1', Image, queue_size=20)
#pubflir2 = rospy.Publisher('flir2', Image, queue_size=20)
#pubflir3 = rospy.Publisher('flir3', Image, queue_size=20)
#pubflir4 = rospy.Publisher('flir4', Image, queue_size=20)
pubfov = rospy.Publisher('eye1', Image, queue_size=20)
pubgaze = rospy.Publisher('eye2', GazePoint, queue_size=20)
publivox1 = rospy.Publisher('livox1', PointCloud2, queue_size=20)
publivox2 = rospy.Publisher('livox2', PointCloud2, queue_size=20)
publivox3 = rospy.Publisher('livox3', PointCloud2, queue_size=20)
publivox4 = rospy.Publisher('livox4', PointCloud2, queue_size=20)
publivox5 = rospy.Publisher('livox5', PointCloud2, queue_size=20)
publivox6 = rospy.Publisher('livox6', PointCloud2, queue_size=20)
#pubvelo = rospy.Publisher('velodyne', PointCloud2, queue_size=20)
pubir1 = rospy.Publisher('ir1', Image, queue_size=20)
pubir2 = rospy.Publisher('ir2', Image, queue_size=20)
pubir3 = rospy.Publisher('ir3', Image, queue_size=20)
pubir4 = rospy.Publisher('ir4', Image, queue_size=20)
#pubgnss = rospy.Publisher('xsens', NavSatFix, queue_size=20)
#pubcan = rospy.Publisher('can', can_std, queue_size=20)
#pubmob = rospy.Publisher('mob', can_std, queue_size=20)

def second_callback(eye1,eye2,livox1,livox2,livox3,livox4,livox5,livox6,ir1,ir2,ir3,ir4): #FOV=eye1, gaze=eye2
    print('hi: ',eye1.header.stamp.secs)
    #pubflir1.publish(flir1)
    #pubflir1.publish(flir2)
    #pubflir1.publish(flir3)
    #pubflir1.publish(flir4)
    pubfov.publish(eye1)
    pubgaze.publish(eye2)
    publivox1.publish(livox1)
    publivox2.publish(livox2)
    publivox3.publish(livox3)
    publivox4.publish(livox4)
    publivox5.publish(livox5)
    publivox6.publish(livox6)
    #pubvelo.publish(velodyne)
    pubir1.publish(ir1)
    pubir2.publish(ir2)
    pubir3.publish(ir3)
    pubir4.publish(ir4)
    #pubgnss.publish(xsens) 
    #pubcan.publish(can)
    #pubmob.publish(mob)

rospy.init_node('sensorsync')
while not rospy.is_shutdown():

    fov_sub_ = message_filters.Subscriber('/FOV',Image)
    gaze_sub_ = message_filters.Subscriber('/gaze',GazePoint)

    lidar1_sub_ = message_filters.Subscriber('/livox/lidar_1HDDH1200104181',PointCloud2)#라이다로부터 정보를 subscribe 그 이름을 lidar_sub_
    lidar2_sub_ = message_filters.Subscriber('/livox/lidar_1HDDH3200106141',PointCloud2)
    lidar3_sub_ = message_filters.Subscriber('/livox/lidar_3WEDH5900101801',PointCloud2)
    lidar4_sub_ = message_filters.Subscriber('/livox/lidar_3WEDH7600101631',PointCloud2)
    lidar5_sub_ = message_filters.Subscriber('/livox/lidar_3WEDH7600113901',PointCloud2)
    lidar6_sub_ = message_filters.Subscriber('/livox/lidar_3WEDH7600114591',PointCloud2)
    #velodyne_sub_ = message_fsilters.Subscriber('/velodyne_points',PointCloud2)
    dept_ir_sub_ = message_filters.Subscriber('/depth/image_raw',Image)
    dept_rgb_sub_ = message_filters.Subscriber('/depth_to_rgb/image_raw',Image)
    rgb_sub_ = message_filters.Subscriber("/rgb/image_raw", Image)
    rgb_dept_sub_ = message_filters.Subscriber("/rgb_to_depth/image_raw", Image)
    #gnss_sub_ = message_filters.Subscriber('/gnss',NavSatFix)
    #can_sub_ = message_filters.Subscriber('/msg_n',can_std)
    #mob_sub_ = message_filters.Subscriber('/msg_m',can_std)

    ts = message_filters.ApproximateTimeSynchronizer([fov_sub_, gaze_sub_,lidar1_sub_,lidar2_sub_,lidar3_sub_,lidar4_sub_,lidar5_sub_,lidar6_sub_,dept_ir_sub_, dept_rgb_sub_, rgb_sub_, rgb_dept_sub_],10,0.1,allow_headerless=False) #여기서는 파라미터만 바꿔라

    ts.registerCallback(second_callback)
    rospy.spin()

```

## _sync2.py

```
#!/usr/bin/env python
#-*- coding:utf-8 -*-
import rospy
import message_filters
from std_msgs.msg import Header
from sensor_msgs.msg import Image,NavSatFix,PointCloud2
from tracker.msg import GazePoint
from kaaican.msg import can_std
from kaaican.msg import Niro
from kaaican.msg import Mobileye
#import cv2
#import numpy as np
#from cv_bridge import CvBridge,CvBridgeError
#import pcl
#import pcl_helper
#import pandas as pd

pubflir1 = rospy.Publisher('flir1', Image, queue_size=20)
pubflir2 = rospy.Publisher('flir2', Image, queue_size=20)
pubflir3 = rospy.Publisher('flir3', Image, queue_size=20)
pubflir4 = rospy.Publisher('flir4', Image, queue_size=20)
#pubvelo = rospy.Publisher('velodyne', PointCloud2, queue_size=20)
#pubreal1 = rospy.Publisher('real1', Image, queue_size=20)
#pubreal2 = rospy.Publisher('real2', Image, queue_size=20)
pubgnss = rospy.Publisher('xsens', NavSatFix, queue_size=20)
pubcan = rospy.Publisher('can', can_std, queue_size=20)
pubmob = rospy.Publisher('mob', can_std, queue_size=20)

def second_callback(flir1, flir2, flir3, flir4, xsens, can, mob): #FOV=eye1, gaze=eye2
    print("camera: ",flir1.header.stamp.secs)
    pubflir1.publish(flir1)
    pubflir2.publish(flir2)
    pubflir3.publish(flir3)
    pubflir4.publish(flir4)

    #pubvelo.publish(velodyne)

    pubgnss.publish(xsens) 
    pubcan.publish(can)
    pubmob.publish(mob)

rospy.init_node('sensorsync2')
while not rospy.is_shutdown():
    image_sub_1 = message_filters.Subscriber('/camera_array/cam0/image_raw',Image)#카메라로부터 정보를 subscribe 그 이름을 image_sub_
    image_sub_2 = message_filters.Subscriber('/camera_array/cam1/image_raw',Image)#카메라로부터 정보를 subscribe 그 이름을 image_sub_
    image_sub_3 = message_filters.Subscriber('/camera_array/cam2/image_raw',Image)#카메라로부터 정보를 subscribe 그 이름을 image_sub_
    image_sub_4 = message_filters.Subscriber('/camera_array/cam3/image_raw',Image)#카메라로부터 정보를 subscribe 그 이름을 image_sub_

    #velodyne_sub_ = message_filters.Subscriber('/velodyne_points',PointCloud2)
    #real1_sub_ = message_filters.Subscriber('/camera/aligned_depth_to_color/image_raw',Image)
    #real2_sub_ = message_filters.Subscriber('/camera/color/image_raw',Image)
    gnss_sub_ = message_filters.Subscriber('/gnss',NavSatFix)
    can_sub_ = message_filters.Subscriber('/msg_n',can_std)
    mob_sub_ = message_filters.Subscriber('/msg_m',can_std)

    ts = message_filters.ApproximateTimeSynchronizer([image_sub_1, image_sub_2, image_sub_3, image_sub_4, gnss_sub_, can_sub_, mob_sub_],10,0.1,allow_headerless=False) #여기서는 파라미터만 바꿔라

    ts.registerCallback(second_callback)
    rospy.spin()

```

[동기화 진행중인 rqt_graph]

![Screenshot from 2023-02-25 13-59-56](https://github.com/johook/Data-Synchronization/assets/116954375/7526fce6-f506-4d8d-a43e-29977af96838)
