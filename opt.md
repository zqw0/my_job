```c++
//
// Created by zqw on 19-9-29.
//

#ifndef PROJECT_OPENCV_OPT_H
#define PROJECT_OPENCV_OPT_H

#include <vector>
#include <algorithm>
#include <stdio.h>
namespace lambda {
#define SIZE std::pair<int, int>
    enum class Interpolation{
        BEGIN  = 0,
        INTER_NEREST,
        INTER_BILINEAR
    };
    template <typename T>
    void Inter_nerest_resize_(T* OldBuffer, T* NewBuffer, int N, int C, int H, int W, double fx, double fy){
        int newH = H * fy;
        int newW = W * fx;
        //NewBuffer = new T[newH * newW * C * N];
        int page = 0;
        int start_addr = 0;
        int channels = 0;
        int newx,newy;
        int oldx,oldy;
        int cnt = 0;
        while(page < N){
            newx = cnt % newW;
            newy = (cnt % (newW * newH) ) / newW;
            oldx = newx / fx;
            oldy = newy / fy;
            NewBuffer[cnt++] = OldBuffer[oldy * W + oldx + start_addr];
            if(cnt % (newH * newW) == 0){//换通道
                start_addr += (H * W);
                channels++;
                if(channels % C == 0)
                    page++;
            }
            //std::cout<<cnt<<std::endl;
        }
    }
    template <typename T>
    void Inter_bilinear_resize_(T* OldBuffer, T* NewBuffer, int N, int C, int H, int W, double fx, double fy){
        int newH = H * fy;
        int newW = W * fx;
        //NewBuffer = new T[newH * newW * C * N];
        int page = 0;
        int start_addr = 0;
        int channels = 0;
        int newx,newy;
        double oldx,oldy;
        int cnt = 0;
        while(page < N){
            newx = cnt % newW;
            newy = (cnt % (newW * newH) ) / newW;
            oldx = (newx*1.0 + 0.5) / fx - 0.5;
            oldy = (newy*1.0 + 0.5) / fy - 0.5;
            int uplx = floor(oldx);
            int uply = floor(oldy);
            int uprx = uplx + 1;
            if(uprx >= W) uprx = W-1;
            int upry = uply;
            int downlx = uplx;
            int downly = uply + 1;
            if(downly >= H) downly = H-1;
            int downrx = uplx + 1;
            if(downrx >= W) downrx = W-1;
            int downry = uply + 1;//计算待加速
            if(downry >= H) downry = H-1;
            double u = oldx - uplx;
            double v = oldy - uply;
            int cufx[2],cufy[2];
            cufx[0] = u * 2048.0;
            cufx[1] = 2048.0 - cufx[0];
            cufy[0] = v * 2048.0;
            cufy[1] = 2048.0 - cufy[0];
            //printf("原点：(%lf, %lf)  左上角(%d, %d)  右上角(%d, %d)  左下角(%d, %d)  右下角(%d, %d)\n", oldx, oldy, uplx, uply, uprx, upry, downlx, downly, downrx, downry);
            NewBuffer[cnt++] = (OldBuffer[uply * W + uplx + start_addr] * cufx[1] * cufy[1] +
                    OldBuffer[upry * W + uprx + start_addr] * cufx[0] * cufy[1] +
                    OldBuffer[downly * W + downlx + start_addr] * cufx[1] * cufy[0] +
                    OldBuffer[downry * W + downrx + start_addr] * cufx[0] * cufy[0]) >> 22;
            if(cnt % (newH * newW) == 0){//换通道
                start_addr += (H * W);
                channels++;
                if(channels % C == 0)
                    page++;
            }
        }
    }
    template<typename T>
    void resize(T* Buffer, T* DstBuffer, int N, int C, int H, int W, SIZE S, double fx = 0.0, double fy = 0.0, Interpolation op = Interpolation::INTER_NEREST){
        if(!fx && !fy){
            fx = S.first*1.0 / (W*1.0);
            fy = S.second*1.0 / (H*1.0);
        }
        if(op == Interpolation::INTER_NEREST){
            Inter_nerest_resize_(Buffer, DstBuffer, N, C, H, W, fx, fy);
        }
        else if(op == Interpolation::INTER_BILINEAR){
            Inter_bilinear_resize_(Buffer, DstBuffer, N, C, H, W, fx, fy);
        }
    }
}
#endif //PROJECT_OPENCV_OPT_H
```
