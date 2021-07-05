## Contributing
1. A python script `demo.py` that uses a pre-trained model to detect objects in a point cloud. (without downloading data | SUN RGB-D val set samples and pre-trained model provided in `demo` folder). 

### Run Demo
The pre-trained model with sample point clouds, RGB Images, Depth Maps, Camera Calib and the 2D bounding box detections are available in the demo folder. 
After completing installation below, Run:
```bash
python demo.py
```
The demo uses a pre-trained model (on SUN RGB-D) to detect objects in a point cloud from an indoor room (from SUN RGB-D val set). You can use 3D visualization software such as the MeshLab to open the dumped file under `demo/results` to see the 3D detection output. Specifically, open `***_pc.ply` and `***_pred_confident_nms_bbox.ply` to see the input point cloud and predicted 3D bounding boxes. Incase you want to check the class labels of the detected objects, set `inference_switch = True` in the second last line of `demo.py`. You can check the `***_pred_map_cls.txt` to get the class labels.

The ImVoteNet model needs the point cloud as well as the geometric, semantic and texture cues extracted from the RGB Image as input. `demo.py` creates a pipeline that inputs the RGB Image, Depth Map, Camera Calib and the 2D bounding box detections (Faster RCNN 2d object detection backbone output) to output all necessary inputs for ImVoteNet model in the right format. It further uses the pre-trained model to detect objects.

# ImVoteNet
**Boosting 3D Object Detection in Point Clouds with Image Votes**

<p align="center">
  <img src="http://xinleic.xyz/images/imvote.png" width="600" />
</p>

This repository contains the code release of the [paper](https://arxiv.org/abs/2001.10692):
```
@inproceedings{qi2020imvotenet,
  title={Imvotenet: Boosting 3d object detection in point clouds with image votes},
  author={Qi, Charles R and Chen, Xinlei and Litany, Or and Guibas, Leonidas J},
  booktitle={IEEE Conference on Computer Vision and Pattern Recognition (CVPR)},
  year={2020}
}
```

## Installation
Overall, the installation is similar to [VoteNet](https://github.com/facebookresearch/votenet). GPU is required. The code is tested with Ubuntu 18.04, Python 3.7.7, PyTorch 1.4.0, CUDA 10.0 and cuDNN v7.4.

First install [PyTorch](https://pytorch.org/get-started/locally/), for example through [Anaconda](https://docs.anaconda.com/anaconda/install/):
```bash
conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
```
Next, install Python dependencies via `pip` ([tensorboardX](https://github.com/lanpa/tensorboardX) is used for for visualizations):
```bash
pip install matplotlib opencv-python plyfile tqdm networkx==2.2 trimesh==2.35.39
pip install tensorboardX --no-deps
```
Now we are ready to clone this repository:
```bash
git clone git@github.com:marmotlab/imvotenet.git
cd imvotenet
```
The code depends on [PointNet++](http://arxiv.org/abs/1706.02413) as a backbone, which needs compilation:
```bash
cd pointnet2
python setup.py install
cd ..
```

## Data
Please follow the steps listed [here](https://github.com/facebookresearch/votenet/blob/master/sunrgbd/README.md) to set up the SUN RGB-D dataset in the `sunrgbd` folder. The expected dataset structure under `sunrgbd` is:
```
sunrgbd/
  sunrgbd_pc_bbox_votes_50k_{v1,v2}_{train,val}/
  sunrgbd_trainval/
    # raw image data and camera used by ImVoteNet
    calib/*.txt
    image/*.jpg
```
For ImVoteNet, we provide 2D detection results from a pre-trained Faster R-CNN detector [here](https://dl.fbaipublicfiles.com/imvotenet/2d_bbox/sunrgbd_2d_bbox_50k_v1.tgz). Please download the file, uncompress it, and place the resulting folders (`sunrgbd_2d_bbox_50k_v1_{train,val}`) under `sunrgbd` as well.

## Training and Evaluation

Once the code and data are set up, one can train ImVoteNet by the following command:
```bash
CUDA_VISIBLE_DEVICES=0 python train.py --use_imvotenet --log_dir log_imvotenet
```
The setting `CUDA_VISIBLE_DEVICES=0` forces the model to be trained on a single GPU (GPU `0` in this case). With the default batch size of 8, it takes about 7G memory during training. 

To reproduce the experimental results in the paper and in general have faster development cycles, one can use a shorter learning schedule: 
```bash
CUDA_VISIBLE_DEVICES=1 python train.py --use_imvotenet --log_dir log_140ep --max_epoch 140 --lr_decay_steps 80,120 --lr_decay_rates 0.1,0.1
```

As a baseline, this code also supports training of the original VoteNet, which is launched by:
```bash
CUDA_VISIBLE_DEVICES=2 python train.py --log_dir log_votenet
```
In fact, the code is based on the VoteNet repository at commit [2f6d6d3](https://github.com/facebookresearch/votenet/tree/2f6d6d3), as a reference, it gives around 58 mAP@0.25.

For other training options, one can use `python train.py -h` for assistance.

After the model is trained, the checkpoint can be tested and evaluated on the `val` set via:
```bash
python eval.py --use_imvotenet --checkpoint_path log_imvotenet/checkpoint.tar --dump_dir eval_imvotenet --cluster_sampling seed_fps --use_3d_nms --use_cls_nms --per_class_proposal
```
For reference, ImVoteNet gives around 63 mAP@0.25.

## TODO
- Add docs for some functions
- Investigate the 0.5 mAP@0.25 gap after moving to PyTorch 1.4.0. (Originally the code is based on PyTorch 1.0.)

## LICENSE

The code is released under the [MIT license](LICENSE).
