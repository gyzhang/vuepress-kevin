---
title: PNG图片处理
date: 2025-01-24 12:05:05
permalink: /pages/efc140/
sidebar: auto
categories:
  - 随笔
tags:
  - 图片处理
author: 
  name: Kevin Zhang
  link: https://github.com/gyzhang
---
# PNG图片处理

我需要对一批图片进行处理，以下代码使用“通义”帮我写了几乎全部的代码，我只是补充了处理图片路径的输入参数和一些提示信息。

```java
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;

/**
 * 对一批PNG图片进行处理：将每张图片中从像素坐标(911,0)到(1009,16)的矩形区域内的每个水平行（y轴上的每一行），用该行最左边的像素（x=911）的颜色值覆盖该行剩余的16个像素。
 * @author Kevin Zhang With tongyi
 * @since 20250124
 */
public class PngImageProcessor {

    public static void main(String[] args) {
        String inputDir = "D:\\temp\\replace\\png"; // 替换为你的目录路径
        if (args.length > 0) {
            inputDir = args[0];
        } else {
            System.out.println("请指定需要处理图片存放的目录。");
            System.out.println("Usage: java PngImageProcessor D:\\temp\\replace\\png");
            return;
        }

        File directory = new File(inputDir);
        File[] files = directory.listFiles((dir, name) -> name.toLowerCase().endsWith(".png"));

        if (files != null) {
            System.out.println("开始处理【" + files.length + "】个图片文件......");
            long startTime = System.nanoTime(); // 记录开始时间
            for (File file : files) {
                try {
                    BufferedImage image = ImageIO.read(file);

                    int startX = 911;
                    int endX = 1009;
                    int startY = 0;
                    int endY = 16;

                    for (int y = startY; y <= endY; y++) {
                        int color = image.getRGB(startX, y); // 获取起始像素颜色
                        for (int x = startX + 1; x <= endX; x++) {
                            image.setRGB(x, y, color); // 用起始像素颜色覆盖其他像素
                        }
                    }

                    ImageIO.write(image, "png", file);

                } catch (IOException e) {
                    System.err.println("Error processing file: " + file.getName());
                    e.printStackTrace();
                }
            }
            
            long endTime = System.nanoTime(); // 记录结束时间
            long duration = (endTime - startTime);  // 计算时间差，单位是纳秒
            double seconds = (double) duration / 1_000_000_000.0; // 将纳秒转换为秒，并输出结果

            System.out.println("【" + files.length + "】个图片文件处理完毕，费时：" + seconds + "秒。");
        }
    }
}
```

测试处理程序运行情况：

```bash
java PngImageProcessor D:\temp\replace\png
开始处理【15】个图片文件......
【15】个图片文件处理完毕，费时：6.7953329秒。
```

现在的 AI 已经可以在工作中为我提供很多助力。

在 AI 辅助编程方面，我认为最为需要的能力有如下几点：

- 技术视野：你得需要大致知道这个需求用程序是能实现的；
- 技术深度：就算不借助 AI，使用搜索引擎，你也能完成这个工作，也就是说你能掌控这个需求；
- 归纳总结：你能清晰的归纳总结这个需求，能让 AI 明确知道你究竟想要什么；
- 分解引导：你能清晰的分解这个需求，在 AI 糊涂的时候，你能循序渐进的引导 AI 帮你做事情。

归根结底，AI 是你的一个“工具人”，能够帮助你快速的完成琐碎的“工作量”层面的事。

在使用得当的情况，AI 也是你探索新鲜事物的“实验室助手”，甚至也会是你的“实验室伙伴”。

当然，这些都得益于你的技术宽度，和发现问题后分解问题的深度。毕竟，人类无法想象出超出我们认知经验的其他事物。

Kevin@Chengdu#20250124