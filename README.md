<p align="center">
    <br>
<img src='https://github.com/elasticdeeplearning/artwork/blob/master/horizontal/color/edl-horizontal-color.png' width = "500">
    <br>
<p>

<h2 align="center">Motivation</h2>

Computing resources on cloud such as [Amazon AWS](https://aws.amazon.com/cn/)、[Baidu Cloud](https://cloud.baidu.com/) have multi-tenancy. Deep learning model training and inference with elastic resources will be common on cloud. We propose Elastic Deep Learning (EDL) that makes training and inference of deep learning model on cloud easier and more efficient.

Now EDL is an incubation-stage project of the [LF AI Foundation](https://lfai.foundation).


<p align="center">
    <br>
<img src='https://github.com/lfai/artwork/blob/master/lfai-project-badge/incubation/color/lfai-projectlogos_incubation-color.png' width = "200">
    <br>
<p>

<h2 align="center">Installation</h2>

You can install with ```pip install paddle_edl```. But we highly **recommend** you use it in our docker:

```
docker pull hub.baidubce.com/paddle-edl/paddle_edl:latest-cuda10.0-cudnn7
nvidia-docker run -name paddle_edl hub.baidubce.com/paddle-edl/paddle_edl:latest-cuda10.0-cudnn7 /bin/bash
```  

This project contains EDL framework and its applications such as distillation and NAS.

# EDL Applications:
## EDL Distill Training
<p align="center">
    <img src="doc/distill.gif" width="700">
</p>
The distillation training consists of two parts, teacher and student. Generally, the teacher and student are placed on the same card.
EDL distillation training decouple teacher and student, treats teacher as a inference service.
Students send sample data to the teacher and obtain prediction results for training.

### Quick Start: Run with fixed teacher in docker
1. First, you need deploy teacher.
``` bash
cd example/distill/mnist_distill
python -m paddle_serving_server_gpu.serve \
  --model mnist_cnn_model \
  --port 9292 \
  --gpu_ids 0
```
2. Run student.
``` python
python train_with_fleet.py \
  --use_distill_service True \
  --distill_teachers 127.0.0.1:9292
```
**A complete example is [here](./mnist_distill).**
More detail usage, please see [Run EDL distillation training](./example/distill/README.md)

# EDL Framework
## How to change from a normal train program to an EDL train program
The main change is that you should `load_checkpoint` at the beginning of training and `save_checkpoint` at the end of every epoch and the checkpoint should be on a distributed file system such as HDFS so all trainers can download from it. A complete example is [here](https://github.com/elasticdeeplearning/edl/tree/develop/example/collective/resnet50)

```
fs=HDFSClient(args.hdfs_name, args.hdfs_ugi,20*60*1000, 3 * 1000)
        
train_status =TrainStatus()
tmp_s = fleet.load_checkpoint(exe, args.checkpoint, fs=fs, trainer_id=trainer_id)
if tmp_s is not None:
   train_status = tmp_s
        
for pass_id in range(train_status.next(), params["num_epochs"]):
    train()
    
    if trainer_id == 0:
        saved_status = TrainStatus(pass_id)    
        fleet.save_checkpoint(exe, train_status=saved_status,
            path=args.checkpoint, fs=fs)
```

## Quickstart
### EDL Resnet50 experiments on a single machine in docker:

1. Start a JobServer on one node which generates changing scripts.
 
```
cd example/demo/collective	
./start_job_server.sh
```

1. Start a Jobclient which controls the worker process.

```
#Set the ImageNet data path
export PADDLE_EDL_IMAGENET_PATH=<your path>
#Set the checkpoint path
export PADDLE_EDL_FLEET_CHECKPOINT_PATH=<your path>

mkdir -p resnet50_pod
./start_job_client.sh
```

1. Experiments result
 
| total batch size | acc1 | acc5 |
| :-----: | ----: | ----: |
| 1024 | 75.5 | 92.8 |


## FAQ

TBD

## License

EDL is provided under the [Apache-2.0 license](LICENSE).
