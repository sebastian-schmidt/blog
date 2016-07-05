.. title: Colorized movies with neural network
.. slug: colorized-movies-with-neural-network
.. date: 2016-07-05 05:30:52 UTC+02:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

I colorized some black and white movies using the trained neural network model from `Zhang et al <http://richzhang.github.io/colorization/>`_.

The movies were converted to single frames via `ffmpeg <https://ffmpeg.org/>`_, colorized and then put together. As you will see this creates a lot of changes from frame to frame and could be improved by some persistence parameters for scenes.
The scripting was done in python using the example from `https://github.com/richzhang/colorization <https://github.com/richzhang/colorization>`_.
The neural network is implemented in caffe.

.. youtube:: qQSViqdd0tU

.. youtube:: S9JFIZ8022I

.. youtube:: tJmXwqvx69I
