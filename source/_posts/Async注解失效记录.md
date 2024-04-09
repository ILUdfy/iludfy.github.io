---
title: '@Async注解失效记录'
date: 2023-04-06 16:15:27
tags:
  - spring
  - spring boot
categories: spring
---
# @Async注解失效记录
今天在做项目的时候，遇到了如下情景：
> 业务场景：
> 系统接入多个监控摄像头，并调用人脸识别算法，若视频中有人经过，启动录像，若无人，十秒后停止录像。

我的做法如下：
- 使用一个类```VideoRecorderService```来记录一个录制任务，它的部分代码如下：


```java
    private String inputFile;//文件输入路径

    private String outputFile;//视频保存路径

    private boolean status = false;//视频中是否有人

    private boolean recording = false;//是否正在录制

    private long noPersonTime;//记录视频中没有人的开始时刻

    public void setStatus(boolean status){
        if (!this.status && status){//上一时刻有人且这一时刻没人，记录noPersonTime
            noPersonTime = System.currentTimeMillis();
            log.info("开始录制，noPersonTime="+noPersonTime);
        }
        this.status = status;
    }

    @Async("threadPoolTaskExecutor")//异步
    public void startRecordVideo(VideoEntity video)
            throws Exception {
        //此处省略一系列流程

        //开始录制视频
        recording=true;
            Frame frame = null;
            while ( (frame = grabber.grabFrame()) != null) {//视频帧图像不为空
                if (!status){
                    if ((System.currentTimeMillis()-noPersonTime)>=(long)10*1000){//没人时间超过10s
                        recording=false;
                        log.info("许久无人，停止录制");
                        break;
                    }
                }
                recorder.record(frame);//录制该帧
            }
        //停止录制视频
    }   
```

> 录制视频的方法使用@Async实现异步，但是它失效了。
> 一般@Async注解失效有如下三种原因：
> 1. 在需要用到的 @Async 注解的类上加上 @EnableAsync，或者直接加在springboot启动类上；
> 2. 异步处理方法（也就是加了 @Async 注解的方法）只能返回的是 void 或者 Future 类型；
> 3. 同一个类中调用异步方法需要先获取代理类，因为 @Async 注解是基于Spring AOP （面向切面编程）的，而AOP的实现是基于动态代理模式实现的。有可能因为调用方法的是对象本身而不是代理对象，因为没有经过Spring容器。
> 
> 我这里正是第三点出了问题，直接获取了```VideoRecorderService```对象而不是通过ioc容器获取它的动态代理。


- 我的错误代码如下：
```java
    @Component
    @Slf4j
    public class InitFFmpeg implements ApplicationListener<ApplicationReadyEvent> {

        @Autowired
        private VideoMapping videoMapping;

        public static HashMap<String,VideoRecorderService> map = new HashMap<>();

        @Override//该方法在spring容器启动完成后自动调用
        public void onApplicationEvent(ApplicationReadyEvent event) {
            List<VideoEntity> videos = videoMapping.selectAllVideos();//查询摄像头信息

            for (VideoEntity video : videos){

                //初始化map，把摄像头和VideoRecorderService对象关联起来
                map.put(video.getMonitorName(),new VideoRecorderService());

                //这里是jar包启动就会自动推流
                
                //这里是调用算法的部分
                
            }
        }}   
```
  > 由于这里直接new的对象，没有把它注册在ioc容器中，所以@Async失效。
- 解决方法：
  在```VideoConfiguration```类中写一个通过spring容器获取和注册bean的方法：
```java
    @Configuration
    public class VideoConfiguration implements ApplicationContextAware {

        private static ApplicationContext applicationContext;

        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            VideoConfiguration.applicationContext = applicationContext;
        }

        public static <T> T getBean(Class<T> tClass,String beanName){
            if (applicationContext.containsBean(beanName)){
                return applicationContext.getBean(beanName, tClass);
            }else {
                return null;
            }
        }

        /**
        *  动态注入bean
        * @param requiredType 注入类
        * @param beanName bean名称
        */
        public static void registerBean(Class<?> requiredType,String beanName){

            //将applicationContext转换为ConfigurableApplicationContext
            ConfigurableApplicationContext configurableApplicationContext = (ConfigurableApplicationContext) applicationContext;

            //获取BeanFactory
            DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) configurableApplicationContext.getAutowireCapableBeanFactory();

            //创建bean信息.
            BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(requiredType);

            //动态注册bean.
            defaultListableBeanFactory.registerBeanDefinition(beanName, beanDefinitionBuilder.getBeanDefinition());

        }}
```


> 把InitFFmpeg类中的map删掉，并改成动态注册bean对象：


```java
for (VideoEntity video : videos){

    //初始化map，把摄像头和VideoRecorderService对象关联起来
    //map.put(video.getMonitorName(),new VideoRecorderService());

    //在ioc容器中注册bean
    //使用摄像头名称来命名bean对象，在使用时根据名称获取bean
    VideoConfiguration.registerBean(VideoRecorderService.class, video.getMonitorName());

    //这里是jar包启动就会自动推流
    
    //这里是调用算法的部分
    
}


//使用下面的语句获取bean对象：
VideoRecorderService videoRecorderService = VideoConfiguration.getBean(VideoRecorderService.class, video.getMonitorName());
```

再次尝试，异步成功，搞定！
