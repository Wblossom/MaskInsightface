##insightface制作自己的数据及其训练
ssh(人脸检测)+prnet(68 landmark 人脸对齐， 3d人脸mask)+insightface
# 0.安装
(1)mxnet
(2)tensorflow
# 生成对齐后的数据集
python3 gen_datasets.py  #完成后会output下生成train.lst

cd make_rec
# 1.生成,'.lst, .rec, .idx, property'
(1)为了合并数据，可采用generate_lst.sh
(2)property是属性文件，里面内容是类别数和图像大小，例如
1000,112,112 其中1000代表人脸的类别数目，图片格式为112x112（一直都不知道怎么自动生成这个，我是自己写的）
(3)generate_lst.sh
# 2.生成测试文件.bin
python gen_valdatasets.py

验证model精度：
# 1.在bash
python3 -u ./src/eval/verification.py --gpu 0 --model "./models/glint-mobilenet/model,1" --target 'lfw','model_ours'
# 2.快捷
sh verification.sh


训练：
# 1.在bash里面训练
CUDA_VISIBLE_DEVICES='2,3,4,5' python3 -u train.py --network r100 --loss arcface --per-batch-size 64 2>&1 > log.log &


#如果想要合并不同数据集
CUDA_VISIBLE_DEVICES=0 python3 src/data/dataset_merge.py --include 001_data,002_data --output ms1m+vgg --model ../../models/model-glink-gate/model,1

# result
network backbone: r100 ( output=E, emb_size=512, prelu )
loss function: arcface(m=0.5)
lr_steps [105000, 125000, 150000], end with 180001, batch-size:256, 4gpu
then retrain with lr = 0.01, lr_steps[200000, 300000, 400000]
one LFW: 99.82; CFP_FP: 98.50; AgeDB30: 98.25
