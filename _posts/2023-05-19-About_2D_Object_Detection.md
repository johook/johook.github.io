# 2D Object Detection에 대한 전반적인 이해

- 1-stage Object Detection과 2-stage Object Detection의 가장 큰 차이
    
    1-stage detection → YOLO, SSD
    
    2-stage detection → Faster RCNN
    
    1-stage Object Detection과 2-stage Object Detection의 가장 큰 차이는 객체 탐지를 위한 접근 방식입니다.
    
    1-stage Object Detection은 입력 이미지를 바로 CNN 모델에 통과시켜 객체를 탐지합니다. 이 방법은 간단하고 빠르게 객체를 탐지할 수 있으며, Region Proposal을 생성하고 처리하는 데 필요한 추가 계산을 생략할 수 있어서 모델의 속도가 빠릅니다. 
    
    2-stage Object Detection은 객체 탐지를 위해 두 단계로 처리합니다. 첫 번째 단계에서는 Region Proposal Network(RPN)을 사용하여 입력 이미지에서 객체가 있을 가능성이 있는 영역, 즉 Region Proposal을 생성합니다. 두 번째 단계에서는 이전 단계에서 생성된 Region Proposal 영역에서 실제 객체를 탐지하는 작업을 수행합니다. 이 방법을 사용하면 객체 탐지 정확도를 높일 수 있으며, 대규모 데이터셋에서도 빠르게 객체 탐지를 수행할 수 있습니다. 대표적으로 Faster R-CNN, R-FCN 등이 있습니다.
    
- Region Proposal이란?
    
     이미지에서 객체가 있을 만한 위치를 예측 및 제안하는 알고리즘으로 Selective Search, EdgeBoxes, R-CNN등의 알고리즘이 사용된다.
     왜 안돼
    

### [object_detection의 기본적인 내용 + YOLO에 대한 전반적인 내용]

[https://youtu.be/fdWx3QV5n44](https://youtu.be/fdWx3QV5n44)

 

![YOLO-1 (1)](https://github.com/johook/Codingtest/assets/116954375/cd180c94-507c-4b8d-ab40-d000ff00d359)
![YOLO-2](https://github.com/johook/Codingtest/assets/116954375/2b32fd2d-2479-4091-a79e-f123a976e43e)
![YOLO-3](https://github.com/johook/Codingtest/assets/116954375/4e11d8b8-4619-4aa7-ace0-197557baabc1)



