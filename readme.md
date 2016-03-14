# An embarrassingly simple approach to one-shot learning

This is an experiment in Python using approaches from the ICML '15 paper “An embarrassingly simple approach to zero-shot learning” by Bernardino Romera-Paredes, Philip H. S. Torr.

http://jmlr.org/proceedings/papers/v37/romera-paredes15.pdf

## An embarrassingly simple approach to zero-shot learning

With matrix factorization you can decompose a n*m array into a n*a array and a a*m array, where a is the number of latent features.

An embarrassingly simple approach to zero-shot learning uses this to do zero-shot learning.

During the training stage an n*m weight/coefficient matrix is trained, where n is the number of features and m is the number of classes. Such that np.argmax( np.dot( x.T, weight_matrix ) ) = predicted class.

They also train an a*m Signature matrix in an unsupervised manner. a is a number of (binary or soft) class attributes which can be found in the dataset, from external data, or in an unsupervised manner.

For instance, when the classes are: bear, horse and the attributes are [brown, can_ride, dangerous] the Signature matrix may look like:

bear horse
[ 1, 1 ] brown
[ 0, 1 ] can_ride
[ 0, 1 ] domesticated

Using the Signature matrix and the Weight matrix we calculate an n*a matrix V, such that np.dot(V, S) = ~W.

### Zero-shot

When we want to predict new classes we create a new signature matrix S'.

moose donkey tiger
[ 1, 1, 0 ] brown
[ 0, 1, 0 ] can_ride
[ 0, 1, 0 ] domesticated

We use np.dot( V, S') to obtain ~W'. For new test samples we do np.argmax( np.dot( x.T, ~W' ) to get our class prediction.

![MF](http://i.imgur.com/rXq8paJ.png "MF Training")

### One-shot

We create the Weight matrix using logistic regression.

We create attributes with unsupervised learning. 

We take the first 2 components of: 
- PCA 
- Local linear embedding

to create 4 class attributes. For every train sample belonging to a class we average the 4-dimensional PCA_LLE filter to get our Signature matrix. For instance with 2 classes "digit1, digit2":

digit1 digit3
[ 0.05, 0.06 ] PCA1
[ 0.11, 0.96 ] PCA2
[ 0.45, 0.11 ] LLE1
[ 0.95, 0.13 ] LLE2

When we want to predict new classes we take at least 1 sample and use the fitted PCA and LLE models to get a 4-dimensional vector. Taking the average of more samples per class improves performance.

digit7 digit9
[ 0.04, 0.19 ] PCA1
[ 0.12, 0.76 ] PCA2
[ 0.49, 0.14 ] LLE1
[ 0.85, 0.11 ] LLE2

### Experiment

We use a 10 class digit dataset with 64 features. We will use digits 0,1,2,7,8,9 for the seen classes. For the unseen digits we use 3,4,5,6.

After fitting logistic regression (0.91 accuracy) on the 6 seen classes our Weight coefficient matrix is 64*6. Our Signature matrix is 4*6. We calculate a 64*4 matrix V.

To create predictions for the unseen classes we take 1 sample per new class and transform them with PCA and LLE to get our Signature matrix S'.

We use V and S' to calculate 64*4 ~W'. Now for all test samples we calculate np.argmax( np.dot( x.T, ~W' ) to get a class prediction.

We obtain a multi-class accuracy of 0.846 with 50 labeled samples per class, 0.759 with 10 labeled samples per class, and a variant accuracy of 0.609 with 1 labeled sample per class. Random guessing is 0.25 accuracy.