---
layout: post
title: "Sharks with laser beams"
date: 2014-05-07 18:56:00
categories: 3D-printing
---

I've progressed a little bit on my calibration. My idea is to combine [OpenCV][1] with the 
[Raspberry Pi camera][2] to optically calibrate the system. I fought with [ThinkRPi's tutorial][4]
a while before stumbling onto [the RaspiCam library][3]. After a quick download of that, I was
suddenly able to read camera frames into OpenCV!

My first pass of `laserfinder.cpp` is as follows:

{% highlight cpp %}
#include <ctime>
#include <unistd.h>
#include <iostream>
#include <raspicam/raspicam_cv.h>
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/features2d/features2d.hpp"
using namespace std; 
 
int main ( int argc,char **argv ) {
   
    time_t timer_begin, timer_end;
    raspicam::RaspiCam_Cv Camera;
    cv::Mat laserOn, laserOff, diff, thresh, out;

    //set camera params
    Camera.set( CV_CAP_PROP_FORMAT, CV_8UC1 );
    Camera.set( CV_CAP_PROP_FRAME_WIDTH, 1280 );
    Camera.set( CV_CAP_PROP_FRAME_HEIGHT, 960 );

    //Open camera
    cout<<"Opening Camera..."<<endl;
    if (!Camera.open()) {cerr<<"Error opening the camera"<<endl;return -1;}

    // Give it half a second to open... otherwise
    // your first image may be underexposed
    usleep(500000);

    // Capture first frame
    Camera.grab();
    Camera.retrieve(laserOff);
    cout << "Off frame grabbed. Please activate laser" << endl;
 
    // Wait 5 seconds so you can plug in the lasers
    usleep(5000000);
    Camera.grab();
    Camera.retrieve(laserOn);
    cout << "On frame grabbed." << endl;
    Camera.release();

    //save images
    cv::imwrite("laser-off.jpg",laserOff);
    cv::imwrite("laser-on.jpg",laserOn);

    // Take the difference of the two images and threshold it
    cv::absdiff(laserOff,laserOn,diff);
    cv::threshold(diff, thresh, 64, 255,0);
 
    // Save these too
    cv::imwrite("diff.jpg",diff);
    cv::imwrite("thresh.jpg",thresh);

    //Search for blobs
    // First, set up the parameters
    cv::SimpleBlobDetector::Params params;
    params.minDistBetweenBlobs = 50.0f;
    params.filterByInertia = false;
    params.filterByConvexity = false;
    params.filterByColor = false;
    params.filterByCircularity = false;
    params.filterByArea = true;
    params.minArea = 20.0f;
    params.maxArea = 500.0f;
        
    // Search for the blobs
    cv::Ptr<cv::FeatureDetector> blob_detector = new cv::SimpleBlobDetector(params);
    blob_detector->create("SimpleBlob");
    vector<cv::KeyPoint> keypoints;
    blob_detector->detect(thresh, keypoints);

    // Convert the "off" image to color so we can draw red circles on it
    cvtColor(laserOff, out, CV_GRAY2RGB);
    cout << keypoints.size() << " points located." << endl;

    // Draw a red circle for each blob we've found
    for (int i=0; i<keypoints.size(); i++){
        float x=keypoints[i].pt.x; 
        float y=keypoints[i].pt.y;
        cv::circle(out, keypoints[i].pt, 3, cv::Scalar(0,0,255), -1);
    }

    // Save the locations
    cv::imwrite("location.jpg",out);

    cout<<"Complete."<<endl;
}
{% endhighlight %}

And a `Makefile` to go with it:
{% highlight make %}
all:
	g++ laserfinder.cpp -o findr -I/usr/local/include -lraspicam -lraspicam_cv -lopencv_core -lopencv_highgui -lopencv_imgproc -lopencv_features2d
{% endhighlight %}

Using it is pretty simple. Make sure the laser beams are off, and launch the program.
The program will capture a frame. When it tells you to, plug in the lasers. The program 
will capture another frame, compute their difference, threshold the difference, and then
search for blobs. It will then write five images to the current directory:

 * `laser-off.jpg`: image with laser turned off
 * `laser-on.jpg`: image with laser turned on
 * `diff.jpg`: the difference between the first two images
 * `thresh.jpg`: the thresholded difference
 * `location.jpg`: a copy of `laser-off.jpg` except with small red circles marking the locations of the lasers.

[![laser-on.jpg](/assets/laser-on.jpg)](/assets/laser-on.jpg)

[![location.jpg](/assets/location.jpg)](/assets/location.jpg)

Next steps: connect what I have to [JNI][5] so I can have the program turn the lasers on and off for me,
recieve the values back, and work out the calibration as I mentioned in a previous post. Lots of work...
but I'm on the right path!

There are a few issues. Currently, as far as I know, this particular library can only handle black-and-white at a maximum resolution of `1280x960`. However, this is the best solution I've been able to come up with, and I should be able to do a fairly accurate calibration even at just this resolution. In addition, I'm going to play around with the exposure a bit so I can capture smaller blobs. If you look at `laser-on.jpg` you might notice that the camera is completely blown out across the whole laser dot. If I can reduce the blown-out area by reducing the exposure, I might be able to obtain more accurate results.

[1]: http://opencv.org/
[2]: http://www.adafruit.com/products/1367
[3]: http://www.uco.es/investiga/grupos/ava/node/40
[4]: http://thinkrpi.wordpress.com/opencv-and-pi-camera-board/
[5]: http://hohonuuli.blogspot.com/2013/08/a-simple-java-native-interface-jni.html
