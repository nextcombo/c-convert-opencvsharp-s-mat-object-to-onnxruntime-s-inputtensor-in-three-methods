# [C#] Convert OpenCVSharp's Mat object to ONNXRuntime's inputtensor in three methods

Method 1: Directly assign values to each tensor during the creation of a tensor, as follows in pseudocode:

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

```
             var image = new Mat(image_path);            inpWidth = image.Width;            inpHeight = image.Height;            //Convert the image toRGBChannel            Mat image_rgb = new Mat();            Cv2.CvtColor(image, image_rgb, ColorConversionCodes.BGR2RGB);            //InputTensor            input_tensor = new DenseTensor<float>(new[] { 1, 3, inpHeight, inpWidth });            for (int y = 0; y < image_rgb.Height; y++)            {                for (int x = 0; x < image_rgb.Width; x++)                {                    input_tensor[0, 0, y, x] = image_rgb.At<Vec3b>(y, x)[0] / 255f;                    input_tensor[0, 1, y, x] = image_rgb.At<Vec3b>(y, x)[1] / 255f;                    input_tensor[0, 2, y, x] = image_rgb.At<Vec3b>(y, x)[2] / 255f;                }            }             //Output: input_tensor Place an input parameter container and specify the name.            input_container.Add(NamedOnnxValue.CreateFromTensor("input", input_tensor));             //Run Inference And obtain the result.            result_infer = onnx_session.Run(input_container);
```

The second method involves assigning values through unsafe pointers, which is pseudocode:

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

```
            var image = new Mat(image_path);             Mat dstimg = new Mat();             Cv2.Resize(image, dstimg, new OpenCvSharp.Size(inpWidth, inpHeight));             dstimg.ConvertTo(dstimg, MatType.CV_32FC3, 1 / 127.5, -1.0);             float* pdata = (float*)dstimg.Data;            float[] input_tensor_data = new float[1 * 3 * inpWidth * inpHeight];            for (int i = 0; i < 1 * 3 * inpWidth * inpHeight; i++)            {                input_tensor_data[i] = pdata[i];            }             // Input Tensor
```

The third method of assigning values through unsafe pointers is essentially the same as the second method, but it involves directly assigning values to image pointers. Here is pseudocode:

Demo code 1:

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

```
            var image = new Mat(image_path);             Mat resize_image = new Mat();            Cv2.Resize(image, resize_image, new OpenCvSharp.Size(512, 512));             float h_ratio = (float)image.Rows / 512;            float w_ratio = (float)image.Cols / 512;             int row = resize_image.Rows;            int col = resize_image.Cols;            float[] input_tensor_data = new float[1 * 4 * row * col];            int k = 0;            for (int i = 0; i < row; i++)            {                for (int j = 0; j < col; j++)                {                    for (int c = 0; c < 3; c++)                    {                        float pix = ((byte*)(resize_image.Ptr(i).ToPointer()))[j * 3 + c];                        input_tensor_data[k] = pix;                        k++;                    }                    input_tensor_data[k] = 1;                    k++;                }            }             input_tensor = new DenseTensor<float>(input_tensor_data, new[] { 1, 512, 512, 4 });             //Put the input_tensor into a container with an input parameter and specify its name
```

Demo code 2:

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

```
            var image = new Mat(image_path);             int img_height = image.Rows;            int img_width = image.Cols;             Mat resize_image = new Mat();            Cv2.Resize(image, resize_image, new OpenCvSharp.Size(inpWidth, inpHeight));             int row = resize_image.Rows;            int col = resize_image.Cols;             float[] input_tensor_data = new float[1 * 3 * inpHeight * inpWidth];            for (int c = 0; c < 3; c++)            {                for (int i = 0; i < row; i++)                {                    for (int j = 0; j < col; j++)                    {                        float pix = ((byte*)(resize_image.Ptr(i).ToPointer()))[j * 3 + c];                        input_tensor_data[c * row * col + i * col + j] = (float)((pix / 255.0 - mean[c]) / std[c]);                    }                }            }            input_tensor = new DenseTensor<float>(input_tensor_data, new[] { 1, 3, inpHeight, inpWidth });             //Put the input_tensor into a container with a name, and specify it
```


## Payment

Here is a pay link on Stripe ( https://buy.stripe.com/3cs8yP7sY87d0vu9AB ). Please contact me lonlonago@foxmail.com after funding $89, and I will send you a complete data files , thank you!

![111.png](111.png)
