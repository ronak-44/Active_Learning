BayesianOptimizer
=================

When a function is expensive to evaluate, or when gradients are not available, optimalizing it requires more sophisticated methods than
gradient descent. One such method is Bayesian optimization, which lies close to active learning. In Bayesian optimization, instead of picking queries by maximizing the uncertainty of predictions, function values are evaluated at points where the promise of finding a better value is large. In modAL, these algorithms are implemented with the ``BayesianOptimizer`` class, which is a sibling of ``ActiveLearner``. They are both children of the ``BaseLearner`` class and they have the same interface, although their uses differ. In the following, we are going to shortly review this.

Differences with ActiveLearner
------------------------------

Initializing a ``BayesianOptimizer`` is syntactically identical to the initialization of ``ActiveLearner``, although there are a few important differences.

.. code:: python

    from modAL.models import BayesianOptimizer
    from modAL.acquisition import max_EI
    from sklearn.gaussian_process import GaussianProcessRegressor
    from sklearn.gaussian_process.kernels import Matern

    kernel = Matern(length_scale=1.0)
    regressor = GaussianProcessRegressor(kernel=kernel)

    optimizer = BayesianOptimizer(
        estimator=regressor,
        query_strategy=max_EI
    )

Most importantly, ``BayesianOptimizer`` works with a regressor. You can use them with a classifier if the labels are numbers, but the result will be meaningless. Bayesian optimization typically uses a Gaussian process regressor to keep a hypothesis about the function to be optimized and estimate the expected gains when a certain point is picked for evaluation. This latter is the task of the acquisition function. (`See below for details. <#acquisition-functions>`__)

The actual optimization loop is identical to the one you would use with the ``ActiveLearner``.

.. code:: python

    # Bayesian optimization: func is to be optimized
    for n_query in range(n_queries):
        query_idx, query_inst = optimizer.query(X)
        optimizer.teach(X[query_idx].reshape(1, -1), func(X[query_idx]).reshape(1, -1))

Again, the bottleneck in Bayesian learning is not necessarily the availability of labels. The function to be optimized can take a long
time and a lot of money to evaluate. For instance, when optimizing the hyperparameters of a deep neural network, evaluating the accuracy of the model can take a few days of training. This is a case when Bayesian optimization is very useful. For more details, see `this
paper <http://www.cs.ox.ac.uk/people/nando.defreitas/publications/BayesOptLoop.pdf>`__ for instance.

To see the maximum value so far, use ``optimizer.get_max()``:

.. code:: python

    X_max, y_max = optmizer.get_max()

Acquisition functions
---------------------

Currently, there are three built in acquisition functions in the ``modAL.acquisition`` module: *expected improvement*, *probability of improvement* and *upper confidence bounds*. You can find more information about them at the page :ref:`Acquisition-functions`.
