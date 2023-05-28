---
published: true

title:  "8_bag->csv,bag->jpg"

categories: 
  - Create_DataSet

---

# bag -> csv

**/gaze** (/eye2)

**/xsense**

위 두 항목은 synchronization이 완료된 .bag파일에서 바로 **csv파일로 변환**하여 확인

[Rosbag to CSV 변환](https://velog.io/@qaszx1004/Rosbag-to-CSV-%EB%B3%80%ED%99%98GUI-%ED%83%80%EC%9E%85)

# bag -> jpg

origin topic name (synced topic name)

**/FOV** (eye1) → fov0001.jpg

**/depth/image_raw** (/ir_1) → dep0001.jpg

**/depth_to_rgb/image_raw** (/ir_2) → dtr0001.jpg

**/rgb/image_raw** (/ir_3) → rgb0001.jpg

**/rgb_to_depth/image_raw** (/ir_4) → rtd0001.jpg (barely used)

[image_view - ROS Wiki](http://wiki.ros.org/image_view#image_view.2Fdiamondback.extract_images)

[image_pipeline/image_view at noetic · ros-perception/image_pipeline](https://github.com/ros-perception/image_pipeline/tree/noetic/image_view)

<br><br>
위 프로그램 gitclone 후 이미지로 변환할 bag파일 터미널에서 실행

`rosrun image_view extract_images image:=eye1`

패키지 내에서
<br><br>
_extract_images_sync.py

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Software License Agreement (BSD License)
#
#  Copyright (c) 2015, Willow Garage, Inc.
#  All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#  modification, are permitted provided that the following conditions
#  are met:
#
#   * Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#   * Redistributions in binary form must reproduce the above
#     copyright notice, this list of conditions and the following
#     disclaimer in the documentation and/or other materials provided
#     with the distribution.
#   * Neither the name of the Willow Garage nor the names of its
#     contributors may be used to endorse or promote products derived
#     from this software without specific prior written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
#  "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
#  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
#  FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
#  COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
#  INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
#  BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
#  LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
#  CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
#  LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
#  ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
#  POSSIBILITY OF SUCH DAMAGE.
"""Save images of multiple topics with timestamp synchronization.

Usage: rosrun image_view extract_images_sync _inputs:='[<topic_0>, <topic_1>]'
"""

import sys

import cv2

import cv_bridge
import message_filters
import rospy
from sensor_msgs.msg import Image

class ExtractImagesSync(object):

    def __init__(self):
        self.seq = 0
        self.fname_fmt = rospy.get_param(
            '~filename_format', 'frame%04i_%i.jpg')
        self.do_dynamic_scaling = rospy.get_param(
            '~do_dynamic_scaling', False)
        img_topics = rospy.get_param('~inputs', None)
        if img_topics is None:
            rospy.logwarn("""\\
extract_images_sync: rosparam '~inputs' has not been specified! \\
Typical command-line usage:
\\t$ rosrun image_view extract_images_sync _inputs:=<image_topic>
\\t$ rosrun image_view extract_images_sync \\
_inputs:='[<image_topic>, <image_topic>]'""")
            sys.exit(1)
        if not isinstance(img_topics, list):
            img_topics = [img_topics]
        subs = []
        for t in img_topics:
            subs.append(message_filters.Subscriber(t, Image))
        if rospy.get_param('~approximate_sync', False):
            sync = message_filters.ApproximateTimeSynchronizer(
                subs, queue_size=100, slop=.1)
        else:
            sync = message_filters.TimeSynchronizer(
                subs, queue_size=100)
        sync.registerCallback(self.save)

    def save(self, *imgmsgs):
        seq = self.seq
        bridge = cv_bridge.CvBridge()
        for i, imgmsg in enumerate(imgmsgs):
            img = bridge.imgmsg_to_cv2(imgmsg)
            channels = img.shape[2] if img.ndim == 3 else 1
            encoding_in = bridge.dtype_with_channels_to_cvtype2(
                img.dtype, channels)
            img = cv_bridge.cvtColorForDisplay(
                img, encoding_in=encoding_in, encoding_out='',
                do_dynamic_scaling=self.do_dynamic_scaling)
            fname = self.fname_fmt % (seq, i)
            print('Save image as {0}'.format(fname))
            cv2.imwrite(fname, img)
        self.seq = seq + 1

if __name__ == '__main__':
    rospy.init_node('extract_images_sync')
    extractor = ExtractImagesSync()
    rospy.spin()

```
<br><br>
extract_image.cpp 파일에서

```python
/*********************************************************************
* Software License Agreement (BSD License)
*
*  Copyright (c) 2008, Willow Garage, Inc.
*  All rights reserved.
*
*  Redistribution and use in source and binary forms, with or without
*  modification, are permitted provided that the following conditions
*  are met:
*
*   * Redistributions of source code must retain the above copyright
*     notice, this list of conditions and the following disclaimer.
*   * Redistributions in binary form must reproduce the above
*     copyright notice, this list of conditions and the following
*     disclaimer in the documentation and/or other materials provided
*     with the distribution.
*   * Neither the name of the Willow Garage nor the names of its
*     contributors may be used to endorse or promote products derived
*     from this software without specific prior written permission.
*
*  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
*  "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
*  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
*  FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
*  COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
*  INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
*  BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
*  LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
*  CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
*  LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
*  ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
*  POSSIBILITY OF SUCH DAMAGE.
*********************************************************************/

#include <opencv2/highgui/highgui.hpp>

#include <ros/ros.h>
#include <sensor_msgs/Image.h>
#include <cv_bridge/cv_bridge.h>
#include <image_transport/image_transport.h>

#include <boost/thread.hpp>
#include <boost/format.hpp>

#include <fstream>

class ExtractImages
{
private:
  image_transport::Subscriber sub_;

  sensor_msgs::ImageConstPtr last_msg_;
  boost::mutex image_mutex_;

  std::string window_name_;
  boost::format filename_format_;
  int count_;
  double _time;
  double sec_per_frame_;

#if defined(_VIDEO)
  CvVideoWriter* video_writer;
#endif //_VIDEO

public:
  ExtractImages(const ros::NodeHandle& nh, const std::string& transport)
    : filename_format_(""), count_(0), _time(ros::Time::now().toSec())
  {
    std::string topic = nh.resolveName("image");
    ros::NodeHandle local_nh("~");

    std::string format_string;
    local_nh.param("filename_format", format_string, std::string("frame%04i.jpg"));
    filename_format_.parse(format_string);

    local_nh.param("sec_per_frame", sec_per_frame_, 0.1);

    image_transport::ImageTransport it(nh);
    sub_ = it.subscribe(topic, 1, &ExtractImages::image_cb, this, transport);

#if defined(_VIDEO)
    video_writer = 0;
#endif

    ROS_INFO("Initialized sec per frame to %f", sec_per_frame_);
  }

  ~ExtractImages()
  {
  }

  void image_cb(const sensor_msgs::ImageConstPtr& msg)
  {
    boost::lock_guard<boost::mutex> guard(image_mutex_);

    // Hang on to message pointer for sake of mouse_cb
    last_msg_ = msg;

    // May want to view raw bayer data
    // NB: This is hacky, but should be OK since we have only one image CB.
    if (msg->encoding.find("bayer") != std::string::npos)
      boost::const_pointer_cast<sensor_msgs::Image>(msg)->encoding = "mono8";

    cv::Mat image;
    try
    {
      image = cv_bridge::toCvShare(msg, "bgr8")->image;
    } catch(cv_bridge::Exception)
    {
      ROS_ERROR("Unable to convert %s image to bgr8", msg->encoding.c_str());
    }

    double delay = ros::Time::now().toSec()-_time;
    if(delay >= sec_per_frame_)
    {
      _time = ros::Time::now().toSec();

      if (!image.empty()) {
        std::string filename = (filename_format_ % count_).str();

#if !defined(_VIDEO)
        // Save raw image if the defined file extension is ".raw", otherwise use OpenCV
        std::string file_extension = filename.substr(filename.length() - 4, 4);
        if (filename.length() >= 4 && file_extension == ".raw")
        {
          std::ofstream raw_file;
          raw_file.open(filename.c_str());
          if (raw_file.is_open() == false)
          {
            ROS_WARN_STREAM("Failed to open file " << filename);
          }
          else
          {
            raw_file.write((char*)(msg->data.data()), msg->data.size());
            raw_file.close();
          }
        }
        else
        {
          if (cv::imwrite(filename, image) == false)
          {
            ROS_WARN_STREAM("Failed to save image " << filename);
          }
        }
#else
        if(!video_writer)
        {
            video_writer = cvCreateVideoWriter("video.avi", CV_FOURCC('M','J','P','G'),
                int(1.0/sec_per_frame_), cvSize(image->width, image->height));
        }

        cvWriteFrame(video_writer, image);
#endif // _VIDEO

        ROS_INFO("Saved image %s", filename.c_str());
        count_++;
      } else {
        ROS_WARN("Couldn't save image, no data!");
      }
    }
  }
};

int main(int argc, char **argv)
{
  ros::init(argc, argv, "extract_images", ros::init_options::AnonymousName);
  ros::NodeHandle n;
  if (n.resolveName("image") == "/image") {
    ROS_WARN("extract_images: image has not been remapped! Typical command-line usage:\\n"
             "\\t$ ./extract_images image:=<image topic> [transport]");
  }

  ExtractImages view(n, (argc > 1) ? argv[1] : "raw");

  ros::spin();

  return 0;
}

```
<br>
input / output format 및 파일 저장 이름 변경 가능. (default 값은 frame0001.jpg~)

⇒ 각각의 토픽의 저장 이름에 맞게 매번 변환 필요.


## [Bag파일에서 JPG로 변환]
![rgb0000](https://github.com/johook/Data-Synchronization/assets/116954375/5807a529-a15f-44e0-b50c-e6eb91056b5c)
[RGB]
<br><br><br>
![dep0000](https://github.com/johook/Data-Synchronization/assets/116954375/a922adfc-2fd3-425a-baac-8dd47ea0a2d0)
[depth]
<br><br><br>
![dtr0000](https://github.com/johook/Data-Synchronization/assets/116954375/b81178d5-3cdb-432b-a259-528e1550e5cf)
[depth_to_RGB]
<br><br><br>
![fov0000](https://github.com/johook/Data-Synchronization/assets/116954375/e8c90cf8-66c4-465a-8851-d64031937587)
[FOV]
<br><br><br>
## [bag파일에서 csv로 변환]

[final_6-eye_2.csv](https://github.com/johook/Data-Synchronization/files/11584318/final_6-eye_2.csv)
[아이트래커 gazepoint 좌표들]<br><br>
[final_6-xsens_.csv](https://github.com/johook/Data-Synchronization/files/11584319/final_6-xsens_.csv)
[GPS 위치좌표들]
