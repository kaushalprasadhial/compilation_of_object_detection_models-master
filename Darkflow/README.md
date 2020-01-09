# YOLO: You Only Look Once
YOLO is an extremely fast real time multi object detection algorithm. YOLO stands for “You Only Look Once”. This is the link to the original paper : 
https://pjreddie.com/media/files/papers/YOLOv3.pdf.

## Collect and annotate images
The steps to annotate images using LabelImg:

  1 Create a folder contains images files and name it "images".  
  2 Create a folder contains annotations files and name it "annotations".  
  3 Open LabelImg application.  
  4 Click on “Open Dir” and then choose the Images folder.  
  5 Click on “Change Save Dir” and then choose the annotations folder.  
  6 You will find that all images are listed in the File List panel.  
  7 Click on the Image you want to annotate.  
  8 Click the letter “W” from your keyboard to draw the rectangle on the desired image object, type the name of the object on the popped  up window.  
  9 Click ”CTRL+S” to save the annotation to the XML file in the annotations folder.  
  10 Repeat steps 8 to 10 till you complete annotating all the images.  
  
## Download Darkflow
Download [darkflow](https://github.com/llSourcell/YOLO_Object_Detection) repository from github
Download Yolo weights and configuration file form [this link](https://pjreddie.com/darknet/yolo/)
you will need to modify the YOLOv3 tiny model (yolov3-tiny.cfg) to train our custom detector. This modification includes:

  * Uncomment the lines 5,6, and 7 and change the training batch to 64 and subdivisions to 2.  
  * Change the number of filters for convolutional layer "[convolution]" just before every yolo output "[yolo]" such that the number of       filters= #anchors x (5 + #ofclasses)= 3x(5+1)= 18. The number 5 is the count of parameters center_x, center_y, width, height, and objectness Score. So, change the lines 127 and 171 to "filters=18".  
  * For every yolo layer [yolo] change the number of classes to 1 as in lines 135 and 177.

## Training on your own dataset
*The steps below assume we want to use tiny YOLO and your dataset has 3 classes*

  1. Create a copy of the configuration file `tiny-yolo-voc.cfg` and rename it according to your preference `tiny-yolo-voc-3c.cfg` (It is crucial that you leave the original `tiny-yolo-voc.cfg` file unchanged, see below for explanation).

  2. In `tiny-yolo-voc-3c.cfg`, change classes in the [region] layer (the last layer) to the number of classes you are going to train for. In our case, classes are set to 3.
    
    ```python
    ...

    [region]
    anchors = 1.08,1.19,  3.42,4.41,  6.63,11.38,  9.42,5.11,  16.62,10.52
    bias_match=1
    classes=3
    coords=4
    num=5
    softmax=1
    
    ...
    ```

  3. In `tiny-yolo-voc-3c.cfg`, change filters in the [convolutional] layer (the second to last layer) to num * (classes + 5). In our case, num is 5 and classes are 3 so 5 * (3 + 5) = 40 therefore filters are set to 40.
    
    ```python
    ...

    [convolutional]
    size=1
    stride=1
    pad=1
    filters=40
    activation=linear

    [region]
    anchors = 1.08,1.19,  3.42,4.41,  6.63,11.38,  9.42,5.11,  16.62,10.52
    
    ...
    ```

  4. Change `labels.txt` to include the label(s) you want to train on (number of labels should be the same as the number of classes you set in `tiny-yolo-voc-3c.cfg` file). In our case, `labels.txt` will contain 3 labels.

    ```
    label1
    label2
    label3
    ```
  5. Reference the `tiny-yolo-voc-3c.cfg` model when you train.

    `flow --model cfg/tiny-yolo-voc-3c.cfg --load bin/tiny-yolo-voc.weights --train --annotation train/Annotations --dataset train/Images`


* Why should I leave the original `tiny-yolo-voc.cfg` file unchanged?
    
    When darkflow sees you are loading `tiny-yolo-voc.weights` it will look for `tiny-yolo-voc.cfg` in your cfg/ folder and compare that configuration file to the new one you have set with `--model cfg/tiny-yolo-voc-3c.cfg`. In this case, every layer will have the same exact number of weights except for the last two, so it will load the weights into all layers up to the last two because they now contain different number of weights.


## Camera/video file demo

For a demo that entirely runs on the CPU:

```bash
flow --model cfg/yolo-new.cfg --load bin/yolo-new.weights --demo videofile.avi
```

For a demo that runs 100% on the GPU:

```bash
flow --model cfg/yolo-new.cfg --load bin/yolo-new.weights --demo videofile.avi --gpu 1.0
```

To use your webcam/camera, simply replace `videofile.avi` with keyword `camera`.

To save a video with predicted bounding box, add `--saveVideo` option.
