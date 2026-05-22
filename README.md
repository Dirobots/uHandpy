# uHandpy

We have one robot hand equipped with a camera.

The goal will be to train a model to be able to recognize ASL. Then an LLM would take the output of the first model and produce a response. This response
will then be the input of another model that will learn how to control the robot. 

We would obviously have to define :

- a reading frequency for the robot (like each 10s)
- a way to recognize spaces between the words (for example deciding that 20s without any sign is a space)
- a way to mark the end of the prompt (can be 30s without a sign or even seeing the hands drop)


The link to the uHandPy documentation is : [uHandPi docs](https://wiki.hiwonder.com/projects/uHandPi/en/latest/docs/1.getting_ready.html#)
A page with an ASL demonstration (for our own understanding) : [ASL letters demonstration](https://alphabet.lingvano.com/glossary/)
A dataset for the ASL letters and numbers : [Kaggle ASL Dataset](https://www.kaggle.com/datasets/ayuraj/asl-dataset/data?select=asl_dataset)


An idea to explore is to make the model learn autonomously, on himself.
