## 基于特征离散化的算法ClOUDS


CLOUDS（assification for Large or OUt-of-core　DataSets)）是基于直方图的算法，在建树之前，先对数值型特征进行离散化（分箱，bining)，这样一来，原始的浮点型特征，可以用整型表示，减少内存占用，减少计算量，减少IO．


### Split Finding

树节点分裂时，需要找到数值型特征的最佳切分点，前面介绍的SLIQ和SPRINT采用的是精确的方法，先对数值特征排序，然后遍历所有切分点，复杂度是O(nlogn)．也有一些方法先对数据抽样，然后再使用精确方法．而CLOUDS提出了两种近似直方图方法．

- Sampling the Splitting points (SS)

第一种方法叫SS，实际上就是将数值特征离散为q个区间，只考察分位点的分裂效果．区间的划分有两种方法，等频与等距．　等频划分得到的每个区间的distinct value个数相等，比较常用，例如LightGBM先统计每个特征的distinct value的个数，如果小于bin个数，则每个distinct value放入一个bin，如果ditinct　value个数多于bin个数，则求出每个bin应该放多少个ditinct value，LightGBM的这个做法可以保证相同的value的样本在同一个bin里．　等距划分得到的每个区间的宽度相等，容易受特征的数值分布影响，但在估计区间边界时会更快，因为它只需要找到特征的最大最小值，然后等间隔划分．

- Sampling the Splitting points with Estimation（SSE)

第二种方法叫做SSE，在SS的基础上进一步改进．　通过SS得到了所有分位点上的最小gini_min，然后估计每个区间里的最小gini_i，如果gini_i比gini_min小，则该区间标记为alive．最后对所有alive区间进行精确搜索切分点．


这两个方法都很简单，在具体实现时，我们需要保持每个区间的上边界(upper boundary)，这些就是要考察的切分点．那么怎么得到每个区间的上边界呢？如果对原始数据进行排序，其复杂度过高，所以CLOUDS先抽样得到原始数据的一个子集，再利用这个子集估计区间上边界，接着就可以对所有原始数据进行离散化了．并且，在测试阶段，同样需要边界信息，对测试数据进行特征分箱．

### 区间个数

将特征划分为q个区间，q的取值为多少合适呢? CLOUDS论文中实验表明，q的值在50~200之间时，得到的精度跟精确方法几乎一样．也就是说，假如我们选取q为256，则原始的用4个字节表示的浮点型数值(float为4字节，double为8字节)，现在只需要用１字节(８bit)．极大地减少内存占用．
