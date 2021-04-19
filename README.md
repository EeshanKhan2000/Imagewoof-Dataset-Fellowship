# Dog Breed Distinction model trained on the Imagewoof Dataset

This project uses a ResNet-18 model as a base in a model with dropout and fully connected layers to perform fine-grained prediction of the Imagewoof Dataset. 
The approach used is as follows:

1. Cutout Augmentations. 2 Random squares of 12*12 pixels each were cutout from the 128*128 3 channel 	 images. The cutouts act as a method of regularization which does not create model sensitive hyperparameters, and hence is preferable for smaller, less deep models. This is suggested in this paper: https://arxiv.org/pdf/1708.04552.pdf 

Before cutout:

![Original](/Images/Original.png)

After cutout:

![Augmented](/Images/Augmented_2.png)

Along with this, some other simple augmentations were implemented.

2. Diffrential learning rates. The base model used (ResNet-18) was divided into 3 segments based on depth. Each segment was given a lower learning rate than the next one. The rationale behind this is that the earlier convolutional layers emphasize more basic and fundamental features of each image, while the later layers are more crucial in the final prediction.

![DLR](/Images/Architecture.png)

3. For initializing the diffrential learning rates optimally, an automatic LR finding method was implemented, in which the model was trained for 2 epochs, with learning rates updated with each batch, starting at a lower vaue and ending at a higher one. A graph was plotted of loss vs learning rate on a semi-log scale. This had 3 distinct regions:
	* Initially, the Loss remained almost constant.
	* It started to reduce very fast.
	* It started to rise very sharply.
A small segment in the second region was used for implementing the final Stochaistic Gradient Descent with Restarts.

![Optimum](/Images/Optimum_LR.png)

4. Cosine Annealing was used to implement cyclically varying LRs (with batch-iterations).
   The initial value of Learning rate is set at maximum desired value,
           At each cycle iteration, the value by which to multiply the learning rate is calculated as sec(pi*(cycle_iters/epoch_iters))+1
           As the cycle_iterations increase, the learning rate decreases, untill the cycle iters reach a fraction of the epoch iterations.
           Also, after each cycle is complete, the number of epoch iterations for each batch is increased by a factor, as was suggested in the paper: 
	Loshchilov & Hutter: "Sgdr: Stochastic gradient descent with restarts".

![SGDR](/Images/SGDR.png)

5. This method was benchmarked against a RedceOnPlateau callback implementation.
