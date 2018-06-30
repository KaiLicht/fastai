# fastai.dataloader

## Introduction and Overview

```eval_rst
.. note:: the fastai DataLoader has a similar API to the PyTorch DataLoader. Please see .. the PyTorch documentation: http://pytorch.org/docs/master/data.html#torch.utils.data.DataLoader for usage and details. The documentation presented here focuses on the differences between the two classes.
```

## Class DataLoader
```eval_rst
.. automodule:: fastai.dataloader

.. autoclass:: DataLoader
   :members:
   :inherited-members:
```

### Methods

{{method jag_stack,b}}

Helper method for `np_collate()`. Returns a np.array of the batch passed in, with zeros added for any shorter rows inside the batch, plus extra zeros if `self.pad_idx > 0`. If all items inside the batch are the same length, no zero padding is added.

{{method np_collate,batch}}

Helper method for `get_batch()`. Based on the input data type, it creates an appropriate np.array, list, or dict. If the method is passed a string or list of strings, it simply returns the parameter without modification. Batches must contain numbers, strings, dicts, or lists, and this method also ensures this is the case.

{{method get_batch,indices}}

Helper method for `__iter__()`. When an iterator of the dataloader object is created, `get_batch()` is used to retrieve items from the dataset and apply transposes if needed based on `self.transpose` and `self.transpose_y`.
