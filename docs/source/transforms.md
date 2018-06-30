# fastai.transforms

## Introduction and Overview
The fastai transforms pipeline for images is designed to convert your images into a form ready to be batched by your DataLoader and passed to your model. It can be used to conveniently create transformations like normalization/denormalization and various augmentations. The module contains different predefined pipelines that you can almost directly pass to your data object. This is especially convenient for working with pretrained models, because the images should be normalized according to the statistics of the (pre)training data set.

## Normalization
One of the most common image transformations is normalizing the images. The transforms module provides predefined pipelines in which you can pass the statistics (mean and standard deviation of each channel) of your training data. 

```eval_rst
.. important:: Make sure you use *only* the statistics of the training data to normalize both training and validation data.
```

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
```eval_rst
.. automodule:: fastai.transforms

.. autofunction:: tfms_from_model
```

### Normalizing with Custom Statistics
If you train a model from scratch you can use the `tfms_from_stats` function and pass in the statistics of your training set. For each channel you have to define mean and standard deviation:

```python
from fastai.transforms import *
from fastai.dataset import *

stats = A([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]) 
sz = 224

data = ImageClassifierData.from_paths(PATH, tfms=tfms_from_stats(stats, sz))
```
```eval_rst
.. autofunction:: tfms_from_stats
```

## Adding Augmentation
Besides normalization this module also contains transformations that can be used for image augmentation. Each of those transformations is defined as a class (see *Available Transformations* below). The functions `tfms_from_model` and `tfms_from_stats` can take lists of those classes as definitions which augmentations to perform.

### Predefined Augmentations
The module contains predefined sets of transformations that can be used for common image augmentations:

* `transforms_basic` contains a random rotation up to 10° and random lighting change up to 5%
* `transforms_side_on` contains `transforms_basic` and a random flip (suitable for pictures of "every day objects")
* `transforms_top_down` contains `transforms_basic` and a random rotation by multiples of 90° (for example suitable for satellite images)

To use those predefined sets of augmentations in `tfms_from_model` and `tfms_from_stats` you have to pass them to `aug_tfms`:

```python
from fastai.transforms import *
from fastai.dataset import *

stats = A([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]) 
sz = 224

data = ImageClassifierData.from_paths(PATH, tfms=tfms_from_stats(stats, sz, aug_tfms=transforms_basic))
```

### Custom Augmentations 
To define custom augmentations you can simply pass your combination of transformations to `aug_tfms` as a list. All available transformations and how to define custom transformations is described in the next section.


## Available Transformations
Each transformation is defined as a class inherited from the abstract parent class `Transform`. To define a transformation overwrite the abstract method `do_transform`. A transform may include a random component. If it does, it will often need to transform `y` using the same random values as `x` (e.g. a horizontal flip in segmentation must be applied to the mask as well). Therefore, the method `set_state` is used to ensure all random state is calculated in one place.

```eval_rst
.. autoclass:: Transform
   :members:
   
   .. automethod:: do_transform
   .. automethod:: set_state
```

There's also a Class `CoordTransform` inherited from `Transform` which is used to represent all coordinate based transformations like cropping, rotating and scaling.

```eval_rst
.. autoclass:: CoordTransform
   :members:
   
   .. automethod:: make_square
   .. automethod:: map_y
   .. automethod:: transform_coord
```

#### The currently implemented transformations are:

```eval_rst
.. autoclass:: Normalize
.. autoclass:: Denormalize
.. autoclass:: ChannelOrder
.. autoclass:: AddPadding
.. autoclass:: CenterCrop
.. autoclass:: RandomCrop
.. autoclass:: NoCrop
.. autoclass:: Scale
.. autoclass:: RandomScale
.. autoclass:: RandomRotate
.. autoclass:: RandomDihedral
.. autoclass:: RandomFlip
.. autoclass:: RandomLighting
.. autoclass:: RandomRotateZoom
.. autoclass:: RandomStretch
.. autoclass:: PassThru
.. autoclass:: RandomBlur
.. autoclass:: Cutout
.. autoclass:: GoogleNetResize
```

```eval_rst
.. note:: Transformations are often run in multiple threads. Therefore any state must be stored in thread local storage. The `Transform` class provide a thread local `store` attribute for you to use. See `RandomFlip` class for an example of how to use random state safely in `Transform` subclasses.
```
The class `TfmType` defines the type of transformation:

```eval_rst
.. autoclass:: TfmType
   :members:
```

