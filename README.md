kerasR: R Interface to the [Keras Deep Learning Library](https://keras.io/)
=====================================================

**Author:** Taylor B. Arnold<br/>
**License:** [LGPL-2](https://opensource.org/licenses/LGPL-2.1)


[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/statsmaths/kerasR?branch=master&svg=true)](https://ci.appveyor.com/project/statsmaths/kerasR) [![Travis-CI Build Status](https://travis-ci.org/statsmaths/kerasR.svg?branch=master)](https://travis-ci.org/statsmaths/kerasR) [![CRAN Version](http://www.r-pkg.org/badges/version/kerasR)](https://CRAN.R-project.org/package=kerasR) [![Coverage Status](https://img.shields.io/codecov/c/github/statsmaths/kerasR/master.svg)](https://codecov.io/github/statsmaths/kerasR?branch=master) [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.814996.svg)](https://doi.org/10.5281/zenodo.814996)
 [![status](http://joss.theoj.org/papers/c83b6694f1192e920aa86912bd08919c/status.svg)](http://joss.theoj.org/papers/c83b6694f1192e920aa86912bd08919c)

## Overview

Keras provides a language for building neural networks as connections
between general purpose layers.
This package provides a consistent interface to the Keras Deep Learning
Library directly from within R. Keras provides specifications for
describing dense neural networks, convolution neural networks (CNN) and
recurrent neural networks (RNN) running on top of either TensorFlow or
Theano. Type conversions between Python and R are automatically handled
correctly, even when the default choices would otherwise lead to errors.
Includes complete R documentation and many working examples.

## Installation

You can download the current released version of CRAN:
```{r}
install.packages("kerasR")
```
Or the development version from GitHub:
```{r}
devtools::install_packages("statsmaths/kerasR")
```
You will also have to install the Python module **keras** and
either the module **tensorflow** or **Theano**. The best resource
for this is the [Keras Documentation](https://keras.io/#installation),
which should be updated as new releases are given. Note that you
should at a minimum be using Keras version 2.0.1. To check that
you have installed this properly, run the following in R, setting
the correct path to the version of Python that has installed
the Keras module:
```{r}
library(kerasR)
library(reticulate)

use_python("/path/to/bin/python")
keras_init()
keras_available()
```
The `keras_init` will throw a helpful message if it fails to
find keras and the function `keras_available` will return
`TRUE` if it is succesfully installed and loaded.
Issues, questions, and feature requests should be opened as
[GitHub Issues](http://github.com/statsmaths/kerasR/issues).

## A Small Example (Boston Housing Data)

Building a model in Keras starts by constructing an empty `Sequential`
model.

```{r}
library(kerasR)
mod <- Sequential()
```

The result of `Sequential`, as with most of the functions provided
by **kerasR**, is a `python.builtin.object`. This object type,
defined from the **reticulate** package, provides direct access to
all of the methods and attributes exposed by the underlying python
class. To access these, we use the `$` operator followed by the
method name. Layers are added by calling the method `add`.
This function takes as an input another `python.builtin.object`,
generally constructed as the output of another **kerasR** function.
For example, to add a dense layer to our model we do the following:

```{r}
mod$add(Dense(units = 50, input_shape = 13))
```

We have now added a dense layer with 200 neurons. The first layer
must include a specification of the `input_shape`, giving the dimensionality
of the input data. Here we set the number of input variables equal to 13.
Next in the model, we add an activation defined by a rectified linear
unit to the model:

```{r}
mod$add(Activation("relu"))
```

Now, we add a dense layer with just a single neuron to serve as the
output layer:

```{r}
mod$add(Dense(units = 1))
```

Once the model is fully defined, we have to compile it before fitting
its parameters or using it for prediction. Compiling a model can be
done with the method `compile`, but some optional arguments to it
can cause trouble when converting from R types so we provide a
custom wrapper `keras_compile`. At a minimum we need to specify
the loss function and the optimizer. The loss can be specified with
just a string, but we will pass the output of another **kerasR**
function as the optimizer. Here we use the RMSprop optimizer as it
generally gives fairly good performance:

```{r}
keras_compile(mod,  loss = 'mse', optimizer = RMSprop())
```

Now we are able to fit the weights in the model from some training
data, but we do not yet have any data from which to train! Let's
load some using the wrapper function `load_boston_housing`. We
provide several data loading functions as part of the package,
and all return data in the same format. In this case it will be
helpful to scale the data matrices. While we could use the
R function `scale`, another option is the keras-specific
function `normalize`, which we use here. One benefit of
`normalize` is that is allows for normalizing arrays along
arbitrary dimensions, a useful feature in convolutional and
recurrent neural networks.

```{r}
boston <- load_boston_housing()
X_train <- normalize(boston$X_train)
Y_train <- boston$Y_train
X_test <- normalize(boston$X_test)
Y_test <- boston$Y_test
```

Now, we call the wrapper `keras_fit` in order to fit the model
from this data. As with the compilation, there is a direct method
for doing this but you will likely run into data type conversion
problems calling it directly. Instead, we see how easy it is to
use the wrapper function (if you run this yourself, you will see
that Keras provides very good verbose output for tracking the
fitting of models):

```{r}
keras_fit(mod, X_train, Y_train,
          batch_size = 32, epochs = 200,
          verbose = 1, validation_split = 0.1)
```

Notice that the model does not do particularly well here, probably
due to over-fitting on such as small set.

```{r}
pred <- keras_predict(mod, X_test)
sd(as.numeric(pred) - Y_test) / sd(Y_test)
```
```
## [1] 0.6818437
```
Several more involved examples are contained in the package
vignette *R Interface to the Keras Deep Learning Library*.

## Contributing

If you would like to contribute to the project please contact
the maintainer directly. Please note that this project is
released with a [Contributor Code of Conduct](CONDUCT.md).
By participating in this project you agree to abide by its terms.
