---
layout: post
category: develop
comments_id: 2
---

Lightning is designed with these principles in mind:

    Principle 1: Enable maximal flexibility.  
    Principle 2: Abstract away unecessary boilerplate, but make it accessible when needed.  
    Principle 3: Systems should be self-contained (ie: optimizers, computation code, etc).  
    Principle 4: Deep learning code should be organized into 4 distinct categories.

    - Research code (the LightningModule).
    - Engineering code (you delete, and is handled by the Trainer).
    - Non-essential research code (logging, etc... this goes in Callbacks).
    - Data (use PyTorch Dataloaders or organize them into a LightningDataModule).

대충 tf의 keras 같은 느낌인데, 필요시 세부 수정이 용이하다.

참고
* [Pair Programming: Deep Learning Model For Drug Classification With Andrey Lukyanenko](https://www.youtube.com/watch?v=VRVit0-0AXE)
* [Target Engineering; CV; ⚡ Multi-Target](https://www.kaggle.com/marketneutral/target-engineering-cv-multi-target)
* [Jane Street Market Prediction](https://www.kaggle.com/c/jane-street-market-prediction)

내 생각 기본 구성 4가지
1. DataLoader Class
2. PL DataModule
3. Model Class
4. PL Module
5. PL Trainer

```python
## 0) Env
import torch
import torch.nn as nn
import torch.nn.functional as F
import pytorch_lightning as pl
```

```python
## 1) Pytorch DataLoader
class JaneStreetDataset:
    def __init__(self, dataset, targets):
        self.dataset = dataset
        self.targets = targets

    def __len__(self):
        return self.dataset.shape[0]

    def __getitem__(self, item):
        return {
            'x': torch.tensor(self.dataset[item, :], dtype=torch.float),
            'y': torch.tensor(self.targets[item, :], dtype=torch.float)
        }
```

```python
## 2) PL DataModule
class JaneStreetDataModule(pl.LightningDataModule):
    def __init__(self, hparams, data, targets, fold=None, n_folds=5):
        super().__init__()
        self.hparams = hparams
        self.data = data
        self.targets = targets
        self.fold = fold

    def prepare_data(self):
        pass

    def setup(self, stage=None):


        # TODO: this should be a fold; see https://www.kaggle.com/krisho007/moa-pytorch-lightning-kfold
        n_samples = len(self.data)
        split_idx = int(n_samples*0.70)

        train_data = self.data.iloc[:split_idx, :]
        valid_data = self.data.iloc[(split_idx + 1):, :]

        train_targets = self.targets.iloc[:split_idx, :]
        valid_targets = self.targets.iloc[(split_idx + 1):, :]


        # TODO: this should be a fold
        self.train_dataset = JaneStreetDataset(
            dataset=train_data.values,
            targets=train_targets.values
        )

        # TODO: this should be a fold
        self.valid_dataset = JaneStreetDataset(
            dataset=valid_data.values,
            targets=valid_targets.values
        )

    def train_dataloader(self):
        train_loader = torch.utils.data.DataLoader(
            self.train_dataset,
            batch_size=BATCH_SIZE,
            num_workers=0,
            shuffle=False
        )
        return train_loader

    def val_dataloader(self):
        val_loader = torch.utils.data.DataLoader(
            self.valid_dataset,
            batch_size=BATCH_SIZE,
            num_workers=0,
            shuffle=False
        )
        return val_loader

    def test_dataloader(self):
        return None
```

```python
## 3) Model
class Model(nn.Module):
    def __init__(self, num_features, num_targets):
        super().__init__()

        ...

        for i in range(len(hidden_size)):
            out_size = hidden_size[i]
            layers.append(nn.Dropout(dropouts[i]))
            layers.append(nn.Linear(in_size, out_size))
            layers.append(nn.BatchNorm1d(out_size))
            layers.append(nn.SiLU())  # SiLU aka swish
            in_size = out_size

        ...

        self.model = torch.nn.Sequential(*layers)

    def forward(self, x):
        x = self.model(x)
        return x
```

```python
## 4) PL Module
class LitJaneStreet(pl.LightningModule):
    def __init__(self, hparams, model):
        super(LitJaneStreet, self).__init__()
        self.hparams = hparams
        self.model = model

        self.criterion = nn.BCELoss()

    def forward(self, x):
        return self.model(x)

    def configure_optimizers(self):
        optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
        scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
            optimizer, patience=3, threshold=0.00001, mode='min',
            verbose=True
        )
        return (
            [optimizer],
            [{'scheduler': scheduler,
              'interval': 'epoch',
              'monitor': 'valid_loss'
             }]
        )

    def training_step(self, batch, batch_idx):
        data = batch['x']
        target = batch['y']
        out = self(data)
        loss = self.criterion(out, target)

        logs = {'train_loss': loss}

        return {'loss': loss, 'log': logs, 'progress_bar': logs}

    def training_epoch_end(self, outputs):

        avg_loss = torch.stack([x['loss'] for x in outputs]).mean()
        logs = {'train_loss': avg_loss}
        return {'log': logs, 'progress_bar': logs}

    def validation_step(self, batch, batch_idx):

        data = batch['x']
        target = batch['y']
        out = self(data)
        loss = self.criterion(out, target)

        logs = {'valid_loss': loss}

        return {'loss': loss, 'log': logs, 'progress_bar': logs}

    def validation_epoch_end(self, outputs):

        avg_loss = torch.stack([x['loss'] for x in outputs]).mean()
        logs = {'valid_loss': avg_loss}
        return {'log': logs, 'progress_bar': logs}    
```

```python
## 5) PL Trainer
NUM_FEATURES = len(FEATURE_NAMES)
NUM_TARGETS = len(TARGET_NAMES)
GPU = 1  # 0 for off, 1 for on
EPOCHS = 5

trainer = pl.Trainer(
    gpus=GPU, max_epochs=EPOCHS, weights_summary='full'
)

## instantiation and run
net = Model(num_features=NUM_FEATURES, num_targets=NUM_TARGETS)
model = LitJaneStreet(hparams={}, model=net)
dm = JaneStreetDataModule(
    hparams={},
    data=train_data[FEATURE_NAMES],
    targets=train_data[TARGET_NAMES]
)

trainer.fit(model, dm)
```
