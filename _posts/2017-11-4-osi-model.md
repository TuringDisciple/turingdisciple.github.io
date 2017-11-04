---
layout: post
title: "The OSI Model: Introduction"
---
 
Intro
=========
The OSI model (the open systems interconnection model) is a conceptual model which is used to abstract the underlying technology and structure of the telecommunication systems.
It is split into several layers. Layers communicate and serve the layers above and below them. In this series of posts I am going to describe the function of the several different layers but first an overview.

The Layers
-----------
The OSI model consists of seven layers as shown in the image.

<p align="center">
	<img src="/assets/osi-model.jpg">
</p>

### Layer 1: Physical
The physical is, as the name suggests, to do with everything that is physical when it comes to telecommunications. This includes

- Cables and the bits that form data
- Physical and electrical signals
- Conversion between data to signals
- Voltages, pinouts etc
- Modems, hubs and repeaters are part of the physical layer 

The main point of the physical layer is for the communication of raw data.

### Layer 2: Data Link
This layer is responsible for handling access to a computer or device. It takes in incoming data bits and packages them into frames. The data link layer, unlike the network layer, is concerned only with point to point communication between devices. This means that in the data link layer, devices will not know the intended final destination of a network packet, but will make sure to forward packets along to a directly connected device. It also defines the protocol for two physically connected devices and also defines flow control between them.

### Layer 3: Network (AKA IP layer)
This layer defines the means for transferring data through one or more networks. Logical addresses are translated to physical addresses in this layer. This layer also contains protocols to handle:
- Network routing 
- Fragmentation and reassembly of data packets

### Layer 4: Transport 
This layer is responsible for management and control of packet transfer. Communications  here are split in packages(datagrams and packets). In my articles [here](https://nashemncube.github.io/2017/10/27/udp-protocol.html) & [here](https://nashemncube.github.io/2017/10/29/tcp-protocol.html) I go into detail of the two protocols available within this layer and their functionality (TCP and UDP). The main thing to know is that TCP is a receipt type protocol you know data has been transferred or not, which data has been transferred and so on. UDP doesn't provide this "receipt". An analogy to use would be to consider the post office. If I send a letter with recorded delivery so that I can track it, I'd be using TCP. I know when it arrives (if at all) and who received it. UDP would be akin to using regular mail. There's no guarantee that the letter arrives, and if lost, I am unaware. Reasons to use UDP are mainly bandwidth related as UDP requires less data, and for applications where we just want to push data as quick as possible without any overhead we'd use UDP (video streaming).

### Layer 5: Session
This layer is responsible for a "admin". By this I mean the session layer handles
- Traffic control
- Controls connections between communications
- Establish, manage and terminate connections 
- Regulates how much can be sent and when by a computer
- Coordinates conversation 

### Layer 6: Presentation
This translates data to something presentable to the application layer and introduces independence from differences in data representation. Data is formatted in this layer, and if using encryption protocols, this is the layer in which data is encrypted or decrypted. 

### Layer 7: Application 
This allows applications to access the net, identifies communication partners, determines resource availability and synchronises communications. Protocols in this layer include HTTP, SMTP and FTP. 

### Conclusion
The reason for this OSI model is not just for abstraction but for debugging. It helps to split different parts of a network into layers so when there are failures, it becomes easier to identify them.

Thanks for reading

~Nashe
