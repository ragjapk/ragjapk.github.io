We all know what GANs are.
The stories of the tough game between the Generator and Discriminator which ends in neither of them winning - but instead reaching a mutual agreement or a compromise(the Nash Equilibrium) have been going around since 2014. Along with the story, the disadvantages of GAN have also gained popularity. Let's address one of them for time being: Mode Collapse.

Mode Collapse: 
Let's consider a 2D example - as it is easier to visualize a 2D plane.
Suppose the following is your data: There is a 2D plane of size 400x100. However, data of your interest is restricted to 3 small clusters compared to the whole plane. Basically you want the GAN to generate only points that belong in the red clusters.


Real Data

You design an ordinary GAN to mimic your data given the 2D space. However, your GAN is able to capture only partial data. It is not able to generate the data present at the top left corner.


Generated Data

This is referred to as Mode Collapse.

What is the issue here? Why is the GAN unable to generate from all three modes(tiny areas of the high dimensional space where the actual data is present)?
Let's think about it.
How can the generator generate from only one area of the data space?
If you look at how loss for a GAN is designed, generator tends to generate samples for which the discriminator gives a high score. 
Hence, it follows that the discriminator is giving a high score to the generator for generating from the particular area it is generating from above. 
Discriminator is right in this case as this area is part of the data distribution.
However, since the generator is getting a good feedback from the discriminator for what its generating currently, it fails to explore other areas fearing it may increase its loss.
My work is getting excellent feedback now. Why should I venture out of my ways and risk that?

A Bayesian formulation to GANs was introduced in order to alleviate this mode collapse problem in GANs.
The generator G and discriminator D of GANs are neural networks parameterized by their respective weights ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$) and ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{d}$). A variant of stochastic gradient descent is applied as the optimization procedure in GANs -hence the weights - ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$) and ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{d}$) are approximated by a single value. 
Bayesian GANs propose to learns distribution ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g})$) and ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{d})$) over ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$) and ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{d}$) instead of following the maximum likelihood approach by learning them as point masses. 
Post training procedure, multiple generators can be constructed by sampling from ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g})$) instead of a single value, ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$). Each of these generators are assumed to have their own interpretation of the training data. Hence even if one generator isn't able to cover all the modes, all of them working towards can give meaningful representation of the data from different modes.

If you are a Bayesian you would be brimming with the following apprehensions,!!!

You have to calculate posterior over a set of neural network weights - There are no exact inference methods that can perform this - and few methods(Variational Inference/ Markov Chain sampling) who approximate it well enough 
In Bayesian approach, ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g})$) is computed with the help of Bayes Rule: ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g}\mid D)=\frac{p(D\mid\theta_{g})*p(\theta_{g})}{p(D)}$)
posterior=(likelihood * prior)/normalization constant                                                                 
In classification and regression tasks, ![formula](https://render.githubusercontent.com/render/math?math=$p(D\mid\theta_{g})$) aka the likelihood term is the softmax and gaussian distributions respectively. However, there isn't any explicit form for the likelihood function in GANs 
The Bayesian GANs paper define explicit forms for posteriors ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g})$) and ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{d})$) that seems like it is derived from Bayes Rule.
They turn to Stochastic Hamiltonian Monte Carlo - a Markov Chain Monte Carlo inference technique to obtain samples from the posterior distribution of the network weights.

Let's consider the posterior over ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$),  ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g})$) :
Intuitively, you want to sample ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$), such that the images generated by this parameter setting results in realistic images.
This thought boils down to the fact that you want to assign a higher chance of picking a value of  ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$) for which ![formula](https://render.githubusercontent.com/render/math?math=$D(G(z;\theta_{g});\theta_{d})$)(Discriminator score) is high. This can be seen as a proxy likelihood function - given ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$), how likely is the Generator going to create a realistic image.
Coupled with the prior distribution over ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$), ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g}\mid\alpha_{g})$) let's define the conditional(as ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$) is dependent on random variables ![formula](https://render.githubusercontent.com/render/math?math=$z$) and ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{d}$)) distribution of ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{g}$):
![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{g}\mid\theta_{d},z) \propto \prod_{i=1}^{n_{g}}D(G(z_{i};\theta_{g});\theta_{d})\times p(\theta_{g}\mid\alpha_{g})$)

Now, we can move onto the posterior over ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{d}$),  ![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{d})$) :
The job of a discriminator is to perform a binary classification task - whether the image is real(label=1)/generated(label=0). It outputs the probability that the image is real. Here, the label can be assumed to follow a Bernoulli Distribution - then we can write down the following Bernoulli likelihood function for the discriminator:

![formula](https://render.githubusercontent.com/render/math?math=$L(\theta)=\prod_{i=1}^{N}p(x_{i})^{y_{i}}(1-p(x_{i}))^{1-y_{i}}$)

In our specific GAN example, ![formula](https://render.githubusercontent.com/render/math?math=$p(x_{i})$ = $D(x_{i}\;\theta_{d})$, $y_{i}=1$)
                                        Also, ![formula](https://render.githubusercontent.com/render/math?math=$p(x_{i})$ =$D(G(z_{i},\theta_{g}),\theta_{d}))$, $y_{i}=0$)

Combining both the cases, the likelihood function can be written as: ![formula](https://render.githubusercontent.com/render/math?math=$\prod_{i=1}^{n_{d}}D(x_{i},\theta_{d})\times\prod_{i=1}^{n_{g}}(1-D(G(z_{i},\theta_{g}),\theta_{d})))$)

Finally, combining the likelihood and the prior, the conditional distribution of ![formula](https://render.githubusercontent.com/render/math?math=$\theta_{d}$) is below:
![formula](https://render.githubusercontent.com/render/math?math=$p(\theta_{d}\mid\theta_{g},z,X) \propto \prod_{i=1}^{n_{d}}D(x_{i},\theta_{d})\times\prod_{i=1}^{n_{g}}(1-D(G(z_{i},\theta_{g}),\theta_{d})))\times p(\theta_{d}\mid\alpha_{d})$)
