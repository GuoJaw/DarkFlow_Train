CuDNNv6安装
	(1)#
	cd cuda/include
	sudo cp cudnn.h /usr/local/cuda/include/

	(2)# cd ../lib64
	sudo cp lib* /usr/local/cuda/lib64/
	cd /usr/local/cuda/lib64/
	(3)删除cudnn
	sudo rm -rf libcudnn.so libcudnn.so.*
	(4)重新软连接
	sudo ln -s libcudnn.so.6 libcudnn.so
	(5)更改权限
	sudo chmod a+x /usr/local/cuda/include/cudnn.h  /usr/local/cuda/lib64/libcudnn*

	1.安装Tensorflow（清华镜像）
		pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ https://mirrors.tuna.tsinghua.edu.cn/tensorflow/linux/gpu/tensorflow_gpu-1.4.0-cp27-none-linux_x86_64.whl
		
		pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple/ https://mirrors.tuna.tsinghua.edu.cn/tensorflow/linux/gpu/tensorflow_gpu-1.0.0-cp35-cp35m-linux_x86_64.whl

	
	【问题】导入tensorflow时报警告
	>>> import tensorflow
	/usr/local/lib/python2.7/dist-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
	  from ._conv import register_converters as _register_converters

	解决方案：
	sudo pip install h5py==2.8.0rc1


## Intro

    本程序用来训练yolo2模型，进行多目标检测

## Dependencies

    Python3, tensorflow 1.0, numpy, opencv 3.

## 一、环境搭建
    python3 setup.py build_ext --inplace
    pip install -e .
    pip install .

    opencv
        conda install -c https://conda.binstar.org/menpo opencv
        conda install --channel https://conda.anaconda.org/menpo opencv3

## 二、测试
    [1]摄像头
         ./flow --model cfg/yolo.cfg --load bin/yolo.weights   --demo camera  --threshold 0.5  --gpu 1.0
    [2]视频
       ./flow --model cfg/yolo.cfg --load bin/yolo.weights   --demo MOT16-13.mp4  --threshold 0.5  --gpu 1.0

## 三、训练自己的数据

    （1）数据集准备：只需要Annotations    JPEGImages
        drive
                Annotations
                JPEGImages
    放在darkflow目录下。
    （2）模型训练
    说明，训练前的准备工作，特别令我恶心，TMD：
        --batch   8           ####显存爆炸####

    [1]第一次训练：
    ./flow --model cfg/yolo_drive.cfg --load bin/yolo.weights --labels labels_drive.txt --train --annotation data/Annotations --dataset data/JPEGImages --gpu 1.0 --batch 4

    [2]接着上一次停止的chechpoint节点继续开始训练：
    ./flow --model cfg/yolo_drive.cfg --load  -1 --labels labels_drive.txt --train --annotation data/Annotations --dataset data/JPEGImages --gpu 1.0 --batch 4

    结果：训练的时候,会周期性地将模型保存到darkflow/ckpt/目录下

    （3）模型转换
    将训练生成的ckpt目录下的4个文件，转换成.pb和.meta文件
    ./flow --model cfg/yolo_drive.cfg --load  -1 --labels labels_drive.txt --savepb 

    (4)Test
    flow --pbLoad built_graph/yolo_drive.pb --metaLoad built_graph/yolo_drive.meta --labels labels_drive.txt --imgdir sample_img/
    [1]摄像头
         ./flow --pbLoad built_graph/yolo_drive.pb --metaLoad built_graph/yolo_drive.meta  --labels labels_drive.txt --demo camera  --threshold 0.5  --gpu 1.0
    [2]视频
       ./flow --pbLoad built_graph/yolo_drive.pb --metaLoad built_graph/yolo_drive.meta --labels labels_drive.txt --demo MOT13.mp4  --threshold 0.5  --gpu 1.0


       

## 四、测试精度darkflow（Yolov2）————mAP     
     参考：https://github.com/Cartucho/mAP
    【1】制作测试集：制作predicted
    (1)
    darkflow:
        ./flow --pbLoad built_graph/yolo_drive.pb --metaLoad built_graph/yolo_drive.meta --labels labels_drive.txt --imgdir data/JPEGImages/ --json
    说明：
    --imgdir data/JPEGImages/ 下面存放着测试集的图片
    执行结果:
        生成: 在目录data/JPEGImages/out/下生成大量的*.josn文件
    (2)将生成的.josn文件，复制到--->mAP/predicted/目录下
    (3)
        cd mAP/extra
        python convert_pred_darkflow_json.py
    执行结果:
        所有的*.json文件，都被转移到---->mAP/predicted/backup目录下
        在mAP/predicted/目录下生成*.txt

    【2】制作grouth-truth
    (1)darkflow-master/data/Annotations/*.xml ----> mAP/ground-truth/
    (2)
        cd mAP/extra
        python convert_gt_xml.py
    执行结果:
        所有的*.xml文件，都被转移到---->mAP/ground-truth/backup目录下
        在mAP/ground-truth/目录下生成*.txt
        
    【3】直接执行
        cd mAP
        python main.py


