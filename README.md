# Sign Language App Workflow

This is a rough guide for how to use the below suite of applications to create a Sign Language training dataset, train an SVM-based model, and embed this in a smartphone application to translate gestures in real time

* [Dataset Creator](https://github.com/Mquinn960/dataset-creator)
* [Offline Trainer](https://github.com/Mquinn960/offline-trainer)
* [Sign Language (App)](https://github.com/Mquinn960/sign-language)

Thank you for taking the time to read this help guide, if you have any additional questions regarding this project please get in touch. Have fun!

![Alt text](/Preview.png?raw=true "Preview")

## 1. Creating your *Training* and *Testing* datasets using [Dataset Creator](https://github.com/Mquinn960/dataset-creator)

This phase involves creating a usable dataset consisting of images of Sign Language gestures which will be used to train the SVM kernel for recognition

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

This phase involves ingesting your training and testing data to train an SVM kernel and produce an output file which can be used with the [Sign Language (App)](https://github.com/Mquinn960/sign-language) in order to make predictions in real time. Performance and accuracy data on any available test images will also be produced during this phase.

1. Use Git to clone the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) repository.
2. Follow the ```README``` steps to ensure you have configured *IntelliJ IDEA* correctly and that you have resolved the project dependencies
3. Edit the ```Main``` class in the ```src``` folder to configure some testing parameters:
    * ```DATASETS_FOLDER``` - The folder created during phase 1 above containing your training datasets. When using multiple datasets, an **example** folder structure is shown below:

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
*note: the dataset folder names are not important, but the **train** and **test** folder names are, and the **image filename**s are, see phase 1 above for more info*

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

6. The testing results will be placed in the ```RESULTS_FOLDER``` specified in step 3 with file names corresponding to the parameters used in step 3.

7. The actual trained SVM models for use with the [Sign Language App](https://github.com/Mquinn960/sign-language) will be output to the project root in the form  ```trained_1.xml```, where ```1``` indicates the sequential run #. This file can then be loaded into the app as shown in phase 3.

## Interpreting Test Result Files ##

The test result files created by the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) have some usable performance and accuracy statistics in them, here is the high level overview:

### Header ###

```
RUN 9
Features: none
SVM Kernel: linear
Dimensionality Reduction: pca
Dataset: mon95
```

This shows the parameters configured in step 3 above and what sequential # of run is this

### Training Results ###
```
Train Sample 1: A1_1_7.jpg
Train Sample 2: A2_1_7.jpg
Train Sample 3: A3_1_7.jpg
...
```

This is a list of every training sample you fed in to this run, this can be useful for finding examples of your *false positives* for example. *Note: these are numbered according to the order they were processed*.

```
Total Image Processing Time: 00:00:15.680
Average Image Processing Time: 00:00:00.11

Actual SVM Training Time: 00:00:12.234
Actual PCA Time: 00:01:09.250
```

This shows the time taken to perform the **training** phase alone in ```HH:mm:ss.mmm```

### Testing Results ###

```
Test Sample 1: A0_1.jpg
Test Sample 2: A0_1_2.jpg
Test Sample 3: A0_1_3.jpg
...
```

Again this shows the testing images as they were processed *Note: these are numbered according to the order they were processed*.

```
Total Image Processing Time: 00:00:05.141
Average Image Processing Time: 00:00:00.15

Actual Testing Time: 00:00:00.15
```
This shows the time taken to perform the **testing** phase alone in ```HH:mm:ss.mmm```

### Sample Category Matrix ###

```
CLASS: A
TP: 1 2 3 4 5 6 7 8 9 10 11 12 13 14
TN: 15 16 17 18 19 20 21 22 23 24 25
FP: 74
FN: 270 277
```
For each class, this output lists each **T**rue **P**ositive, **T**rue **N**egative, **F**alse **P**ositive and **F**alse **N**egative - the numbers correspond to the sample # shown above in "Testing Results". For example ```TP: 1``` would correspond to ```Test Sample 1: A0_1.jpg```

This allows you to create something like the following:

![Alt text](/Examples.png?raw=true "Preview")

### Simple Confusion Matrix ###

```
14 0 0 0 0 0 0 0
0 13 0 0 0 0 0 0
0 0 13 0 0 0 0 0
0 0 0 14 0 0 0 0
0 0 0 0 13 0 0 0
0 0 0 0 0 13 0 0
0 0 0 0 0 0 14 0
...
```
This is a simple confusion matrix of each class against each other class when testing. This matrix can be thought of as the following:

```
                        Actual Class

                   A  B  C  D  E  F  G  H
                A  14 0  0  0  0  0  0  0
 Predicted      B  0  13 0  0  0  0  0  0
 Class          C  0  0  13 0  0  0  0  0
                D  0  0  0  14 0  0  0  0
                E  0  0  0  0  13 0  0  0
                F  0  0  0  0  0  13 0  0
                G  0  0  0  0  0  0  14 0
...
```

This allows you to create something like the following:

![Alt text](/Matrix.png?raw=true "Preview")

## 3. Using the trained model to power the [Sign Language App](https://github.com/Mquinn960/sign-language)

This phase involves importing your trained model (```trained_1.xml```) created using the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) and then using this on an Android app.

1. Use Git to clone the [Sign Language App](https://github.com/Mquinn960/sign-language)

2. Follow the ```README``` steps to ensure you have configured *Android Studio* correctly and that you have resolved the project dependencies

3. Attach an Android smartphone to your computer, enabling ADB to allow debugging

4. Import your ```trained_1.xml``` into the ```sign-language\app\src\main\res\raw``` folder

5. **Important**: Make sure the [Sign Language App](https://github.com/Mquinn960/sign-language) is configured to use the same parameters as you used in your testing run:
    * For example, if you used:

        ```detection method: CANNY_EDGES ```

        ```kernel: RBF```
        
        ```dimensionality reduction: none```

        In your training run, then these need to match when using the [Sign Language App](https://github.com/Mquinn960/sign-language) otherwise you're going to pass in invalid data to the SVM.

      * ```Detection method``` can be set in the ```MainActivity``` class in the ```onCameraViewStarted``` method
      * ```SVM kernel``` type can be set in the ```FrameClassifier``` class in the constructor
      * ```Dimensionality Reduction``` can currently be used in the ```FrameClassifier``` **although there is an open issue to create an export process for PCA eigenvectors to support this, for now stick to ```none```**

6. Run the application, which will load the trained SVM file and allow for the live translation of sign language gestures **using your trained model!**

*Some information on the overall processing steps taken by the app are shown below*

![Alt text](/Process.png?raw=true "Preview")

## 4. (Optional) Changing the imaging kernel of the [Sign Language App](https://github.com/Mquinn960/sign-language) and importing this into the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) in order to perform your own experiments

It is encouraged to experiment with your own image pre-processing techniques when training your SVM model. In order to train and refine your App's imaging kernel and thus produce better results, you can alter the [Sign Language App](https://github.com/Mquinn960/sign-language)'s imaging kernel and import this into the [Offline Trainer](https://github.com/Mquinn960/offline-trainer). 

To do this, see the ```README``` of the [Sign Language App](https://github.com/Mquinn960/sign-language) regarding the ```.jar``` export process and the ```README``` of the [Offline Trainer](https://github.com/Mquinn960/offline-trainer) regarding the library import process.

Following this process ensures that both the app and the trainer have the same data processing methods and will result in a cohesive workflow when perfecting the recognition workflow.

*Some visual information on the current image processing steps taken by the app are shown below*

![Alt text](/Processing.png?raw=true "Preview")
