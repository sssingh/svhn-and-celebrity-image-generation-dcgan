# Street View House Numbers and Celebrity Faces Generation using DC-GAN 

<img src="assets/svhn_dcgan_title_pic.png" width="500">

### Generate fake images of Street View House Numbers and Celebrity Faces using Deep Convolutional Generative Adversarial Network (DC-GAN)


## Table of Contents

- [Introduction](#introduction) 
- [Objective](#objective)
- [House Number Image Generation](#svhn-image-generation)
- [Celebrity Face Image Generation](#celebrity-face-image-generation)
- [License](#license)
- [Author Info](#author-info)

---
## Introduction
In this project, we try to generate _fake_ house numbers based on Street View House Number (SVHN) dataset and _fake_ human faces using Large-scale CelebFaces Attributes (CelebA) Dataset. Both CelebA and SVHN dataset is more complex than the MNIST dataset; hence for this problem, we are going to use a variation of Generative Adversarial Network (GAN) called Deep Generative Adversarial Network or DC-GAN. DC-GAN is an extension and builds on the same idea as GAN networks. Refer [MNIST Digit Generation Using GAN](https://github.com/sssingh/mnist-digit-generation-gan) repo for a detailed discussion and a full-fledged working GAN network. Just like GANs, `DC-GANs` is also composed of two competing networks, the `Discriminator` that classifies images as _real_ or _fake_ and the `Generator` that learns from the real dataset (MNIST, SVHN, CelebA, etc.) and generates the _fake_ images. However, in the case of DC-GAN, instead of simple `fully-connected` _Linear_ layers, `Convolution` and `Transpose Convolution` layers are used for network composition. Moreover, we give DC-GAN networks the capability of learning from `spatial` image data instead of just flattened image vectors. Hence, DC-GAN can work with complex image datasets and produce better results than simple GANs. 

Have a look at the [DC-GAN Original Paper](https://arxiv.org/pdf/1511.06434.pdf) for more details.

As shown in the diagram below, the Generator consumes the `latent noise sample` and up-samples it to produce _fake_ images of the same size as that of images coming from the real dataset. The Discriminator down-samples the image to produce a single `logit` that'd be eventually used to classify an image as _fake_ or _real_

<img src="assets/dcgan_highlevel.png" width="800">


Like a simple GAN network, the DC-GAN also trains both `Generator` and `Discriminator` in parallel. Discriminator gets to see images from the real dataset and the _fake_ images generated by the Generator, and its job is to correctly classify _real_ images vs. _fake_ images. At the same time, based on how Discriminator is performing in real vs. fake classification, Generator keeps improving its generated images to make them look like the images taken from the real dataset. The Generator tries to fool the Discriminator, and the Discriminator tries not to get fooled.

---
## Objective
Our goal in this project is to... <br>
    1. Build a DC-GAN network and train it over ***SVHN*** dataset so that the network learns to generate _fake_ `street-house-numbers` images that would look like to have come from the real SVHN dataset.<br>
    2. Build a DC-GAN network and train it over the ***CelebA*** dataset so that the network learns to generate _fake_ `celebrity-faces` images that would look like to have come from the real CelebA dataset.

---
### SVHN Image Generation

#### Dataset
- SVHN dataset consists of 73,257  training images and 26,032 testing images. So for this task, we'll use the training dataset.
Every image in the dataset belongs to one of the 10 classes (digit 0 to 9); however, the image labels do not matter and we won't use them for this task.
- Each image in the dataset is a 32x32 RGB image; a few samples of SVHN images are shown below...

<img src="assets/svhn_samples.png" width="800">

- We will use the in-built SVHN dataset from PyTorch's `torchvision' package. Alternatively, the raw dataset can be downloaded from the original source [The Street View House Numbers (SVHN) Dataset](http://ufldl.stanford.edu/housenumbers/). The raw dataset provides train and test sets in the `.mat` file format.


#### Solution Approach
##### Data Load
- Just the `training` dataset is downloaded using the `torchvision` SVHN dataset in the `svhn_data` folder
- The training dataset is then wrapped in a dataloader object with a `batch_size` of 128. Note that even though the dataloader will give us the images associated, we'll simply ignore them.

##### Network Definition
A high-level DC-GAN network schematic is shown below...

<img src="assets/svhn_dcgan.png" width="800">

- The `Discriminator` network is simply a binary-classifier for classifying the input image as _real_ (1) or _fake_ (0).
- Network is composed to consume a batch of images (real or fake) and pass them through a series of 3 `Convolution` (Conv2D) layers and 1 `fully-connected` layer
- `LeakyReLU` is used as the activation function with a `negative-slope` of 0.2 per the original paper recommendation.
- The last layer of the network is a `fully-connected` `Linear` layer producing a single `logit` which will then be passed through a `sigmoid` function to generate prediction probability.
- `Batch Normalization` (BatchNorm2D) is applied after each convolution layer. However, the original paper recommends using it only after the 2nd and 3rd layers, but during training, I found that using it after every layer makes it perform better. 
- Based on the recommendations from the original DC-GAN paper, the down-sampling of images is performed by using a `stride of 2` instead of using `MaxPooling.`

<img src="assets/svhn_discriminator.png" width="800">

Note that there is an error in the diagram where shows BatchNorm2D being applied only after the 2nd and 3rd layers. The configured Discriminator Layers are shown below...

<img src="assets/svhn_discriminator_layers.png" width="800">


- `Generator` is the most interesting and the heart of a GAN. It learns to generate _synthetic_ (_fake_) data based on the underlying pattern/structure of the training data.
    - Network is defined to consume a `latent sample noise` (z of length 100) and up-samples it first by passing it through a `fully-connected` Linear layer and then through a series of 3 `Transpose Convolution` (ConvTranspose2D) layers. ConvTranspose2D layers work like Conv2D layers, but in reverse, i.e., instead of going from `wide and shallow` input to `narrow and deep` output, i.e., from an image to long/deep feature vector, it does the opposite of it, i.e., goes from long/deep feature vector to an image.
    - A `ReLU` activation is used per the original paper recommendation.
    The last ConvTranspose2D layer output is then passed through the `tanh` function to produce the final output of the Generator between -1 and 1. 
    - `Batch Normalization` (BatchNorm2D) is applied after each of the convolution layers

<img src="assets/svhn_generator.png" width="800"><br><br>

Note that there is an error in the diagram where shows BatchNorm2D being applied only after the 2nd and 3rd layers. The configured Discriminator Layers are shown below...

<img src="assets/svhn_generator_layers.png" width="800"><br><br>


##### Loss Definition
DC-GAN loss computation is the same as that of GAN networks. 

Note that GAN network training is different from our typical supervised neural-network training. In the case of GAN, two separate networks are being trained together, and this network has different and opposing objectives (i.e., they are competing). The `Discriminator` is trying to identify if the image sample is _real_ and from the actual  dataset or its  _fake_ images generated by our `Generator.` Note that we are NOT interested incorrectly classifying the digits themselves. We'd need to define two separate loss functions.
1. real_loss: calculates loss when images are drawn from the actual dataset. The predicted output is compared against the target label `1`, indicating real images.
2. fake_loss: calculates loss where images are generated by the Generator. The predicted output is compared against the target label `0`, indicating fake images. 

`Discriminator` computes both of the above losses and adds them together to get a `total-loss` for back-propagation
`Generator` computes the `real_loss` to check its success in _fooling_ the `Discriminator.` i.e., even though it generates fake images (target 0), by computing real_loss its compares Discriminator's output with `1`. In effect, generator loss has its labels `flipped.`


##### Network Training
DC-GANs are trained in the same manner as that of GAN networks. 

We are training two separate networks; we need two separate optimizers for each network. As per the original paper recommendations, in both cases, we use `Adam` optimizer with `0.0002`  `learning-rate,` `beta1` as default `0.5` and `beta2` as `0.999`. 
- Since classification is between two classes (real and fake) and our Discriminator outputs a `logit,` we use `BCEWithLogitsLoss` as the loss function. This function internally first applies a `sigmoid` activation to logits and then calculates the loss using the `BCELoss` (log loss) function.
Before starting training, we create a (16 x 100) `fixed-random-noise-vector` drawn from a `normal distribution` between range `-1 and 1`. This vector is kept fixed throughout the training. After each epoch of training, we feed the noise vector to, so far, trained Generator to generate _fake_ images; these images help us visualize how and if generated image quality is improving or not. A sample of the noise vector is shown below..

<img src="assets/latent_vector.png" width="800"><br><br>

- `Discriminator` is trained as follows...
    - A batch of `real` SVHN images are drawn from the dataloader
    - Each image in the batch is then `scaled` to values between `-1 and 1`. This is a crucial step and required because Discriminator looks at _real_ images from SVHN dataset and looks at _fake_ images from Generator whose output is in range `-1 to 1` (last layer output of Generator network is `tanh` activated). So we need to ensure that the range of input values is consistent in both cases.
    - data batch is then fed to Discriminator, its predicted output is captured, and `real_loss` is calculated
    - A batch noise data (`z`) drawn from a `normal distribution` between range `-1 and 1` is created
    - Noise `z` is then fed through the Generator, its outputs (fake images) are captured, and `fake_loss` is calculated
    - Then discriminator's `total_loss` is computed as `real_loss + fake_loss`
    - Finally, `total_loss` is back-propagated using Discriminator's optimizer
- After one batch of `Discriminator` training (above), the `Generator` is trained as follows...
    - A batch of noise data (`z`) drawn from a `normal distribution` between range `-1 and 1` is created
    - Noise `z` is then fed through the Generator, its outputs (fake images) are captured
    - The generated _fake_ images are then fed through the `Discriminator,` and its predicted output is captured, and `real_loss` is calculated
    - Note that for _fake_ generated images we are calculating `real_loss` (and not fake_loss) as discussed in [Loss Definition](#loss-definition) section above 
    - Above computed loss is then back-propagated using Generator's optimizer
- At the end of each epoch, the `fixed-random-noise-vector` is fed to the trained Generator to produce a batch of _fake_ images; we then save these images as `fixed_generated_samples.pkl`. We can load and view these saved images later for further analysis.
After training our GAN network for `25` epochs, we plot both Generator and Discriminator losses and it looks like this...

<img src="assets/svhn_losses.png" width="800"><br><br>

The above plot does not look like a typical neural-network training loss plot. There are considerable fluctuations in Generator's loss initially, and even after that, it's very spiky. This behavior is typical of GAN training and expected since the Generator continuously tries to fool the Discriminator. We observe that Discriminator loss is low and somewhat went down at the end; this possibly indicates that our Generator may not be powerful enough for this dataset. We may need to tweak our network to add a few more layers or tune hyperparameters.

Below are the images generated by the `Generator` after 25 epochs of training...

<img src="assets/svhn_results.png" width="800"><br><br>

We can see that it has started to generate `recognizable` house numbers that appear to be coming from the SVHN dataset. However, images are blurry, and some images show just random blobs; this is evident in the loss plot above, where Generator loss is high throughout compared to Discriminator loss. However, we have trained only for 25 epochs, still a good result. We can improve the quality by introducing the network for longer, say 100 or more epochs, or/and tuning the hyperparameters. It's also a good idea to use an `odd` kernel size instead of even (we are 4 in all cases).


#### How To Use
1. Ensure the below-listed packages are installed
    - `NumPy`
    - `pickle`
    - `matplotlib`
    - `torch`
    - `torchvision`
2. Download `svhn_generation_dcgan.ipynb` jupyter notebook from this repo
3. Execute the notebook from start to finish in one go. If a GPU is available (recommended), it'll use it automatically; otherwise, it'll fall back to the CPU. 
4. Train for 25 or more epochs. More extended training will yield better results
7. A trained model can be used to generate fake SVHN house numbers, as shown below...

```python
    # Generate latent noise samples
    fixed_z = np.random.uniform(-1, 1, (16, z_size))
    # Ask trained generator to generate fake images
    fake_images =  generator(z)
    # Re-scale generated images form [-1, 1] value range to matplotlib friendly range [0, 1]
    rescaled_images = (fake_images + 1) / 2
    # Display generated images
    display_images(rescaled_images, figsize=(12, 10))
```

---
### Celebrity Face Image Generation
We will build and train another DC-GAN again to generate new _fake_ faces of celebrities based on the `CelebA` dataset. This task is more complex than MNIST or SVHN numbers generation because learning details of human face patterns and then generating realistic-looking human faces is not a trivial task. Moreover, we can't use the network architecture used for SVHN; instead need to modify our network to make it deeper and more powerful to account for dataset complexity.

#### Dataset
- The actual [CebebA Dataset](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html) consists on 202,599 images of 10,177 distinct celebrities
- For this project, we will not use the complete dataset; instead, we will use a small subset of the CelebA dataset that can be downloaded from [here]https://s3.amazonaws.com/video.udacity-data.com/topher/2018/November/5be7eb6f_processed-celeba-small/processed-celeba-small.zip)
- Every image in this dataset belongs to one of the 10,177 celebrities, and it has been cropped to remove parts of the image that don't include a face, then resized down to 64x64x3 NumPy images
- Few samples of images from the dataset are shown below...

<img src="assets/celebA_face_samples.png" width="800">


#### Solution Approach

##### Data Load
- The small subset dataset is downloaded from the location mentioned in the above section, and it's unzipped into a folder called `processed_celeba_small.` The actual images are contained in a sub-folder `celeba` and another sub-folder under `celeba` called `New Folder With Items.` Finally, we move all photos from these sub-folders to the root folder `processed_celeba_small.`
- Images are resized into 32x32x3 size. The original 64x64x3 images would produce a better result, but it's very resource-intensive; hence I had to downsize the images further. 
- The dataset is then wrapped in a dataloader object with a `batch_size` of 128. Note that even though the dataloader will give us the images and associated labels, we'll simply ignore them.
- Preprocessing the images to have values between -1 and 1 as we know that the output of a `tanh` activated Generator will contain pixel values in a range from -1 to 1, and so, we need to rescale our training images to the same range


##### Network Definition and Training
Like SVHN DC-GAN we built earlier, we need to define a `Discriminator`  and a `Generator` network.
    Below is the network structure for `Discriminator` and `Generator.` Note that these networks are far deeper than the one we used for SVHN. Another difference is that we are using `LeakyReLU` for both networks. I found that, in this case, instead of `ReLU if LeakyRelU` is used for `Generator` as well, the network performs much better. The rest of the design elements are the same as SVHN, `BatchNorm2D` is used after each layer. The `Discriminator` generates a single logit for binary classification, and `Generator` uses `ConvTranspose2D` and generates a `tanh` activated output. Like SVHN, no `MaxPooling` is used; instead, the up and downsampling is done using `stride > 1`.


<img src="assets/celeb_discriminator.png" width="800">


<img src="assets/celeb_generator.png" width="800">

- To help our network converge, we should initialize the weights of the convolutional and linear layers. For example, the [original DCGAN paper](https://arxiv.org/pdf/1511.06434.pdf) says:
> All weights were initialized from a zero-centered normal distribution with a standard deviation of 0.02.
- The `loss function` and `optimizers` are defined the same manner as that of SVHN DC-GAN we built above. 
- The training strategy used is also the same as SVHN DC-GAN. Both `Discriminator`  and a `Generator` networks are trained together for 25 epochs, and we save the training generator samples as `train_samples.pkl` after each epoch for the latter visualization.
Finally, we plot both Generator and Discriminator training losses, and it looks like this...

<img src="assets/celeba_losses.png" width="800"><br><br>

The plot looks very similar to what we saw in the case of SVHN DC-GAN, which is typical DC-GAN behavior.
- Once we have a trained `Generator,` the Discriminator can be discarded as it's no longer needed.


##### Visualize Generator Training Progress
When we visualize the _fake_ images of celebrity faces generated by our `Generator,` it gradually improves from non-recognizable faces to somewhat recognizable face images.

Generated images after one epoch...

<img src="assets/celeba_gen_images_1.png" width="800">

Generated images after five epochs...

<img src="assets/celeba_gen_images_5.png" width="800">

Generated images after ten epochs...

<img src="assets/celeba_gen_images_10.png" width="800">

Generated images after 15 epoch...

<img src="assets/celeba_gen_images_15.png" width="800">

Generated images after 20 epoch...

<img src="assets/celeba_gen_images_20.png" width="800">

Generated images after 25 epochs...

<img src="assets/celeba_gen_images_25.png" width="800">


The model generates a reasonable quality of fake faces, given that it has been trained only for 25 epochs. However, it's evident that faces are predominantly white faces, and that's because the dataset consists of primarily white celebrities. Below modifications can further improve the quality of generated images...
1. instead of 32x32 images, use full-size 64x64 images; it'd take longer to train
2. preprocess images and apply the transformation to sharpen the images that may help in highlighting certain features
3. Increase model depth by adding a few more convolutions
4. Experiment with various learning-rate and beta1/beta2 hyperparameters values
5. Train for a longer time 


#### How To Use
1. Ensure the below-listed packages are installed
    - `NumPy`
    - `pickle`
    - `matplotlib`
    - `torch`
    - `torchvision`
2. Download `celebrity_face_generation_dcgan.ipynb` jupyter notebook from this repo
3. Execute the notebook from start to finish in one go. If a GPU is available (recommended), it'll use it automatically; otherwise, it'll fall back to the CPU. 
4. Train for 25 or more epochs. More extended training will yield better results
7. A trained model can be used to generate fake CelebA faces, as shown below...

```python
    # Generate latent noise samples
    fixed_z = np.random.uniform(-1, 1, (16, z_size))
    # Ask trained generator to generate fake images
    fake_images =  G(z)
    # Re-scale generated images form [-1, 1] value range to matplotlib friendly range [0, 1]
    rescaled_images = (fake_images + 1) / 2
    # Display generated images
    display_images(rescaled_images, figsize=(12, 10))
```

---
## License

MIT License

Copyright (c) [2021] [Sunil S. Singh]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the software, and to permit persons to whom the software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## Author Info

- Twitter - [@sunilssingh6](https://twitter.com/sunilssingh6)
- Linkedin - [Sunil S. Singh](https://linkedin.com/in/sssingh)
---
