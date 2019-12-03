+++
title = "Train-Validation-Test split in PyTorch"
author = ["Stanislav Arnaudov"]
description = "A short utility class for loading data into pytorch project"
date = 2019-10-20T00:00:00+02:00
keywords = ["machine-learning", "pytorch", "python", "data-loading"]
lastmod = 2019-10-31T12:01:23+01:00
tags = ["machine-learning", "python"]
categories = ["machine-learning"]
draft = false
weight = 100
+++

## Abstract {#abstract}

[PyTorch](https://pytorch.org/) is great! It offers tons of utilities that make every ML project a little bit less daunting. It's easy to have your DNN-model up and running in almost no time. At the same time, the framework is still relatively unopinionated and lets you decide on the exact structure of your project. I've been playing around with PyTorch recently and the one thing I've been missing so far is to be able to create a train-validation-test split of my data in an "out of the box" manner. It's not that it is impossible but it requires some boilerplate code. In this post, I present a small utility class for doing exactly this.


## The _DataSplit_ class {#the-datasplit-class}

The data loading process in PyTorch involves defining a dataset class that inherits from [data.Dataset](https://pytorch.org/docs/stable/data.html#torch.utils.data.Dataset). The class defines only what the data point at a given index is and how much data points there are. PyTorch can then handle a good portion of the other data loading tasks -- for example batching.

<br />

My utility class _DataSplit_ presupposes that a dataset exists. It takes a dataset as an argument during initialization as well as the ration of the train to test data (`test_train_split`) and the ration of validation to train data (`val_train_split`). The data can also be optionally shuffled through the use of the `shuffle` argument (it defaults to false). With the default parameters, the test set will be 20% of the whole data, the training set will be 70% and the validation 10%. To note is that `val_train_split` gives the fraction of the training data to be used as a validation set. The split is performed by first splitting the data according to the `test_train_split` fraction and then splitting the train data according to `val_train_split`.

<br />

The full source code of the class is in the following snippet.

```python
import logging
from functools import lru_cache

import torch
from torch.utils.data import DataLoader
from torch.utils.data import Dataset
from torch.utils.data.sampler import SubsetRandomSampler

import numpy as np

class DataSplit:

    def __init__(self, dataset, test_train_split=0.8, val_train_split=0.1, shuffle=False):
        self.dataset = dataset

        dataset_size = len(dataset)
        self.indices = list(range(dataset_size))
        test_split = int(np.floor(test_train_split * dataset_size))

        if shuffle:
            np.random.shuffle(self.indices)

        train_indices, self.test_indices = self.indices[:test_split], self.indices[test_split:]
        train_size = len(train_indices)
        validation_split = int(np.floor((1 - val_train_split) * train_size))

        self.train_indices, self.val_indices = train_indices[ : validation_split], train_indices[validation_split:]

        self.train_sampler = SubsetRandomSampler(self.train_indices)
        self.val_sampler = SubsetRandomSampler(self.val_indices)
        self.test_sampler = SubsetRandomSampler(self.test_indices)

    def get_train_split_point(self):
        return len(self.train_sampler) + len(self.val_indices)

    def get_validation_split_point(self):
        return len(self.train_sampler)

    @lru_cache(maxsize=4)
    def get_split(self, batch_size=50, num_workers=4):
        logging.debug('Initializing train-validation-test dataloaders')
        self.train_loader = self.get_train_loader(batch_size=batch_size, num_workers=num_workers)
        self.val_loader = self.get_validation_loader(batch_size=batch_size, num_workers=num_workers)
        self.test_loader = self.get_test_loader(batch_size=batch_size, num_workers=num_workers)
        return self.train_loader, self.val_loader, self.test_loader

    @lru_cache(maxsize=4)
    def get_train_loader(self, batch_size=50, num_workers=4):
        logging.debug('Initializing train dataloader')
        self.train_loader = torch.utils.data.DataLoader(self.dataset, batch_size=batch_size, sampler=self.train_sampler, shuffle=False, num_workers=num_workers)
        return self.train_loader

    @lru_cache(maxsize=4)
    def get_validation_loader(self, batch_size=50, num_workers=4):
        logging.debug('Initializing validation dataloader')
        self.val_loader = torch.utils.data.DataLoader(self.dataset, batch_size=batch_size, sampler=self.val_sampler, shuffle=False, num_workers=num_workers)
        return self.val_loader

    @lru_cache(maxsize=4)
    def get_test_loader(self, batch_size=50, num_workers=4):
        logging.debug('Initializing test dataloader')
        self.test_loader = torch.utils.data.DataLoader(self.dataset, batch_size=batch_size, sampler=self.test_sampler, shuffle=False, num_workers=num_workers)
        return self.test_loader
```

After an instance of the class is created, the `get_split` method can be used to get a tuple of three [data.DataLoader](https://pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader) objects -- one for the train, validation, and test sets. The method has a couple of keyword arguments for the batch size(`batch_size`) and the number of threads to be used while loading the data (`num_workers`). Both arguments are passed to the created _DataLoader_ object.

<br />

A possible use case is:

```python
class PlainDataSet(Dataset):
    def __init__(self):
        self.arr = np.array([1.0]*1000)
    def __getitem__(self, index):
        return np.array([self.arr[index]])
    def __len__(self):
        return len(self.arr)

dataset = PlainDataSet()
split = DataSplit(dataset, shuffle=True)
train_loader, val_loader, test_loader = split.get_split(batch_size=75, num_workers=8)

for iteration, batch in enumerate(train_loader,1):
    print('{} : {}'.format(iteration, str(batch.size())))
```
