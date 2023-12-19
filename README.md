# Custom_Model_for_Object_Detection

This repo will guide you to train and validate your model for custom object detection using MS COCO, Darknet and Yolo.

## Requirements

- Google Colab
- Google Drive

Create a folder in your drive and name it `custom_detec`. Create a `backup` and `results` folder in it as well.

## Dataset

We will be using MS COCO dataset to build our training and validation dataset with annotations. First, we will need the images for this we will be using the `01-coco.ipynb` file. For the annotations we will run the `utils.py` file (annotations in label folder). This will give you the images and annotations. **Copy paste both the images and annotations in a new folder named obj, zip it and store it in the drive folder custom_detec**, we will need this for our training/validation later.

### To modify the classes you need

in 01-coco.ipynb:
```
catIds = coco.getCatIds(catNms=['person', 'car', 'bicycle']) # write the list of classes you need
```

in utils.py:
```
target = ['person', 'car', 'bicycle'] # write the list of classes you need
```

### To switch between training and validation data

in 01-coco.ipynb:
```
coco = COCO('COCOdataset2017/annotations/instances_train2017.json') 
coco = COCO('COCOdataset2017/annotations/instances_val2017.json')

...

im_folder = 'COCOdataset2017/images/train/'
im_folder = 'COCOdataset2017/images/valid/' 
```

in utils.py:
```
with open('train.txt', 'w') as f:
with open('test.txt', 'w') as f:

...

ap.add_argument('--anno', help='path to COCO annotation file', default=f'instances_train2017.json')
ap.add_argument('--anno', help='path to COCO annotation file', default=f'instances_val2017.json')
```

> Note: **utils.py will also create a obj.names file, store it in the drive folder custom_detec**, which we will use later.

## Training

Before diving into darknet, we will create a obj.data file (you can create it using VS Code) and add the following lines, based on you requirements change the number of classes. **After this store obj.data in your drive folder custom_detec**.

```
classes = <number of classes>
train = data/train.txt
valid= data/test.txt
names= data/obj.names
backup = /mydrive/custom_detec/backup
```

Now, for training we will be running the `custom_detec.ipynb` file on colab. Clone the darknet repo and make changes to the makefile as follows:

```
!git clone https://github.com/AlexeyAB/darknet

%cd darknet
!sed -i 's/OPENCV=0/OPENCV=1/' Makefile
!sed -i 's/GPU=0/GPU=1/' Makefile
!sed -i 's/CUDNN=0/CUDNN=1/' Makefile
# !sed -i 's/LIBSO=0/LIBSO=1/' Makefile # Uncomment this line if you want to use darknet.py helper functions
```

For training we will need, change the config file based on our custom classes. So, in `darknet/cfg` find the file `yolov3.cfg` copy and paste the same file and rename it as `yolov3_custom.cfg` in the same folder. Now, we have have to make the changes in the `yolov3_custom.cfg` file:


- batch and subdivision (64 and 16 or 64 and 32) should be set properly, or you’ll encounter OOM
- max_batches should be max(no_of_classes*2000, 6000)
- steps should be 80% and 90% of max_batches
- classes=80 should be modified to the number of target categories. Eg. classes = 2 if there are 2 classes
- filters=255 in yolo layer should be modified to (no_of_classes+5)*3

Now, you can either use the train.txt/test.txt file created by utils or use generate_train.py file I have provided to get the train.txt/test.txt file.

- If you are using generate_train.py copy it into your drive folder custom_detec and let the code create train.txt file or change give the validation data path and change file name to test.txt. 

- If not using generate_train.py file, comment the following lines from custom_detec.ipynb:

```
!cp /mydrive/bag_detec/generate_train.py ./
!python generate_train.py
```

After this you can see a darknet folder in colab files. Run the rest of the cells.

For training uncomment following lines:

```
!wget http://pjreddie.com/media/files/darknet53.conv.74
#!./darknet detector train data/obj.data cfg/yolov3_custom.cfg darknet53.conv.74 -dont_show
```
> The weigths file can be seen in you drive folder `custom_detec/backup`, we will use the final weights file for testing.

## Test on Image and Videos

We have to change the config file to run only 1 video or image:

```
%cd cfg
!sed -i 's/batch=64/batch=1/' yolov3_custom.cfg
!sed -i 's/subdivisions=16/subdivisions=1/' yolov3_custom.cfg
%cd ..
```

Run this command on image from drive folder custom_detec:
```
!./darknet detector test data/obj.data cfg/yolov3_custom.cfg /mydrive/custom_detec/backup/yolov3_custom_final.weights /mydrive/custom_detec/img_name.jpeg
imShow('predictions.jpg')
```

Run this command on video from drive folder custom_detec:
```
!./darknet detector demo data/obj.data cfg/yolov3_custom.cfg /mydrive/custom_detec/backup/yolov3_custom_final.weights -dont_show /mydrive/custom_detec/video_name.mp4 -out_filename output.mp4
```
