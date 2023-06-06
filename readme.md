# plan 2  调用PS 接口方式  生成困难样本
# 关键字  透明度边缘！！  PS脚本生成!!
* 设置了最终的双边缘gt的阈值200，而非原本的255才是篡改区域，理论上和实际的篡改区域的重合度很高，但是由于是透明度的像素，也是一种伪命题
* 全自动脚本已经完成，自动关闭所有打开的PSD 文件，不需要人工操作关闭文件,加入一个命令即可

* 出错的概率大约为1%

* __减少人力的介入__ 突然觉得做一个自动脚本每次跑50个，跑2轮 用起来比较舒服，电脑的内存可以跑50个，不去操作ps.(PS必须操作,ps存在叫做暂存盘的东西，不能够一次打开过多的PS文件，一次100个最多)
也就是说，每次跑100张图片，中间不需要人工操作，结束后删除所有PS打开的 文件，再点击运行按钮。
比之前的方案人工操作的次数高效了4倍

* 在使用之前把调色板的颜色调为黑色  （0 0 0） ,否则 ,填充gt的颜色会出错

* 重点是边缘的透明度要使得网络检测不出篡改痕迹

* 这个方法很复杂,效率低了一点，大概是最优方法的1/3效率。最优方法的思路是在同一个框架里面把所有事情处理完毕

* 半自动脚本，电脑旁边配个人，点按钮和删除PS文件,每循环一次输入一个"y"

* 但是使用只需要运行all_actions.py 即可 .但是，前提是文件夹和PS动作要配置到对应的文件夹和PS动作文件中

* PS动作文件为 plan2 目录下的”默认动作.atn“ 需要导入到PS中

* 工程所需 __环境 库__ 在同目录下requirements.txt中

* 本方法是基于PS开发的，计算机必须安装合适版本的PS软件

* 每次运行暂停时，根据提示输入'y' ,否则程序终止

* 对于本身标注数据集的实例 贴边的那种会出错的，需要矫正（已矫正）

* 发现魔棒设置有不连续的选项，可以通过设置魔棒的属性，添加到动作记录中，使得gt找出不连续部分的黑色部分

* 发现由于物体位置分布的不确定性，可能会有刚好选择选取时，魔棒种子点点击到了物体部分，此时只能够多运行几次

## plan2 基本思路

* 调用ps接口 解决实际的操作问题 ，结果保存为psd文件,但是这个东西不方便关闭
* 两次使用psd_tools 保存psd分别为tempered picture和 gt_mask1
* 在实际PS图像操作中，使用Ps动作命令，这个操作要求高度的模式化

## 实际操作
* __add_layer.py__  :  photoshop库 用于图层的叠加   使得mask和src在同一个psd中
* __PS_acitons.py__  :  win32com库  似乎是用于系统调用的库，可以调用一些软件
在这里有PS动作的使用
* __get_gt.py__  : 有win32com库继续使用脚本做出篡改的掩膜，使用已有的普通mask 生成双边缘的gt
* __psd2png.py__  : psd_tools库 用途是把psd转为png,且只有可见图层
* __all_actions.py__ : 对上面所有的函数进行集成 ，只要运行这个就行了,但是ps 动作运行起来不稳定，当心
* __delete_pic.py__  : 为了解决ps动作的不稳定性，此脚本是删除不合规的图片
## 使用的三种 PS操作库
* photoshop库 https://github.com/loonghao/photoshop-python-api
* win32com库   系统级的库
* psd_tools库   https://blog.csdn.net/qq_33337811/article/details/103036113


 对同一件事情调用三种功能相近的库，我觉得有点蠢。我相信其中的任何一个库都可以独立完成这件事
 
 这个__photoshop库__ 是2021年6月写的，但是作者很热情，给出了examples，不懂的可以提问他
 
## 对于cod10k全体数据集生成篡改图
* 会有错误
* 若进行了reshape,错误主要是在于设计的动作点击了黑色区域以外的地方，比较刁钻的mask会取到这个位置.难以避免，总会有一些数据集的图片白色标注部分刚好在那个位置
* 就内存极限而言，同时处理50张图片没有问题，但是会把内存占用90%以上，还是25个每次比较好
* 使用delete_pic.py 进行除错
* 对于整张图片进行移动的图片，认为应当保留，
网络会检测出来的边缘是移动的方形边缘和新的图片中的实例，
原则上讲新的图片中的实例是不能够算做是篡改的，边缘应当是完全符合周围环境的。（这种情况在现有方案中已经不存在了）
* 若不进行reshape,图片的利用率很低  只有一半的图片有用  其他会因为报错被删除  报错原因是不符合设置的PS动作模式  
图片大小  mask位置等因素都会导致ps处理图片时报错.
* photoshop.api.errors.PhotoshopPythonAPIError: Please check if you have Photoshop installed correctly.
像是这种错误就是卡bug的问题，将cod10k中导入图片去掉，停止程序后，运行all_actions.py和delete_pic.py 即可。
* 所有 的动作都是对应 的320*320的图片大小的。若要改大小，则自行录入动作。
* 在ps运行脚本时，不能操作PS,否则会报错误。进程冲突
* 在每个循环结束后，操作PS，PS主界面-选择文件-关闭全部-不保存-应用全部
* 如果图片进行自动篡改的效果 和预期不同，那么就多运行几次相同的图片

### reshape 到320*320 的合理性
* ps脚本的 __高度模式化__ 使得处理不同大小的图片时候，有概率会有选区落在图片外的情况，此时就会报错了。
而操作错误必须人工把报错弹窗关闭，为了节约人力，把图片先进行reshape 就很重要了。
* 同时，相同大小的图片有助于训练。似乎在什么论文里面看到这个理论

### “默认动作.atn”

* 使用脚本之前导入PS动作中

* 尽量给320*320的图片

通过PS记录动作手工录入，这里录入了3个动作，向左缩小-向右缩小-向左放大

给的魔棒取样点不完全相同，至少三个动作同时取到物体实例的情况的几乎没有的

* test数据集共2000张图片，做的train 数据集可能原理上有点问题，在生成最终的双边缘gt时候，判定有问题，但是效果概率会差1个像素，也有可能不存在误差