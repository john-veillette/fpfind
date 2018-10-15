# FixedPointFinder - A Tensorflow toolbox for finding fixed points and linearized dynamics in recurrent neural networks

This code finds and analyzes the fixed points of recurrent neural networks that have been built using Tensorflow. The approach follows that outlined in Sussillo and Barak, "Opening the Black Box: Low-Dimensional Dynamics in High-Dimensional Recurrent Neural Networks", Neural Computation 2013 ([link](https://doi.org/10.1162/NECO_a_00409)).

Written using Python 2.7.12.

## Recommended Installation

1. [Clone or download](https://help.github.com/articles/cloning-a-repository/) this repository.
2. Create a virtual environment for the required dependencies:
    To create a new virtual environment specific to Python 2.7, enter at the command line:
    ```bash
    $ virtualenv --system-site-packages -p python2.7 your-virtual-env-name
    ```
    where `your-virtual-env-name` is a path to the the virtual environment you would like to create (e.g.: `/home/fpf`). Then   activate your new virtual environment:
    ```bash
    $ source your-virtual-env-name/bin/activate
    ```
    When you are finished working in your virtual environment (not now), enter:
    ```bash
    $ deactivate
    ```
3. Automatically assemble all dependencies using `pip` and the `requirements*.txt` files. This step will depend on whether you require Tensorflow with GPU support.

    For GPU-enabled TensorFlow, use:

    ```bash
    $ pip install -r requirements-gpu.txt
    ```

    For CPU-only TensorFlow, use:

    ```bash
    $ pip install -r requirements-cpu.txt
    ```

## Advanced Installation

Advanced Python users may skip the Recommended Installation, opting to instead clone this repository and ensure that compatible versions of the following prerequisites are available:

* **TensorFlow** (requires at least version 1.10) ([install](https://www.tensorflow.org/install/)).
* **NumPy, SciPy, Matplotlib** ([install SciPy stack](https://www.scipy.org/install.html), contains all of them).
* **Scikit-learn** ([install](http://scikit-learn.org/)).
* **PyYaml** ([install](https://pyyaml.org/)).
* **RecurrentWhisperer** ([install](https://github.com/mattgolub/recurrent-whisperer/)).

## General Usage

1. Start by building, and if desired, training an RNN. ```FixedPointFinder``` works with any arbitrary RNN that conforms to Tensorflow's `RNNCell` API.
2. Build a ```FixedPointFinder``` object:
    ```python
    >>> fpf = FixedPointFinder(your_rnn_cell, tf_session, **hyperparams)
    ```
    using `your_rnn_cell`, the `RNNCell` that specifies the single-timestep transitions in your RNN, `tf_session`, the Tensorflow session in which your model has been instantiated, and `hyperparams`, a python dict of optional hyperparameters for the fixed-point optimizations.
  
3. Specify the `initial_states` from which you'd like to initialize the local optimizations implemented by ```FixedPointFinder```. These data should conform to shape and type expected by `your_rnn_cell`. For Tensorflow's `BasicRNNCell`, this would mean an `(n, n_states)` numpy array, where `n` is the number of initializations and `n_states` is the dimensionality of the RNN state (i.e., the number of hidden units). For Tensorflow's `LSTMCell`, `initial_states` should be an  `LSTMStateTuple` containing one `(n, nstates)` numpy array specifying the initializations of the hidden states and another `(n, nstates)` numpy array specifying the cell states.

4. Specify the `inputs` under which you'd like to study your RNN. Currently, To study the RNN given a set of static inputs, `inputs` should be a numpy array with shape `(1, n_inputs)` where `n_inputs` is an int specifying the depth of the inputs expected by `your_rnn_cell`. Alternatively, you can search for fixed points under different inputs by specifying a potentially different input for each initial states by making `inputs` a `(n, n_inputs)` numpy array.

5. Run the local optimizations that find the fixed points:
    ```python
    >>> fps = fpf.find_fixed_points(initial_states, inputs)
    ```
    The fixed points identified, the Jacobian of your RNN state transition function at those points, and some metadata corresponding to the optimizations will be returned in the `FixedPoints` object.`fps` (see [FixedPoints.py](https://github.com/mattgolub/fixed-point-finder/blob/master/FixedPoints.py) for more detail).

6. Finally, visualize the identified fixed points:
    ```python
    >>> fps.plot()
    ```
    You can also visualize these fixed points amongst state trajectories from your RNN (see `plot` in [FixedPoints.py](https://github.com/mattgolub/fixed-point-finder/blob/master/FixedPoints.py) and the example in [run_FlipFlop.py](https://github.com/mattgolub/fixed-point-finder/blob/master/example/run_FlipFlop.py))

## Example

``FixedPointFinder`` includes an end-to-end example that trains a Tensorflow RNN to solve a task and then identifies and visualizes the fixed points of the trained RNN. To run the example, descend into the example directory: `fixed-point-finder/example/` and execute:

```bash
>>> python run_FlipFlop.py
```

The task is the "flip-flop" task previously described in Sussillo and Barak (2013). Briefly, the task is to implement a 3-bit binary memory, in which each of 3 input channels delivers signed transient pulses (-1 or +1) to a corresponding bit of the memory, and an input pulse flips the state of that memory bit (also -1 or +1) whenever a pulse's sign is opposite of the current state of the bit. The example trains a 16-unit LSTM RNN to solve this task (Fig. 1). Once the RNN is trained, the example uses ``FixedPointFinder`` to identify and characterize the trained RNN's fixed points. Finally, the example produces a visualization of these results (Fig. 2). In addition to demonstrating a working use of ``FixedPointFinder``, this example provides a testbed for experimenting with different RNN architectures (e.g., numbers of recurrent units, LSTMs vs. GRUs vs. vanilla RNNs) and characterizing how these lower-level model design choices manifest in the higher-level dynamical implementation used to solve a task.

---
![Figure 1](paper/task_example.png)

##### Figure 1. Inputs (gray), target outputs (cyan), and outputs of a trained LSTM RNN (purple) from an example trial of the flip-flop task. Signed input pulses (gray) flip the corresponding bit's state (green) whenever an input pulse has the opposite sign of the current bit state (e.g., if gray goes high when green is low). The RNN has been trained to nearly perfectly reproduce the target memory state (purple closely overlaps cyan).
---
![Figure 2](paper/fixed_points.png)

##### Figure 2. Fixed-point structure of an LSTM RNN trained to solve the flip-flop task. ``FixedPointFinder`` identified 8 stable fixed points (black points), each of which corresponds to a unique state of the 3-bit memory. ``FixedPointFinder`` also identified a number of unstable fixed points (red points) along with their unstable modes (red lines), which mediate the set of state transitions trained into the RNN's dynamics. Here, each unstable fixed point is a "saddle" in the RNN's dynamical flow field, and the corresponding unstable modes indicate the directions that nearby states are repelled from the fixed point. State trajectories from example trials (blue) traverse about these fixed points. All quantities are visualized in the 3-dimensional space determined by the top 3 principal components computed across 128 example trials.

## Contribution Guidelines

Contributions are welcome. Please see the [contribution guidelines](https://github.com/mattgolub/fixed-point-finder/blob/master/CONTRIBUTING.md).
