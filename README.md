# Fast camera live streaming with UDP

### Introduction

For camera live-streaming application over the computer network, TCP/IP or Transmission Control Protocol/Internet Protocol could come first as a reasonable choice. This method allows a large piece of information to be transmitted reliably, as it manages how a large packet being broken into smaller packets to be transmitted and reassembled in the right order at the the destination. [1]

Another famous network protocol is UDP or User Datagram Protocol. The implementation of this method is for fast data transmission (low-latency). However, UDP's drawback is less reliable compared to TCP/IP as there is always chance of data loss (packet drop). [2] UDP is structured in the way that not neccessarily implementing many of the control checks as in TCP/IP for the speed trade-off. From author's experience in using UDP, lossy is there but not always a concern in reliable network.

In this project, I'll explore live streaming over the network with UDP for speedy streaming.

### Pre-requisite
- Python 3
- OpenCV
- Numpy
- Socket
- Struct
- Webcam

### Image Frame Packaging

Explore the `sender.py` file.

The implementation are two steps. OpenCV library is being used to obtain image from wecam in `main()`, then passing `frame` into the `FrameSegment()` class to break down the frame and send over UDP. 

Something to note here is that the **encoding** and **decoding** feature in OpenCV being used to further compress the raw image size significantly but still maintaining decent image quality after decompress. 

After the raw frame is compressed, implementing `numpy`'s `tostring()` feature to convert the array into one-dimension bytes-array. The data packet sent by UDP is called **datagram**, the maximum limit of each datagram is **2^16 or 65536 bytes** with some bytes are reserved for header. To avoid data overflow and corrupt the codes, I do not use up to the maximum datagram's limit.

To determine the number of segments, I am using ceiling to round up the quotient of **bytearray_size/MAX_IMAGE_DGRAM**. Because `math.ceil` in Python2 is not rounding up natively as in Python3, so adding `from __future__ import division` helps to get this function to ceil up properly [3].

In this simple implementation, I use the first byte for current segment number which serves as a simple flag for the receiver to know whether there are still more segments of the image data coming or not. If the segment number is 1, this is the last data segment. Others are welcomed to try other flag/checksum method for more reliable check (for example: if a packet is dropped/never arrived, how the logic being handled?)

### Image Frame Decode

Explore the `receiver.py` file.
The logic on the receiver's end is simple. It is to reverse the process of the sender:
- Make sure to buffer out the current frame to make sure we are not starting in the middle of the segment stream.
- Gather and concate all data segments (using the segment number for logic).
- Using `numpy`'s `fromstring()` to convert the byte-array back to numpy array.
- Using `OpenCV`'s `imdecode` to decode the compressed numpy array back to numpy image.
- Lastly, display the image.
