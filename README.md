# Sign Language App Workflow

This is a rough guide for how to use the below suite of applications to create a Sign Language training dataset, train an SVM-based model, and embed this in a smartphone application to translate gestures in real time

* [Dataset Creator](https://github.com/Mquinn960/dataset-creator)
* [Offline Trainer](https://github.com/Mquinn960/offline-trainer)
* [Sign Language (App)](https://github.com/Mquinn960/sign-language)

![Alt text](/Preview.png?raw=true "Preview")

## 1. Creating your *Training* and *Testing* datasets using [Dataset Creator](https://github.com/Mquinn960/dataset-creator)

This stage involves creating a usable dataset consisting of images of Sign Language gestures which will be used to train the SVM kernel for recognition

1. Use Git to clone the [Dataset Creator](https://github.com/Mquinn960/dataset-creator) repository
2. Attach a webcam device to your computer and make sure it's producing output
3. Follow the ```README``` steps to ensure you have Python installed and that you resolve the external dependencies using ```pip```
4. Run ```python create.py``` from the repository root to begin the script
     * You can edit the create.py script to define config variables to suit your testing needs:
        * ```mode``` - ```auto``` or ```manual```, manual if you want to be in control of when pictures are taken using spacebar, or auto if you want images taken in succession
        * ```images``` - the number of images to capture per class
        * ```delay``` - the delay (seconds) before capture starts, useful if you're performing the signs yourself so you can get into position and let the camera's aperture adjust to the scene lighting
        * ```delaybetween``` - delay (seconds) between each image being taken
        * ```path``` - folder path to store the images
5. Whether running in ```manual``` or ```auto``` mode, you will be prompted to enter which class you're performing, which will automatically be prepended to the output files.
    * For example, entering "A" as your input class will result in images being saved as ```A_0.png```, ```A_1.png``` and so on.

Try to capture at least 100 images of each gesture you require. When complete, you can split your data into training and testing subsets. If you're unsure what sort of split you require, aim for **80:20** in favour of training data - so if you capture 100 images for the class *"A"*, for example, you would set aside 20 of these as *test* data, so that these 20 could be used to verify your remaining 80 *train* images.

Your resultant image folder should look something like this. For use with the [Offline Trainer](https://github.com/Mquinn960/offline-trainer), the images can be nested within subdirectories within the ```train``` and ```test``` folders, only the file name matters as the first letter of the image file is used to classify it.

```javascript
/dataset-creator/
       |----train
              |---A_0.jpg
              |---A_1.jpg
              |---A_2.jpg
              |---...
              |---Y_9.jpg
       |----test
              |---A_81.jpg
              |---A_82.jpg
              |---...
              |---Y_81.jpg
```

*If you are unable to create a dataset of your own in this manner, consider downloading the Raw MNIST Sign Language Dataset from [Mon95](https://github.com/mon95/Sign-Language-and-Static-gesture-recognition-using-sklearn/blob/master/Dataset.zip) on GitHub and using this instead*

## 2. Training the SVM recognition kernel using the [Offline Trainer](https://github.com/Mquinn960/offline-trainer)

This stage involves ingesting your training and testing data to train an SVM kernel and produce an output file which can be used with the [Sign Language (App)](https://github.com/Mquinn960/sign-language) in order to make predictions in real time. Performance and accuracy data on any available test images will also be produced during this stage.

1. Use Git to clone the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) repository.
2. Follow the ```README``` steps to ensure you have configured *IntelliJ IDEA* correctly and that you have resolved the project dependencies
3. Edit the ```Main``` class in the ```src``` folder to configure some testing parameters:
    * ```DATASETS_FOLDER``` - The folder created during stage 1 above containing your training datasets. When using multiple datasets, an **example** folder structure is shown below:

```javascript
/hand-datasets/
       |----dataset-one
                |----train
                        |---A_0.jpg
                        |---A_1.jpg
                        |---A_2.jpg
                        |---...
                        |---Y_9.jpg
                |----test
                        |---A_81.jpg
                        |---...
                        |---Y_81.jpg
        |----dataset-two
                |----train
                        |---A_0.jpg
                        |---...
                |----test
                        |---A_81.jpg
                        |---...
```
*note: the dataset folder names are not important, but the **train** and **test** folder names are, and the **image filename**s are, see stage 1 above for more info*

3. (cont.)
    * ```RESULTS_FOLDER``` - This is the folder to which your testing result files will be saved
    * ```detectionMethods``` - This is a variable containing a list of image preprocessing methods to use on your data. Remove or comment out those you don't want to use, remember if you opt to use *canny edges* image processing for example, then the [Sign Language (App)](https://github.com/Mquinn960/sign-language) must also use this image processing technique - see section 3 for more info 
    * ```kernels``` - The type of SVM kernels which will be used when training, remove or comment out those you don't wish to use
    * ```dimReduceType``` - Whether dimensionality reduction should be used or not, remove or comment out those you don't wish to use
    * ```datasets``` - A list of strings containing the **folder names** of the datasets contained within the ```DATASETS_FOLDER``` folder. Following the example above this list would contain ```"dataset-one"``` and ```"dataset-two"```

    *Note: The trainer will use **every combination** of the above parameters! So this can be thought of as ```detection methods * kernels * dimensionality reduction types * datasets```. If you want to perform a single training run, there should be 1 entry in each list. For example, if you had ```2``` detection methods, ```2``` SVM kernel types, ```1``` type of dimensionality reduction and ```4``` datasets, the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) will perform ```16``` training runs (```2 * 2 * 1 * 4```).*

4. **(Optional)** Configure the ```EmailNotifier``` class with valid SMTP details in order to receive email updates about your training progress, useful when performing large scale training runs
5. Run the project using *either* method 1 or method 2 below
    * Method 1 - Manual Classpath
        * Add the following to your ```PATH``` under [environment variables](https://www.java.com/en/download/help/path.xml) in windows: ```F:\Repos\offline-trainer\new``` but substitute for the path to which you have cloned the repo on your computer
        * Load the project in IntellIJ IDEA, load the project and hit run

    * Method 2 - Build artifact and command line
        * If you want to run the project from the command line you can elect to ```Build->Build Artifact``` which will produce an executable ```.jar``` in the ```offline-trainer\out\artifacts\``` folder
        * Run this ```.jar``` file from the command line using ```java -Djava.library.path=F:\Repos\offline-trainer\new\ -jar offline-trainer.jar``` but substitute the file path to your cloned repo in the ```java.library.path``` string

6. The testing results will be placed in the ```RESULTS_FOLDER``` specified in step 3 with file names corresponding to the parameters used:


7. The actual trained SVM models for use with the [Sign Language App](https://github.com/Mquinn960/sign-language) will be output to the project root in the form  ```trained_1.xml```, where ```1``` indicates the sequential run #. This file can then be loaded into the app as shown in stage 3.