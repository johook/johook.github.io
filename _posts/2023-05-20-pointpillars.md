---
layout: post
title:  "pointpillars"
---

# pointpillars


- 새로운 encoder 및 Network 제안 → end-to-end learning 가능
- Kitti Dataset에서 성능이 제일 좋다(이 당시)
- 모든 pillar에 대해 density한 2D CNN 표현가능하기 때문에 빠르다




# Pipeline


Pillar Feature Network→Backbone(2D CNN) → Detection Head (SSD)



![IMG_0192](https://github.com/johook/Codingtest/assets/116954375/debf48b1-d25f-42c0-84d0-13751aa04e70)
