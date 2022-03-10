# Preparing Dataset

## Existing datasets (DND, SIDD, NIND)

You can download existing public datasets: [DND](https://noise.visinf.tu-darmstadt.de/), [SIDD](https://www.eecs.yorku.ca/~kamel/sidd/), [NIND](https://commons.wikimedia.org/wiki/Natural_Image_Noise_Dataset).  

Dataset directories follow as below:

```
AP-BSN
├─ dataset
│  ├─ DND
│  │  ├─ dnd_2017
│  │     ├─ images_srgb
│  ├─ SIDD
│  │  ├─ SIDD_Medium_Srgb
│  │  ├─ BenchmarkNoisyBlocksSrgb.mat
│  │  ├─ ValidationGtBlocksSrgb.mat
│  │  ├─ ValidationNoisyBlocksSrgb.mat
│  ├─ NIND
│  │  ├─ 7D-1
│  │  ...
│  │  ├─ whistle
│  ├─ prep
│  │  ├─ DND_s512_o128
│  │  │  ├─ RN
│  │  ├─ NIND_s512_o128
│  │  │  ├─ CL
│  │  │  ├─ RN
│  │  ├─ SIDD_benchmark_s256_o0
│  │  │  ├─ RN
│  │  ├─ SIDD_s512_o128
│  │  │  ├─ CL
│  │  │  ├─ RN
```

To prepare (crop image) above datasets, you should run below scripts:  

```
# Default patch size is 512x512 with 128 overlapping

# prepare DND dataset
python prep.py --dataset DND

# prepare SIDD dataset
python prep.py --dataset SIDD

# prepare SIDD benchmark dataset (benchmark have only patches of 256x256 size.)
python prep.py --dataset benchmark --patch_size 256 --overlap 0

# prepare NIND dataset
python prep.py --dataset NIND
```

## Custom dataset

If you want to train our AP-BSN on custom images, you should make custom dataset class in [custom.py](./custom.py) file. There is a skeleton code for this. Fill in _scan() and _load_data() functions.  

```
@regist_dataset
class CustomSample(DenoiseDataSet):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def _scan(self):
        # check if the dataset exists
        dataset_path = os.path.join('WRITE_YOUR_DATASET_DIRECTORY')
        assert os.path.exists(dataset_path), 'There is no dataset %s'%dataset_path

        # WRITE YOUR CODE FOR SCANNING DATA
        # example:
        for root, _, files in os.walk(dataset_path):
            for file_name in files:
                self.img_paths.append(os.path.join(root, file_name))

    def _load_data(self, data_idx):
        # WRITE YOUR CODE FOR LOADING DATA FROM DATA INDEX
        # example:
        file_name = self.img_paths[data_idx]

        noisy_img = self._load_img(os.path.join(self.dataset_path, 'RN' , file_name))
        clean = self._load_img(os.path.join(self.dataset_path, 'CL' , file_name))

        # return {'clean': clenan, 'real_noisy': noisy_img} # paired dataset
        # return {'real_noisy': noisy_img} # only noisy image dataset
```
In our code, we load images into [0, 255] range and RGB color order. Additionally, we recommend you to prepare your custom dataset similar with above procedure.  
After this, change training dataset name in configuration file. For example:

```
...

training:
  dataset: YOUR_CUSTOM_DATASET_NAME

  dataset_args:
    add_noise: None # e.g.) None bypass uni-15. gau-15. gau_blind-10.:50. het_gau-10.:50. see more detail in denoise_dataset.py
    crop_size: [120, 120]
    aug: ['hflip', 'rot']
    n_repeat: 1

...
```


