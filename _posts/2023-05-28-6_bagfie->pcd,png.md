---
published: true

title:  "6_bagfie->pcd,png"

categories: 
  - Create_DataSet

---

# map_generate(bag->pcd, bag->png)

동기화 처리 후 bag file로 저장된 topic들을 각각의 형식(.pcd, .png, .csv등)으로 변환 해줘야한다.

map_generate code는 그 중 Lidar와Camera를 각각 pcd, png파일로 변환해주는 code이다.

아래 github에서 Tools_RosBag2KITTI 패키지를 git clone하면 map_generate code를 받을 수 있다.

[https://github.com/leofansq/Tools_RosBag2KITTI](https://github.com/leofansq/Tools_RosBag2KITTI)

```cpp
#include "map_generation_node.h"
//This code is to read the .bag files in ROS and decode them into .png and .pcd
//Coordinate system transformation function is optional. 
using namespace cv;

double trans_roll_  = 0.0;
double trans_pitch_ = 0.0;
double trans_yaw_   = 0.0;
double trans_tx_    = 0.0;
double trans_ty_    = 0.0;
double trans_tz_    = 0.0;
Eigen::Affine3f transform_matrix_ = Eigen::Affine3f::Identity();

MapGenerationNode::MapGenerationNode():	lidar_index(0),camera_index(0), camera_captured(false), it(nh),
										init_camera_time(false), init_lidar_time(false)
{
    
	ros::param::set("~image_transport", "compressed");
	//이부분이 topic을 subscribe하는 부분이다. topic명만 수정하면된다.
	sub_camera = nh.subscribe("/flir_1", 1000, &MapGenerationNode::cameraCallback, this);
	sub_lidar = nh.subscribe("/livox_1", 1000, &MapGenerationNode::lidarCallback, this);
    ros::spin();
}

//About Lidar Points Cloud (Read & Trans & Save)
void MapGenerationNode::lidarCallback(const sensor_msgs::PointCloud2::ConstPtr& lidar)
{
	if(!init_lidar_time)
	{
		lidar_base_time = lidar->header.stamp.sec * 1e3 + lidar->header.stamp.nsec / 1e6;
		init_lidar_time = true;
	}

	long long lidar_delta_time = lidar->header.stamp.sec * 1e3 + lidar->header.stamp.nsec / 1e6 - lidar_base_time;
	ROS_INFO("get lidar : %lld ms", lidar_delta_time);

	char s[200];

	// 이부분이 Lidar의 저장경로를 설정해주는 부분이다. 저장경로에 해당되는 폴더 생성 후 경로설정해주면 된다.
	sprintf(s, "/home/kaai/catkin_ws/src/Tools_RosBag2KITTI/catkin_ws/output/K6/3/a1/%06lld.pcd", lidar_index); 
	++lidar_index;
	camera_captured = false;

	pcl::fromROSMsg(*lidar, lidar_cloud);

	//Building the transformation matrix
	transform_matrix_.translation() << trans_tx_, trans_ty_, trans_tz_;
  	transform_matrix_.rotate(Eigen::AngleAxisf(trans_yaw_ * M_PI / 180, Eigen::Vector3f::UnitZ()));
  	transform_matrix_.rotate(Eigen::AngleAxisf(trans_pitch_ * M_PI / 180, Eigen::Vector3f::UnitY()));
  	transform_matrix_.rotate(Eigen::AngleAxisf(trans_roll_ * M_PI / 180, Eigen::Vector3f::UnitX()));

  	//Transformation
	pcl::PointCloud<pcl::PointXYZI>::Ptr trans_cloud_ptr(new pcl::PointCloud<pcl::PointXYZI>);
  	pcl::transformPointCloud(lidar_cloud, *trans_cloud_ptr, transform_matrix_);
	//Save in .pcd
	pcl::io::savePCDFileASCII (s, *trans_cloud_ptr);
}

//About Visual Image (Read & Save)
void MapGenerationNode::cameraCallback(const sensor_msgs::ImageConstPtr& camera)
{
	if(!init_camera_time)
	{
		camera_base_time = camera->header.stamp.sec * 1e3 + camera->header.stamp.nsec / 1e6;
		init_camera_time = true;
	}

	long long camera_delta_time = camera->header.stamp.sec * 1e3 + camera->header.stamp.nsec / 1e6 - camera_base_time;
	if(camera_captured)
	{
		//ROS_INFO("discard camera: %lld ms", camera_delta_time);
		return;
	}

	ROS_INFO("get camera: %lld ms", camera_delta_time);
	char s[200];

	// 이부분이 Lidar의 저장경로를 설정해주는 부분이다. 저장경로에 해당되는 폴더 생성 후 경로설정해주면 된다.
	sprintf(s, "/home/kaai/catkin_ws/src/Tools_RosBag2KITTI/catkin_ws/output/K6/3/flir1/%06lld.png", camera_index); 
    ++camera_index;
	cv_bridge::CvImagePtr cv_ptr;
	try
	{
		cv_ptr = cv_bridge::toCvCopy(camera, sensor_msgs::image_encodings::BGR8); 
		//cv::imshow("view", cv_bridge::toCvShare(msg, "bgr8")->image);
	}
	catch (cv_bridge::Exception& e)
	{
		ROS_ERROR("Could not convert from '%s' to 'bgr8'.", camera->encoding.c_str());
	}
	Mat img_rgb = cv_ptr->image;
	imwrite(s, img_rgb);

	//camera_captured = true;
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "map_generation1");
    MapGenerationNode mapgeneration;
    return 0;
}

```

