---
layout: post
title: android随机生成ip地址类型的字符串
categories: [Android, ]
description: android随机生成ip地址类型的字符串 
keywords: Android, 
---


由于最近使用okhttp时，需要访问网络，但是ip被别人家封了，所以我整个人都疯了，好在okhttp中可以添加一个头，"x-forwarded-for" 来设置ip去访问，所以，人总是能想到办法来解决问题的。捂嘴笑)
这里仅仅实现java生成一个随机的IP类型的字符串比如:192.168.0.1;

### 代码如下
```
public class RandomIp {
    public static String getRandomIp() {
        //ip的范围
        int[][] range_ip = {{607649792, 608174079},//36.56.0.0-36.63.255.255
                {1038614528, 1039007743},//61.232.0.0-61.237.255.255
                {1783627776, 1784676351},//106.80.0.0-106.95.255.255
                {2035023872, 2035154943},//121.76.0.0-121.77.255.255
                {2078801920, 2079064063},//123.232.0.0-123.235.255.255
                {-1950089216, -1948778497},//139.196.0.0-139.215.255.255
                {-1425539072, -1425014785},//171.8.0.0-171.15.255.255
                {-1236271104, -1235419137},//182.80.0.0-182.92.255.255
                {-770113536, -768606209},//210.25.0.0-210.47.255.255
                {-569376768, -564133889}, //222.16.0.0-222.95.255.255
        };
        //生成一个随机数
        Random random = new Random();
        int index = random.nextInt(10);
        String ip = numToip(range_ip[index][0] + new Random().nextInt(range_ip[index][1] - range_ip[index][0]));//获取ip

        return ip;
    }

    /**
     * 数字拼接成ip字符串
     *
     * @param ip
     * @return
     */
    private static String numToip(int ip) {
        int[] b = new int[4];

        b[0] = (int) ((ip >> 24) & 0xff);
        b[1] = (int) ((ip >> 16) & 0xff);
        b[2] = (int) ((ip >> 8) & 0xff);
        b[3] = (int) (ip & 0xff);
        String ip_str = Integer.toString(b[0]) + "." + Integer.toString(b[1]) + "." + Integer.toString(b[2]) + "." + Integer.toString(b[3]);
        return ip_str;
    }


}
```
然后在MainActivity中用一个button按钮打印一下下了，

![这是来搞笑的图片](http://upload-images.jianshu.io/upload_images/1365793-f166c80dd9734228.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最主要的是下面的日志了~~~；

![我是ip](http://upload-images.jianshu.io/upload_images/1365793-c090f940f8073c8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 


### 后记:
使用这种生成的ip类型字符串，放到网络的请求头中，啊啊啊啊啊，就可以访问那个接口了，悄悄的~~~。本文只是小菜鸟在开发过程中的一些记录。如有不足之处，请老司机带带我啊。。。
 No newline at end of file
