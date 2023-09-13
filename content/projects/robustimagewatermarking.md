---
title: "Robust Image Watermarking"
---

<br></br>

The Robust Image Watermarking is a technique based on Discrete Wavelet Transform ([DWT](https://en.wikipedia.org/wiki/Discrete_wavelet_transform)) and All Phase Discrete Cosine Biorthogonal Transform ([APDCBT](https://link.springer.com/article/10.1007/s11042-023-16106-1)), implemented for the "Multimedia Data Security" project.  

To have a look at the project source code, go [here](https://github.com/Harachili/robust_image_watermarking), while to read the paper on which this implementation was based, go [here](https://www.mdpi.com/2073-8994/10/3/77).  

Since this project was destined for a final exam, in which we had to implement the most resilient watermarking technique, while attacking the others', in the repository can also be found a couple of scripts which implement attacks that were used with the end goal of ruining the other teams' watermarks without losing the Weighted Peak Signal-to-Noise Ratio ([WPSNR](https://ieeexplore.ieee.org/document/8803307)).  


The implemented method is divided into two phases: insertion and extraction.  

The insertion phase is divided into 7 steps:

1. One-level DWT decomposition with “Haar” wavelet is first performed on the carrier image, yielding four sub-bands: three high-frequency sub-bands (LH, HL, and HH) and a low-frequency sub-band (LL).
2. We applied again on the low-frequency sub-band the DWT decomposition, yielding four sub-bands: three high-frequency sub-bands (LH2, HL2, and HH2) and a low-frequency sub-band (LL2).
3. To reduce the influence of watermark insertion, two high-frequency sub-bands, LH2 and HL2, are selected for the insertion of two identical watermarks. Using the HL2 sub-band as an example, the block-based APDCBT is applied to each sub-block obtained by dividing the image into 8x8 blocks.
4. The DC coefficient of each sub-block is used to generate a new coefficient matrix, denoted by `M`.
5. The first watermark `W1` is inserted into the coefficient matrix, which is shown as follows:  
   `SW = S + alpha*W1, SW = U_W * S_W1*V^T_W`
   
   where `alpha` is the watermark embedding intensity, `SW` is a matrix containing the watermark information, `U_W` and `V_W` are newly generated orthogonal matrices, and `S_W1` is the modified singular value matrix after watermark insertion.
6. Repeat the points 4. and 5. for the second watermark
7. At this point we apply the InverseAPDCBT and twice the InverseDWT to have in the end the Watermarked image
   
The extraction phase is divided in ?? steps:

1. We divide the original image using the DWT decomposition twice
2. We apply the APDCBT transformation on the HL2 to have in the end 1024 highest coefficients divided in 4x4 of the horizontal wavelet (HORIZ)
3. We apply the APDCBT transformation on the LH2 to have in the end 1024 highest coefficients divided in 4x4 of the vertical wavelet (VERTIC)
4. Apply the same steps on the watermarked image to obtain HORIZ_w and VERTIC_w
5. Now calculate W1 and W2 calculated respectively as (VERTIC_w-VERTIC)/alpha and (HORIZ_w-HORIZ)/alpha
6. Retrieve the watermark W which is calculated as  
   `W = (W1+W2)/2`

Regarding the attacks, we implemented the classic attacks (i.e. Blur, Median filtering, JPEG compression, etc); the most effective combination of them (i.e. Blur + Median, Blur + Median + JPEG, AWGN + Median, Resize + JPEG, etc); localized attacks after dividing the target's watermarked image in n parts.


