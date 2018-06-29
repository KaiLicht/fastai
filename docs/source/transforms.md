# fastai.transforms

## Introduction and Overview
The fastai transforms pipeline for images is designed to convert your images into a form ready to be batched by your DataLoader and passed to your model. It can be used to conveniently create transformations like normalization/denormalization and various augmentations. The module contains different predefined pipelines that you can almost directly pass to your data object. This is especially convenient for working with pretrained models, because the images should be normalized according to the statistics of the (pre)training data set.

## Normalization
One of the most common image transformations is normalizing the images. The transforms module provides predefined pipelines in which you can pass the statistics (mean and standard deviation of each channel) of your training data. Make sure you use *only* the statistics of the training data to normalize both training and validation data.

### Normalizing for Pretrained Models
The statistics for all pretrained models available via the fastai.conv_learner module are predefined and accesible via the `tfms_from_model` function. Here's an example of how to use this function to create a pipeline for a pretrained _resnet34_ model:
```python
from fastai.transforms import *
from fastai.dataset import *

arch = resnet34 
sz = 224

data = ImageClassifierData.from_paths(PATH, tfms=tfms_from_model(arch, sz))
```
The `tfms_from_model` function returns separate image transformers for the training and validation set. It calls the `tfms_from_stats` function for custom image statistics and passes the mean and the standard deviation of the imagenet dataset.

### Normalizing with Custom Statistics
If you train a model from scratch you can use the `tfms_from_stats` function and pass in the statistics of your training set. For each channel you have to define mean and standard deviation:

```python
from fastai.transforms import *
from fastai.dataset import *

stats = A([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]) 
sz = 224

data = ImageClassifierData.from_paths(PATH, tfms=tfms_from_stats(stats, sz))
```

## Adding Augmentation
Besides normalization this module also contains transformations used for image augmentation

### Predefined Augmentations

### Custom Augmentations 

The most common types of transforms are predefined in ...

The most likely customizations you might need to do are ...

You can create custom transform pipelines using an approach like: ...

If you want to create a custom transform, you will need to : ...

## Transformations
Each transformation is defined as a class inherited from the abstract parent class `Transform`. To define a transformation overwrite the abstract method `do_transform`. The currently implemented transformations are:


== Class Transform [.small]#(tfm_y=TfmType.NO)#

=== Arguments

tfm_y (type TfmType, default TfmType.NO)::
        Type of transform. For details, see xref:TfmType[TfmType]

=== Methods

set_state::
A transform may include a random component. If it does, it will often need to transform `y` using the same random values as `x` (e.g. a horizontal flip in segmentation must be applied to the mask as well). Therefore, this method is used to ensure all random state is calculated in one place.
+
**NB:** Transformations are often run in multiple threads. Therefore any state must be stored in thread local storage. The `Transform` class provide a thread local `store` attribute for you to use. See {{xref RandomFlip}} for an example of how to use random state safely in `Transform` subclasses.

[[TfmType]]
== Class TfmType:IntEnum

.Type of transformation.

NO:: the default, y does not get transformed when x is transformed.
PIXEL:: x and y are images and should be transformed in the same way. _E.g.: image segmentation._
COORD:: y are coordinates (i.e bounding boxes)
CLASS:: y are class labels (same behaviour as PIXEL, except no normalization)

