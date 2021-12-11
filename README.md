# SimSwap-train
Reimplement of SimSwap training code<br />
- 20210919 这份代码原本是中秋节的时候写的；<br />
- 20211130 后来我们团队有换脸相关需求了，做了很多改进与优化，不过应该没法分享出来；<br />
- 20211211 我把这份代码的使用文档更新了一版，512pix是可以训的，希望能帮助到各位。<br /><br /><br />

# Instructions
## 1.Environment preparation
### Step 1.Install packages for python
1) Refer to the [SIMSWAP preparation](https://github.com/neuralchen/SimSwap/blob/main/docs/guidance/preparation.md) to install the python packages.<br />
2) Refer to the [SIMSWAP preparation](https://github.com/neuralchen/SimSwap/blob/main/docs/guidance/preparation.md) to download the 224-pix pretrained model (for finetune) or none and other necessary pretrained weights.<br /><br />
### Step 2.Modify the ```insightface``` package to support arbitrary-resolution training
- If you use CONDA and conda environment name is ```simswap```, then find the code in place: <br />
 `C://Anaconda/envs/simswap/Lib/site-packages/insightface/utils/face_align.py`<br /><br />
change #line 28 & 29:<br />
`src = np.array([src1, src2, src3, src4, src5])`<br />
`src_map = {112: src, 224: src * 2}`<br />
into<br />
`src_all = np.array([src1, src2, src3, src4, src5])`<br />
`#src_map = {112: src, 224: src * 2}`<br /><br />
change #line 53:<br />
`src = src_map[image_size]`<br />
into<br />
`src = src_all * image_size / 112`<br /><br />
After this, you can extract faces of any resolution and pass them to the model for training. <br /><br />
- If you don't use CONDA, just find the location of the package and change the code in the same way as above.<br /><br /><br /><br />



## 2.Preparing training data
- Put all the image files(`.jpg`, `.jpeg`, `.png` and `.bmp` supported) in your datapath (eg. `./dataset/CelebA`)
- Run the commad with :<br />
`python make_dataset.py --dataroot ./dataset/CelebA --extract_size 512 --output_img_dir ./dataset/CelebA/imgs --output_latent_dir ./dataset/CelebA/latents`<br /><br />
- When processing done, the `cropped face images` and `net_Arc embedded latents` will be recored in the `output_img_dir` and `output_latent_dir` directories.<br /><br /><br /><br />

## 3.Start Training
- Finetuning, run command with:<br />
`CUDA_VISIBLE_DEVICES=0 python train.py /`<br />
&emsp;&emsp;`--name CelebA_512_finetune /`<br />
&emsp;&emsp;`--which_epoch latest /`<br />
&emsp;&emsp;`--dataroot ./dataset/CelebA /`<br />
&emsp;&emsp;`--image_size 512 /`<br />
&emsp;&emsp;`--display_winsize 512 /`<br />
&emsp;&emsp;`--continue_train`<br /><br />
NOTICE:<br />
&emsp;&emsp;If `chekpoints/CelebA_512_finetune` is an un-existed folder, it will first copy the official model from `chekpoints/people/*` to `chekpoints/CelebA_512_finetune/`.<br /><br />

- Or New training, run command with:<br />
`CUDA_VISIBLE_DEVICES=0 python train.py /`<br />
&emsp;&emsp;`--name CelebA_512 /`<br />
&emsp;&emsp;`--which_epoch latest /`<br />
&emsp;&emsp;`--dataroot ./dataset/CelebA /`<br />
&emsp;&emsp;`--image_size 512 /`<br />
&emsp;&emsp;`--display_winsize 512 /`<br /><br />

- When training is done, you will see some files in `chekpoints/CelebA_512_finetune` folder:<br /><br />
`web/`: training-process visualization files<br />
`latest_net_G.pth`: Latest checkpoint of G network<br />
`latest_net_D1.pth`: Latest checkpoint of D1 network<br />
`latest_net_D2.pth`: Latest checkpoint of D2 network<br />
`loss_log.txt`: Doc to record loss during whole training process<br />
`iter.txt`: Doc to record iter information<br />
`iter.txt`: Doc to record options for the training<br />
<br /><br /><br /><br />


## 4.Training Result
### （1）CelebA with 224x224 res
![Image text](https://github.com/a312863063/SimSwap-train/blob/main/docs/img/train_celeba_224.png)

### （2）CelebA with 512x512 res
![Image text](https://github.com/a312863063/SimSwap-train/blob/main/docs/img/train_celeba_512_1.png)
![Image text](https://github.com/a312863063/SimSwap-train/blob/main/docs/img/train_celeba_512_2.png)


## 5.Inference
&emsp;&emsp;I applied spNorm to the high-resolution image during training, which is conducive to the the model learning. Therefore, during Inference you need to modify<br />
&emsp;&emsp;`swap_result = swap_model(None, frame_align_crop_tenor, id_vetor, None, True)[0]`<br />
&emsp;&emsp;to <br />
&emsp;&emsp;`swap_result = swap_model(None, spNorm(frame_align_crop_tenor), id_vetor, None, True)[0]` <br />

# Inference result
&emsp;&emsp;The demo presented below uses modified architecture and with many training optimizations, and I'm sorry I cannot share that for the business issues.<br />
![Image text](https://github.com/a312863063/SimSwap-train/blob/main/docs/img/apply_example.jpg)
&emsp;&emsp;Watch video here:<br />
&emsp;&emsp;Video file here: ```docs/apply_example.mp4```<br /><br />


