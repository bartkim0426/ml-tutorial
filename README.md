## nnstreamer

### In mac OSX

Install

```
brew install nnstreamer

brew install gst-plugins-badh
```

Additional installation

```
# For example
brew install pygobject3 gtk+3
```

> brew로 설치된 python 환경에서만 사용가능 (`/usr/local/bin/python3`)


#### Examples & get model

```
git clone https://github.com/nnstreamer/nnstreamer-example.git

cd nnstreamer-example/bash_script/example_models
# Usage: ./get_model.sh <model_name>
./get_model.sh object-detection-tf
```

#### Example in mac

mac config (ubuntu 대신)

- v4l2src -> avfvideosrc (autovideosrc)
- ximagesink -> autovideosink

**Object detection example**

https://github.com/nnstreamer/nnstreamer-example/blob/master/bash_script/example_object_detection_tensorflow/gst-launch-object-detection-tf.sh

```
# 위 예시에서 get_model 한 directory에서 진행

gst-launch-1.0 \
    autovideosrc device=/dev/vedio0 name=cam_src ! videoscale ! videoconvert ! video/x-raw,width=640,height=480,format=RGB ! tee name=t \
    t. ! queue leaky=2 max-size-buffers=2 ! videoscale ! tensor_converter ! \
        tensor_filter framework=tensorflow model=tf_model/ssdlite_mobilenet_v2.pb \
            input=3:640:480:1 inputname=image_tensor inputtype=uint8 \
            output=1,100:1,100:1,4:100:1 \
            outputname=num_detections,detection_classes,detection_scores,detection_boxes \
            outputtype=float32,float32,float32,float32 ! \
        tensor_decoder mode=bounding_boxes option1=tf-ssd option2=tf_model/coco_labels_list.txt option4=640:480 option5=640:480 ! \
        compositor name=mix sink_0::zorder=2 sink_1::zorder=1 ! videoconvert ! autovideosink \
    t. ! queue leaky=2 max-size-buffers=10 ! mix.
```


> 원래 설정에서 video src를 변경하고, 이유는 모르겠지만 원래 예제에서 `framerate=30/1`을 제거해주어야 잘 작동한다.
