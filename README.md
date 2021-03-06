# OpenCV Cheat Sheet


### Table of Contents

  * [Version](#version)
  * [Installation](#installation)
  * [Eclipse](#eclipse)
  * [Window](#window)
  * [Image](#image)
  * [Pixel](#pixel)
  * [Blur](#blur)
  * [Contrast and Brightness](#contrast-and-brightness)
  * [Colors and Masks](#colors-and-masks)
  * [Channles](#channels)
  * [Trackbar](#trackbar)
  * [Drawing](#drawing)
  * [Geometry](#geometry)
  * [Contour](#contour)
  * [Contributing](#contributing)
  
### Version
* OpenCV 2.4
  
### Installation

#### Ubuntu

* **Required**
```
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
```

* Download OpenCV

```
wget https://github.com/Itseez/opencv/archive/2.4.13.zip 
mkdir -p /tmp/OpenCV
mv opencv-2.4.13.zip /tmp/OpenCV
cd /tmp/OpenCV
unzip opencv-2.4.13.zip
```
  
* Make OpenCV 
```
cd opencv-2.4.13
mkdir release
cd release
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
make
sudo make install
```

#### Reference:  
http://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html#linux-installation

---

### Eclipse

* Project > Properties > C/C++ Build [Menu] > Tool Settings [Tab] > GCC C++ Compiler [Option] > Includes [Option] > Include paths (-I)  

  Add:
```
/usr/local/include/opencv
```

* Project > Properties > C/C++ Build [Menu] > Tool Settings [Tab] > GCC C++ Linker [Option] > Libraries [Option] > Libraries (-l)  

  Add:
```
opencv_core
opencv_imgproc
opencv_highgui
opencv_ml
opencv_video
opencv_features2d
opencv_calib3d
opencv_objdetect
opencv_imgcodecs
opencv_flann
```

* Project > Properties > C/C++ Build [Menu] > Tool Settings [Tab] > GCC C++ Linker [Option] > Libraries [Option] > Library search path (-L)  

  Add:
```
/usr/local/lib
```

#### Reference:  
* http://docs.opencv.org/2.4/doc/tutorials/introduction/linux_eclipse/linux_eclipse.html

---

### Window

Create window
```
namedWindow("My window", WINDOW_AUTOSIZE);
```

Destroy window
```
destroyWindow("My window");
```

Destroy all windows
```
destroyAllWindows();
```

Move window
```
moveWindow("My window", 150 /*x*/, 160 /*y*/);
```

Resize window
```
resizeWindow("My window", 500 /*width*/, 500 /*height*/);
```

Set mouse call back
```
setMouseCallback("My Image", [](int event, int x, int y, int flags, void* userdata) {
 cout << "X: " << x << ", Y: " << y << endl;
}, 0);
```

Wait instead of terminating the program
```
waitKey(0);
```

#### Examples  
* [examples/window/](examples/window)

#### Reference:  
* http://docs.opencv.org/2.4/modules/highgui/doc/user_interface.html

---

### Image

Load and store image in a matrix
```
Mat image = imread("img.jpg", CV_LOAD_IMAGE_COLOR);
```

Save image from matrix to file
```
imwrite( "img.jpg", image);
```

Show image
```
imshow("My Image", image);
```

Create a matrix of zeros
```
Mat new_image = Mat::zeros( image.size(), image.type() );
```

Clone image
```
image.clone();
```

Rotate image
```
Mat matRotate = getRotationMatrix2D(Point(imageSrc.rows/2, imageSrc.cols/2) /*center*/, 180 /*angle*/, 1 /*scale*/);
warpAffine(imageSrc, imageDest, matRotate, imageSrc.size() /*new size*/);
```

Save part of an image in a new matrix
```
Mat imageDest = imageSrc(Rect(0, 0, 100, 100));  // Mat::operator() overloaded
```

Resize image
```
resize(imageSrc, imageDest, Size(imageSrc.rows/2, imageSrc.cols/2)); // Resize to width=50% and height=50%
```

#### Examples  
* [examples/image/](examples/image)

#### Reference:  
* http://docs.opencv.org/2.4/modules/highgui/doc/reading_and_writing_images_and_video.html?highlight=imread#imread
* http://docs.opencv.org/2.4/modules/highgui/doc/user_interface.html

---

### Pixel

Loop on all pixels for a matrix
```
for( int row = 0; row < imageSrc.rows; row++ ){
	for( int col = 0; col < imageSrc.cols; col++ ){
		for( int c = 0; c < imageSrc.channels(); c++ ) {
			// image.at<Vec3b>(row,col)[c] = ...;
		}
	}
}
```

---

### Blur

#### Algorithms

Normalized Block Filter
```
int kernelSize=20;
blur( imageSrc, imageDest, Size(kernelSize, kernelSize), Point(-1,-1));
```

Gaussian Filter
```
int kernelSize=21; // Int: Positive & Odd
GaussianBlur( imageSrc, imageDest, Size(kernelSize, kernelSize), 0, 0);
```

Median Filter
```
int kernelSize=21; // Int: Odd
medianBlur(imageSrc, imageDest, kernelSize);
```

#### Other

Blur part of the image
```
Rect region(0 /*x*/, 0 /*y*/, 200 /*width*/, 200 /*height*/);
int kernelSize = 25;
GaussianBlur(imageSrc(region), imageDest(region), Size(kernelSize,kernelSize), 0, 0); // Mat::operator() overloaded
```

Save only the blurred region in a matrix
```
Rect region(0 /*x*/, 0 /*y*/, 200 /*width*/, 200 /*height*/);
int kernelSize = 25;
Mat imageDest;
GaussianBlur(imageSrc(region), imageDest, Size(kernelSize,kernelSize), 0, 0); // Mat::operator() overloaded
```

#### Examples  
* [examples/blur/](examples/blur)

#### Reference:  
* http://docs.opencv.org/2.4.13/doc/tutorials/imgproc/gausian_median_blur_bilateral_filter/gausian_median_blur_bilateral_filter.html#goal

---

### Contrast and Brightness

#### Information

Contrast and brightness can be controlled by the following formula:  `g(row, col) = alpha * f(row, col) + beta`, where:
* `g(row, col)` is the result pixel value
* `f(row, col)` is the original pixel value
* `alpha` controls the contrast
* `beta` controls the brightness

#### Manually

Using nested loops
```
double alpha = 1;
int beta = 100;

// Loop on matrix
for( int row = 0; row < imageSrc.rows; row++ ){
	for( int col = 0; col < imageSrc.cols; col++ ){

		// Loop on channels (0: Blue, 1: Green, 2: Red)
		for( int c = 0; c < imageSrc.channels(); c++ ) {
			
			// g(row, col) = alpha * f(row, col) + beta
			imageDest.at<Vec3b>(row,col)[c] = saturate_cast<uchar>(alpha *( imageSrc.at<Vec3b>(row,col)[c] ) + beta);
		}
	}
}
```
*saturate_cast<uchar>: Template function for accurate conversion from one primitive type to another.*

#### Automatically

Using `Mat::convertTo()` which applies the same formula `g(row, col)` explained above
```
double alpha = 1;
int beta = 100;
imageSrc.convertTo(imageDest, imageSrc.type(), alpha, beta);
```

Brightness can also be increased using `Scalar`
```
Mat imageDest = imageSrc + Scalar(80, 80, 80); // Add to each channel for all pixels of a matrix the amount specified in the arguments
```

#### Examples  
* [examples/contrast_and_brightness/](examples/contrast_and_brightness)

#### Reference:  
* http://docs.opencv.org/2.4/doc/tutorials/core/basic_linear_transform/basic_linear_transform.html
* http://docs.opencv.org/2.4/modules/core/doc/utility_and_system_functions_and_macros.html?highlight=saturate_cast#saturate-cast
* http://docs.opencv.org/2.4/modules/core/doc/basic_structures.html#mat-convertto

---

### Colors and Masks

Convert image to grayscale
```
cvtColor(imageSrc, imageDest, CV_BGR2GRAY);
```

Mark pixels in specified range by value 255 and all others by 0 in the destination matrix
```
inRange(imageSrc, Scalar(50, 50, 50) /*low*/, Scalar(180, 180, 180) /*high*/, imageDest);
```

Shrink bright regions within an image
```
erode(imageSrc, imageDest, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));
```

Grow bright regions within an image
```
dilate(imageSrc, imageDest, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));
```

Threshold
```
threshold(imageDest, imageDest, 150 /*threshold*/, 255 /*max*/, CV_THRESH_BINARY);
```

#### Examples  
* [examples/colors/](examples/colors)

#### Reference:  
* http://docs.opencv.org/2.4/modules/imgproc/doc/miscellaneous_transformations.html#cvtcolor
* http://docs.opencv.org/2.4/doc/tutorials/imgproc/erosion_dilatation/erosion_dilatation.html
* http://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html#inrange
* http://opencv-srf.blogspot.ca/2010/09/object-detection-using-color-seperation.html

---

### Channels

Get the number of channels for a matrix
```
image.channels();
```

Split multi-channel array into several single-channel arrays  
*Note: matrices created have one channel each*
```
vector<Mat> bgr;
split(imageSrc, bgr);
```

Merge several single-channel into one multichannel array
```
merge(bgr /*vector<Mat> bgr*/, imageDest);
```

#### Examples  
* [examples/channels/](examples/channels)

#### Reference:  
* http://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html

---

### Trackbar

Create trackbar (min=0 always)
```
int brightnessValue = 50;
createTrackbar("Brightness" /*label*/, "My window" /*existing window*/, &brightnessValue, 255 /*max*/);
```

Create trackbar with callback function (on change event)
```
int brightnessValue = 0;
createTrackbar("Brightness", "My window 1", &brightnessValue, 255, [](int brightnessValue, void *userData){
	imshow("My window 1", *(Mat*)userData);
}, &imageSrc);
```

#### Examples  
* [examples/trackbar/](examples/trackbar)

#### Reference:  
* http://docs.opencv.org/2.4/modules/highgui/doc/user_interface.html?highlight=createtrackbar#createtrackbar

---

### Drawing

Draw circle
```
circle(imageSrc, Point(imageSrc.rows/2, imageSrc.cols/2) /*circle center*/, 100 /*radius*/, Scalar(255, 0, 0) /*color*/, 10/*thickness*/);
```

Draw ellipse
```
ellipse(imageSrc, Point(imageSrc.rows/2, imageSrc.cols/2) /*ellipse center*/, Size(100, 50) /*min-max radius*/, 40 /*ellipse angle*/, 0 /*start angle*/, 359 /*end angle*/, Scalar(255,0,0) /*color*/, 10 /*thickness*/);
```

Draw line
```
line(imageSrc, Point(0, 0) /*from*/, Point(imageSrc.rows/2, imageSrc.cols/2) /*to*/, Scalar(255,0,0) /*color*/, 10 /*thickness*/);
```

Draw arrowed line
```
arrowedLine(imageSrc, Point(0, 0) /*from*/, Point(imageSrc.rows/2, imageSrc.cols/2) /*to*/, Scalar(255,0,0) /*color*/, 10 /*thickness*/);
```

Draw rectangle
```
rectangle(imageSrc, Point(0, 0) /*top-left*/, Point(imageSrc.rows/2, imageSrc.cols/2) /*bottom-right*/, Scalar(255,0,0) /*color*/, 10 /*thickness*/);
```

Draw polylines
```
vector<Point> points;
points.push_back(Point(0, 0));
points.push_back(Point(50, 50));
points.push_back(Point(200, 50));
polylines(imageSrc, points, true /*closed*/, Scalar(255,0,0) /*color*/, 10 /*thickness*/);
```

Draw text
```
putText(imageSrc, "Hello world", Point(50, 50), FONT_HERSHEY_PLAIN, 2.5 /*scale*/, Scalar(255,0,0) /*color*/, 5 /*thickness*/);
```

#### Reference:  
* http://docs.opencv.org/2.4/modules/core/doc/drawing_functions.html

---

### Geometry

Get center of a Binary matrix or contour
```
Moments oMoments = moments(imageSrc); // Mat should have 1 channel
// Moments oMoments = moments(contour /*vector<Point>*/);
double x = mmts.m10 / mmts.m00; // center x
double y = mmts.m01 / mmts.m00; // center y
```

#### Examples  
* [examples/geometry/](examples/geometry)
* [examples/contour/06_Multi_contour_center.cpp](examples/contour/06_Multi_contour_center.cpp)

#### Reference:  
* http://docs.opencv.org/2.4/modules/imgproc/doc/structural_analysis_and_shape_descriptors.html?highlight=moments#moments
* https://en.wikipedia.org/wiki/Image_moment

---

### Contour

Find and draw edges in 1 channel image
```
int thresh = 100;
Canny(imageSrc, imageDest, thresh /*threshold1*/, thresh*2 /*threshold2*/, 3/*apertureSize*/);
```

Find contours in a binary image  
*Note: Shape must be filled in white color, otherwise it will count it as more than one contour*  
```
vector<vector<Point> > contours;
vector<Vec4i> hierarchy;
findContours(imageSrc, contours, hierarchy, CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, Point(0, 0));
```
Parameter: mode  
* CV_RETR_EXTERNAL retrieves only the extreme outer contours.
* CV_RETR_LIST retrieves all of the contours without establishing any hierarchical relationships.
* CV_RETR_CCOMP retrieves all of the contours and organizes them into a two-level hierarchy. 
* CV_RETR_TREE retrieves all of the contours and reconstructs a full hierarchy of nested contours.


Draw contours
```
drawContours(imageDest, contours, i /*contours[i]*/, Scalar(0, 0, 255) /*color*/, 2/*thickness*/, 8/*type*/, hierarchy, 0, Point());
```

Approximates a polygonal curve(s) with the specified precision
```
approxPolyDP(Mat(contours[i]), approx_contours[i] /*don't overwrite*/, 3 /*accuracy*/, true /*closed*/);
```

Contour area
```
contourArea(contour /*vector<Point>*/);
```

Draw rectangle around contour
```
Rect rect = boundingRect(Mat(contours[i]));
rectangle(tmpImage, rect.tl(), rect.br(), Scalar(0,255,0), 2);
```

Draw circle around contour
```
Point2f center;
float radius;
minEnclosingCircle(contours[i], center, radius);
circle( tmpImage, center, radius, Scalar(255, 0, 0), 2/*thickness*/);
```

#### Examples  
* [examples/contour/](examples/contour)

#### Reference:  
* http://docs.opencv.org/2.4/modules/imgproc/doc/feature_detection.html?highlight=canny#canny
* http://docs.opencv.org/2.4/modules/imgproc/doc/structural_analysis_and_shape_descriptors.html
* http://docs.opencv.org/2.4/doc/tutorials/imgproc/shapedescriptors/find_contours/find_contours.html
* https://en.wikipedia.org/wiki/Ramer%E2%80%93Douglas%E2%80%93Peucker_algorithm

---

### Contributing

* Edit README.md
* Add your changes
* Write description of your changes in the `Commit changes` form
* Select the radio button "Create a new branch for this commit and start a pull request."
* Specify a name for your commit branch
* Click on "Propose file change" and then on "Create pull request"
