# Random Shuffling Data for Hyperspectral Image Classification with Siamese and Knowledge Distillation Network
****
文章链接：https://www.mdpi.com/2072-4292/15/16/4078    （论文也可见项目内名为RS_样刊的PDF文件）  
****
## SKDN: Siamese and Knowledge Distillation Network
SKDN_TD presents the model using the data from the new dataset (Random Shuffle Data) to test.  
SKDN_OD presents the model using the data from the original dataset to test.
****
## Some details about this article  
### (1) All experiments were performed using three patch-data-based CNN methods in HSI classification, which are DPRN, SSTFF and MRViT.  
The paper name of the method DPRN:  
Deep Pyramidal Residual Networks for Spectral-Spatial Hyperspectral Image Classification.  
(Project Repository address: https://github.com/mhaut/pResNet-HSI)  
The paper name of the method SSTFF:  
Spectral-Spatial Feature Tokenization Transformer for Hyperspectral Image Classification.  
(Project Repository address: https://github.com/zgr6010/HSI_SSFTT)  
The paper name of the method MRViT:  
Mixed Residual Convolutions with Vision Transformer in Hyperspectral Image Classification.  
(Project Repository address: https://github.com/Candy-CY/Hyperspectral-Image-Classification-Models/tree/main/MRViT)  
  
### (2) The Code of Random Shuffling Strategy.  
```
# We designed a random shuffling strategy to disrupt the data homogeneity of the patch data, 
# which is randomly assigning the pixels from the original dataset to other positions to form a new dataset.
X, y = loadData()
print('Hyperspectral data shape: ', X.shape)
print('Label shape: ', y.shape)
print('\n... ... Random Shuffle Data and Label... ...')
H,W,B = X.shape
H1,W1 = y.shape
Data = X.reshape(H*W,B)
label = y.reshape(H1*W1,-1)
# 合并数据和标签
finalData = np.hstack((Data,label))
print("finalData is :\n",finalData.shape)
# 随机打乱合成的新数据finalData的百分之多少的数据集
num_shuffle = int(finalData.shape[0] * 0.2) #0.2是随机打乱的比例数
print("num_shuffle:\n",num_shuffle)
shuffle_idx = random.sample(range(finalData.shape[0]), num_shuffle)
# 随机打乱需要打乱的数据
shuffle_data = np.random.permutation(finalData[shuffle_idx,:])
# 将随机打乱的数据替换到原向量中
finalData[shuffle_idx,:] = shuffle_data[:,:]
# 取出打乱后的标签数据，并在打乱的数据中删除标签列
ly = finalData[:,[-1]]
Dx = np.delete(finalData,-1,axis=1)
NewData = Dx.reshape(H,W,B)
Newlabel = ly.reshape(H1,W1)
```
  
### (3)The Code of RFL (fuse the loss rates).   
```
# RFL: fuse the loss rates. The loss rate calculated by RFL in the new patch data 
# is cross combined with the loss value calculated by another sub-branch in the original patch data.
for epoch in range(epochs):
     net.train()
     for i, (data, target) in enumerate(train_loader):#train_loader:原始数据集中的数据
         data, target = data.to(device), target.to(device)
         outputs = net(data)
         # 计算损失函数
         lossi = criterion(outputs, target)
         for j, (dataj, targetj) in enumerate(train_loader_Random):#train_loader_Random:打乱数据集中的数据
             dataj, targetj = dataj.to(device), targetj.to(device)
             outputsj = net(dataj)
             lossj = criterion(outputsj, targetj)
             loss = lossi + 0.5*(lossj)
         # 优化器梯度归零
         optimizer.zero_grad()
         loss.backward()
         optimizer.step()
         total_loss += loss.item()
```
