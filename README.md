# Custom_Model_for_Object_Detection

This repo will guide you to train and validate your model for custom object detection using MS COCO, Darknet and Yolo.

## Requirements

- Google Colab
- Google Drive

Create a folder in your drive and 

## Dataset

We will be using MS COCO dataset to build our training and validation dataset with annotations. First, we will need the images for this we will be using the `01-coco.ipynb` file. For the annotations we will run the `utils.py` file (annotations in label folder). This will give you the images and annotations. Copy paste both the images and annotations in a new folder named obj, we will need this for our training/validation later.

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

> Note: utils.py will also create a obj.names file, which we will use later.

