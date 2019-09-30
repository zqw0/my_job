```c++
#include <iostream>
#include <opencv2/opencv.hpp>
#include "opt.h"
using namespace std;
using namespace cv;
int main() {
    Mat src;
    src = imread("/home/zqw/my_file/phone.png");
    cvtColor(src, src, CV_BGR2GRAY);
    int * ptr = new int[src.cols * src.rows + 1];
    Mat test;
    int step = 0;
    test.create(src.rows * 2.0, src.cols * 2.0, CV_8UC1);
    int *reptr = new int[1 * 1 * src.rows * 2 * src.cols * 2];
    for(int i = 0; i < src.rows; i++){
        for(int j = 0; j < src.cols; j++){
            ptr[step++] = src.at<uchar >(i,j);
            //cout<<"bug"<<endl;
        }
    }
    Mat dst;
    cv::resize(src, dst, Size(src.cols*2, src.rows*2), 0, 0, INTER_LINEAR_EXACT);
    lambda::resize(ptr, reptr, 1, 1, src.rows, src.cols, pair<int,int>(src.cols * 2.0, src.rows * 2.0), 2, 2, lambda::Interpolation::INTER_BILINEAR);
    int cntt = 0;

    for(int i = 0; i < test.rows; i++){
        for(int j = 0; j < test.cols; j++){
            test.at<uchar>(i,j) = saturate_cast<uchar>(reptr[cntt++]);
        }
    }
    int sum = 0;
    for(int i = 0; i < test.rows; i++){
        for(int j = 0; j < test.cols; j++){
            sum += abs(test.at<uchar >(i,j) - dst.at<uchar >(i,j));
        }
    }
    cout<<sum<<endl;
    cout<<"bug"<<endl;
    imshow("name",test);
    imshow("name1",dst);
    waitKey(0);
}
```
