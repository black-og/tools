# Augmentor简介
Augmentor是一个用于在机器学习中进行图像增强的Python库，使我们可以很方便的进行图像增强等操作。文档简洁，函数接口也很直观。支持多线程操作，此外，**它还支持将图像和与图像对应的groundtruth图像成对进行操作，以保证图像和它对应的groundtruth图像可以进行完全相同的操作**，这对于语义分割等任务来说非常便利。而且，它还允许用户自定义自己的操作，这里主要介绍Augmentor一下常用的自带的操作，关于如何自定义操作以及其他方面的详细信息，请自行阅读文档。
# 文档与github
文档：[http://augmentor.readthedocs.io/](http://augmentor.readthedocs.io/)

github地址：[https://github.com/mdbloice/Augmentor](https://github.com/mdbloice/Augmentor)
# 安装
Augmentor库的安装非常简单，在命令行中通过```pip```命令即可完成安装：

```pip install Augmentor```
# 快速入门
Augmentor通过建立一个pipeline来工作，其中pipeline中定义了在一些图像上进行的一系列操作。
首先，用图像所在目录来实例化一个Pipeline对象：

```Python
import Augmentor
p = Augmentor.Pipeline("/path/to/image")
```

然后，可以直接调用Pipeline对象支持的操作，将其加入到当前的pipeline中，例如：

```Python
p.rotate(probability=0.7, max_left_rotation=10, max_right_rotation=10)
p.zoom(probability=0.5, min_factor=1.1, max_factor=1.5)
```

你也可以通过调用`status()`函数查看当前pipeline的状态，例如：

```p.status()```

![](https://i.imgur.com/sVc8td8.png)

利用`status()`函数显示出的pipeline中所有操作的索引，调用`remove_operation(operation_index)`函数即可将指定的索引对应的操作从当前pipeline中移除。

如果你想使得图像和与其对应的groundtruth图像一起进行相同的操作，只需通过`ground_truth`函数将groundtruth图像所在目录添加到pipeline即可，即：

```p.ground_truth("/path/to/ground_truth_images")```

然后再调用各种图像增强函数即可实现图像与groundtruth图像成对进行操作的效果。

一旦你创建好了你的pipeline，你可以通过下面的函数来将图像增强后的得到的新图像存储到硬盘上：

```p.sample(n)```

其中n表示你想要得到的新图像的数目。

这只是一些常见的简单操作，Pipeline对象支持的操作很多，下面来对其中的函数进行详细的介绍。

# Pipeline对象的相关操作
* ```status()```

	输出pipeline的状态，包括pipeline中操作的数目和每个操作的索引，每个操作的参数，图像的数目，还有图像的性质，例如图片大小和格式等
		
		参数：无
		返回值：无
* ``` add_further_directory(new_source_directory, new_output_directory=u'output')```

	添加另一个你希望进行数据增强的目录
		
		参数： new_source_directory(String) - 待增强图像的目录
			 new_output_directory(String) - 生成图像的输出目录
		返回值：无
* ```add_operation(operation)```

	向pipeline中添加一个操作，可以用来添加自定义操作

		参数：operation(Operation) - 希望添加到pipeline中的operation对象
		返回值：无
* ```remove_operation(operation_index=-1)```

	移除由operation_index指定的操作，默认为-1，表示移除最后一个操作。操作的索引可以通过```status()```函数得到。
		
		参数：operation_index(Integer) - 要移除的操作的索引
		返回值：无
* ```sample(n, multi_threaded=True)```

	从当前的pipeline中随机选取n张图像，存储到output目录中。默认使用多线程进行加速，然而，如果图片非常小，可能会导致速度反而减慢，此时应设置为multi_threaded为False关闭多线程

		参数：n(Integer) - 产生新样本的数目
			multi_threaded(Boolean) - 是否使用多线程处理图片，默认为True
		返回值：无
* **Groundtruth**
	* ```ground_truth(ground_truth_directory)```
	
		指定和图像对应的groundtruth目录，将图像与groundtruth目录下与其同名的groundtruth图像关联起来。因此，图像与其对应的groundtruth图像需同名

			参数：ground_truth_directory(String) - groundtruth图像的目录
			返回值：无
	* ```get_ground_truth()```
	
		返回groundtruth路径
* **Croping**
	* ```crop_by_size(probability, width, height, centre=True)```
	
		根据width和height裁剪图像，默认从图像的中心进行裁剪。若centre=False，则在图像的随机位置进行裁剪。如果裁剪区域超出了图像的大小，则裁剪出整个图像

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				width(Integer) - 希望裁剪的宽度
				height(Integer) - 希望裁剪的高度
				centre(Boolean) - 如果为True，则从图像中心进行裁剪，否则，从图像的随机位置进行裁剪
			返回值：无
		注：crop操作会改变图像的大小！！！`crop_by_size`，`crop_center`和`crop_random`都是如此
	* ```crop_centre(probability, percentage_area, randomise_percentage_area=False)```
	
		以percentage_area指定的比例为大小在图像中心对图像进行裁剪

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				percentage_area(Float) - 需要裁剪的区域大小占原图像大小的比例
				randomise_percentage_area(Boolean) - 如果为True，则在图像的中心位置，随机选取0到percentage_area之间的区域大小进行裁剪
			返回值：无
	* ```crop_random(probability, percentage_area, randomise_percentage_area=False)```
	
		以percentage_area指定的比例为大小，在图像的随机位置进行裁剪

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				percentage_area(Float) - 需要裁剪的区域大小占原图像大小的比例
				randomise_percentage_area(Boolean) - 如果为True，则在图像的随机位置，随机选取0到percentage_area之间的区域大小进行裁剪
			返回值：无
* **Flip(Mirror)**

	* ```flip_left_right(probability)```

		将图像沿着水平方向进行翻转

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
	* ```flip_top_bottom(probability)```
	
		将图像沿着垂直方向进行翻转

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
	* ```flip_random(probability)```
	
		随机选取水平或者垂直方向对图像进行翻转

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
* **Rotate**
	* ```rotate(probability, max_left_rotation, max_right_rotation)```

		随机选取度数，旋转图像。max\_left\_rotation和max\_right\_rotation控制度数的范围。实际上，由于旋转之后要进行裁剪，因此，当旋转度数过大时，可能会导致图像不正确，所以max\_left\_rotation和max\_right\_rotation不能超过25。此外，当使用该函数导致图像不正确时，调小它们旋转的角度的范围。

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				max_left_rotation(Integer) - 图像能够向左旋转的最大角度，取值在0到25之间
				max_right_rotation(Integer) - 图像能够向右旋转的最大角度，取值在0到25之间
			返回值：无
	* ```rotate90(probability)```

		以一定的概率将图像旋转90度

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
	* ```rotate180(probability)```

		以一定的概率将图像旋转180度

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
	* ```rotate270(probability)```

		以一定的概率将图像旋转270度

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
	* ```rotate_random_90(probability)```

		随机选择90、180或270度，对图像进行旋转

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			返回值：无
	* ```rotate_without_crop(probability, max_left_rotation, max_right_rotation, expand=False)```

		将图像随机旋转一定角度，并且不进行裁剪。其中max_left_rotation和max_right_rotation控制旋转度数范围。expand默认为False，表示不对图像区域进行扩大，不必将旋转后的图像都完全包含在图像区域中；若expand为True，则表示对图像区域进行扩大，使得旋转后的图像都完全包含在图像区域中

		注：因为不对旋转后的图像进行裁剪，不会出现图像不正确的情况，因此不必限制max_left_rotation和max_right_rotation的大小

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				max_left_rotation(Integer) - 图像能够向左旋转的最大角度，取值不应小于0
				max_right_rotation(Integer) - 图像能够向右旋转的最大角度，取值不应小于0
				expand(Boolean) - 是否将图像区域进行扩大，使得包含旋转后的图像被完全包含在图像区域中，默认为False
			返回值：无

* **Distortion**
	* ```random_distortion(probability, grid_width, grid_height, magnitude)```

		在图像中随机加入畸变
	
			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				grid_width(Integer) - 网格的宽度，取值最好在2到10之间
				grid_height(Integer) - 网格的高度，取值最好在2到10之间
				magnitude(Integer) - 畸变的程度，取值最好在1到10之间
			返回值：无
			注：参数超出上述建议的范围，可能会导致不可预测的行为
	* ```gaussian_distortion(probability, grid_width, grid_height, magnitude, corner, method, mex=0.5, mey=0.5, sdx=0.05, sdy=0.05)```

		在图像中随机加入高斯畸变
* **Shear**

	* ```shear(probability, max_shear_left, max_shear_right)```

		随机选取角度，对图像沿x轴或者y轴进行错切，max\_shear\_left和max\_shear\_right表示角度的变化范围，它们应该在0到25之间，道理同```rotate```操作

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				max_shear_left(Integer) - 向左错切的最大角度，取值应在0到25之间
				max_shear_right(Integer) - 向右错切的最大角度，取值应在0到25之间
			返回值：无
* **Skew**

	* ```skew(probability, magnitude=1)```

		随机选取一个方向对图像进行skew，共有十二种情况，如下图所示：
	
		![](https://i.imgur.com/P0UQoa6.png)
		![](https://i.imgur.com/PzOoXku.png)
		![](https://i.imgur.com/6KC6Xxk.png)
	
			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				magnitude(Float) - 进行skew操作的最大值，取值应在0.1到1.0之间
			返回值：无
	* ```skew_left_right(probability, magnitude=1)```
	
		随机选取左和右中的一个方向进行skew
	
			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				magnitude(Float) - 进行skew操作的最大值，取值应该在0.1到1.0之间
			返回值：无 
	* ```skew_top_bottom(probability, magnitude=1)```

		随机选择前和后中的一个方向进行skew
	
			参数：probability (Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				magnitude(Float) - 进行skew操作的最大值，取值应该在0.1到1.0之间
			返回值：无
	* ```skew_tilt(probability, magnitude=1)```

		随机选择前、后、左和右中的一个方向进行skew
	
			参数：probability(Float) - 0 到1之间的值，表示在图像上进行这个操作的概率
				magnitude(Float) - 进行skew操作的最大值，取值应该在0.1和1.0之间
			返回值：无
	* ```skew_corner(probability, magnitude=1)```

		随机选择八种corner方向中的一种(即上图中后面八种情况)进行skew
	
			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				magnitude(Float) - 进行skew操作的最大值，取值应该在0.1到1.0之间
			返回值：无
* **Zoom**
	* ```zoom(probability, min_factor, max_factor)```

		随机放大图像，同时保持图像大小不变，放大倍数的范围由min\_factor和max\_factor控制。相当于先放大图像，然后以图像中心位置为中心，裁剪与原图像大小相同区域。注意和```crop```功能的区别
	
			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				min_factor(Float) - 图像放大范围的下界
				max_factor(Float) - 图像放大范围的上界
			返回值：无
		注：min\_factor和max\_factor可以小于1，此时会缩小图像，然后在边界处补像素值0，使得图像大小不变
	* ```zoom_random(probability, percentage_area, randomise_percentage_area=False)```

		在随机位置放大图像，放大程度由percentage_area控制。randomise_percentage_area默认为False，此时放大程度一定，即为percentage_area的值；若为True，则随机选择0到percentage_area之间的数值进行放大。
	
			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				percentage_area(Float) - 表示将大小与原图像大小比例为percentage_area的区域剪切出来，然后放大到原图像大小
				randomise_percentage_area(Boolean) - 默认为False，表示是否随机确定放大程度
			返回值：无
	
		注：**percentage_area越小，表示放大倍数越大**
* **Others**
	* ```random_brightness(probability, min_factor, max_factor)```
		
		随机改变图像的亮度

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				min_factor(Float) - 取值在0到max_factor之间，为调整图像亮度范围的最小值。为0时，图像调整为全黑；为1时，调整后和原图相同；大于1时，调整后的亮度大于原图亮度
				max_factor(Float) - 取值应大于min_factor，为调整图像亮度范围的最大值。为0时，图像调整为全黑；为1时，调整后和原图相同；大于1时，调整后的亮度大于原图亮度
			返回值：无
	* ```random_color(probability, min_factor, max_factor)```
	
		随机改变图像的饱和度

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				min_factor(Float) - 取值在0到max_factor之间，为调整图像饱和度范围的最小值。为0时，图像调整后为黑白图像；为1时，调整后和原图相同
				max_factor(Float) - 取值应大于min_factor，为调整图像饱和度范围的最大值。为0时，图像调整后为黑白图像；为1时，调整后和原图相同
			返回值：无
	* ```random_contrast(probability, min_factor, max_factor)```

		随机改变图像的对比度

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				min_factor(Float) - 取值在0到max_factor之间，为调整图像对比度范围的最小值。为0时，图像调整后为单一灰度图像；为1时，调整后和原图相同
				max_factor(Float) - 取值应大于max_factor，为调整图像对比度范围的最大值。为0时，图像调整后为单一灰度图像；为1时，调整后和原图相同
			返回值：无
	* ```random_erasing(probability, rectangle_area)```

		随机删除图像中的部分区域，使用随机像素值对其进行填充。目的在于提高模型的鲁棒性，增强对遮挡的处理能力。

			参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
				rectangle_area(Float) - 删除区域的大小占原图大小的比例，取值范围为0.1到1
			返回值：无

* ```resize(probability, width, height, resample_filter=u'BICUBIC```

	根据width和height指定的大小resize图像

		参数：probability(Float) - 在图像上进行这个操作的概率，对于resize，推荐设置为1
			width(Integer) - 调整后的图像的宽度
			height(Integer) - 调整后的图像的高度
			resample_filter(String) - BICUBIC, BILINEAR, ANTIALIAS, 或NEAREST
* ```scale(probability, scale_factor)```

	放大图像，同时保持长宽比。

		参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			scale_factor(Float) - 缩放因子，应大于1，以保证对原图像进行放大
		返回值：无
* ```blacke_and_white(probability, threshold=128```

	将图像转变为二值化图像，默认阈值为128

		参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
			threshold(Integer) - 0到255之间的值，控制图像二值化的阈值。像素值小于它转变为黑色，大于它则转变为白色
		返回值：无
* ```greyscale(probability)```

	将图像转化为灰度图，推荐将probability设置为1
* ```invert(probability)```

	对图像进行色彩翻转

		参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率
		返回值：无		
* ```histogram_equalisation(probability=1.0)```

	对图像进行直方图均衡化

		参数：probability(Float) - 0到1之间的值，表示在图像上进行这个操作的概率。推荐设置为1
		返回值：无
* ```set_save_format(save_format)```

	设置图片的存储格式。通过令save\_format="auto"可以根据每幅图像的扩展名为分别为它们选择正确的存储格式。也可以将存储格式指定为固定的格式，例如令save\_format="JPEG"或"JPG"。但是这可能会导致一些错误，例如将含有alpha通道的PNG图像存储为JPEG格式。
* ```keras_generator(batch_size, scaled=True, image_data_format=u'channels_last')```

	返回图像生成器，图像来源于pipeline中一系列操作产生的图像

		参数：batch_size(Integer) - batch大小
			scaled(Boolean) - 默认为True，此时像素值转变为0到1之间的float32类型；为False时，像素值是0-255之间的整数
			image_data_format(String) - 数据格式，为'channels_last'和'channels_first'两者之一
		返回值：一个图像生成器
* ```keras_generator_from_array(images, labels, batch_size, scaled=True, image_data_format=u'channels_last')```
* ```process()```
* ```sample_with_array(image_array, save_to_disk=False)```
* ```static categorical_labels(numerical_labels)```
* ```static set_seed(seed)```
* ```image_generator()```
* ```torch_transform()```
