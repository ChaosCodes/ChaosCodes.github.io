---
title: A few views about OpenFaas
date: 2019-06-28 17:33:21
tags: OpenFaas
---

# A few views about OpenFaas

> After learning openfaas, I find it a wonderful open source framework that can help me build up a web service without lots of efforts. So here I recommend OpenFaas and my reasons are listed below.

## Why I Use OpenFaas to Deploy my function

### High Efficiency

In OpenFaas, when you want to deploy a web server, you don't need to build a scaffold or configure the router. All you need to do is just focusing on your own function, which means that you just need to write a function, and then you server can be built automatically by OpenFaas.



### Any language

In OpenFaas, you can deploy your own function in any language you like. (As far as I'm concerned,) now in front-end, the mainstream language is Javascript. However, in back-end, Java, Golang, C is favored, which means that if we want to work as a front-end coder, we need to get a command of one of the mainstream language. Yet in OpenFaas, you don't neet to worry about the language problem, just use the language you are familiar with and you can build function whatever you like.



### No limit

Compared with the other serverless service such as AWS Lambda, OpenFaas have no time limit and run on any platform with Kubernetes or Docker. Without OpenFaas, when you want to  deploy a serverless function, you are required to sign up with your credit card, edit code in a web browser and even you need to limit the storage of the image and the time you want to run the function. OpenFaas, the open source Functions-as-a-Service framework can help you deploy the funciton wherever you want without any limit.



### Auto-Scaling

Nowadays in mang companies, when the request increase in a short time, you need to scale you VMs for the server manually in advance, or the server may crash under a huge number of requests. But in OpenFaas, Prometheus will analysis the requests number and scale the server automatically(see more detail in [Auto-Scaling-in-OpenFaas](https://chaoscodes.github.io/2019/06/15/Auto-Scaling-in-OpenFaas/)).



### Metric

In OpenFaas, the Gateway has been exposed a lot of metrics to help you monitor the health and behavior of the functions. You don't need to inject the metrics in the server as you make metrics in the other framework like flask, Django and so on. Just query, you can see the state of your function, how many times the function has been called, how many replicas it has or the execution time of the function, which is really convenient for developer.

 

# Application Scenarios of OpenFaas

![aso](aso.png)

We can find the scenarios in the silde of [Alex Ellis's talk](https://www.youtube.com/watch?v=C3agSKv2s_w)

1. **Machine Learning**

   Because ML needs a great amount of memory to work, the other serverless framework couldn't have enough memory to support the algorithm. However, OpenFaas can scales to help ML run well. In addition, ML is suitable to work as a simple server which can deploy easily. Nowadays, there are many company use OpenFaas to deploy a ML function such as to transfer a image from color to black and white or to recognize what object is in a image. When ML work with OpenFaas, it will be more powerful. You can just pack it as a function and call it as you need it. You can see the example in this [post](https://finnian.io/blog/colourising-video-with-openfaas-serverless-functions/)

   

2. **Batch Jobs**

   OpenFaas can scale its node by Swarm or Kubernetes, and it is suitable for OpenFaas to handle some batch jobs like CI. You can put your not-very-good machine together and build up a stronger cluster where you can run your batch jobs with OpenFaas.

   

3. **Image/Video conversion**

   If you want to build a file conversations api interface, OpenFaas can make it perfect. It's shown by the [post](https://medium.com/iconscout/how-openfaas-came-to-rescue-us-ec129518cd46) that the author need to take 20 - 30 minutes to process the task about Image conversion and the task requries high memory. Moreover, they need to scaling the service manually before he turns to OpenFaas. So what can OpenFaas benefit their project? We can get it from his words, *Those 20–30 minute time taking tasks came down to 5–7 minutes duration. And most important part is, **We can deploy it to a new VM in a moment!*** 

4. **IoT**

   Due to the flexibility of OpenFaas, IoT can work with OpenFaas very well. Always, IoT devices don’t tend to be constantly making requests, nor do they tend towards constant connection strategies like websockets. So by serverless framework like OpenFaas, it can scale when the request comes. In addition, OpenFaas can make your devices become a cluster, and each part works as a part of a large system to make service togother.

   

5. **Chat Bots**

   In a Chatbot, conversation is the interface. User talks to the chat bots first. The bots parse the words and then analyse which interface can help answer the question. So, beacause what human will ask is unknown by the interface, auto-scaling in chat bots become more important. It will scale when you need it, which can make use of the storage more effictively. By the way, it is easier for chat bot to integrate with other service. 

   

6. **HTTP APIs**

   Compared with the traditional http API, OpenFaas can make it build quickly, configure and deploy resources within a few comands. More importantly, developer will not have to think about the servers, and focusing on the function is OK.



## In Conclusion

Although OpenFaas is an immature framework yet, but it can be powerful in cloud and distributed system, which is the trend of future.

##### Preference

1. [Serverless Faas Current Status and Future]([http://jolestar.com/serverless-faas-current-status-and-future/](http://jolestar.com/serverless-faas-current-status-and-future/)) 
2. [How Openfaas Came to Rescue Us](https://medium.com/iconscout/how-openfaas-came-to-rescue-us-ec129518cd46)
3. [Why IoT and Serverless Fit So Well!](https://read.iopipe.com/why-iot-and-serverless-fit-so-well-9fc9b9e94de9)
4. [Serverless and Chatbots: Made for Each Other](https://dzone.com/articles/serverless-and-chatbots-made-for-each-other)
5. [Build a RESTful API with the Serverless Framework](https://dev.to/sagar/build-a-restful-api-with-the-serverless-framework-ene)


