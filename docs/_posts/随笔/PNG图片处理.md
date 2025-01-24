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
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

/**
 * 对一批PNG图片进行处理：将每张图片中从像素坐标(1230,0)到(1330,16)的矩形区域内的每个水平行（y轴上的每一行），用该行最左边的像素（x=1230）的颜色值覆盖该行从1230到1330的像素。
 *
 * @author Kevin Zhang With tongyi
 * @since 20250124
 */
public class PngImageProcessor {

    public static void main(String[] args) {
        String inputDir = "D:\\temp\\replace\\png"; // 默认目录路径

        if (args.length > 0) {
            inputDir = args[0];
        } else {
            System.out.println("请指定需要处理图片存放的目录。");
            System.out.println("Usage: java PngImageProcessor D:\\temp\\replace\\png");
            return;
        }

        Path dirPath = Paths.get(inputDir);
        long startTime = System.nanoTime(); // 记录开始时间

        try {
            System.out.printf("开始递归处理【%s】目录下的png图片文件...\n", inputDir);
            final int[] fileCount = {0};
            Files.walkFileTree(dirPath, new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                    if (file.toString().toLowerCase().endsWith(".png")) {
                        processPngFile(file);
                        fileCount[0]++;
                    }
                    return FileVisitResult.CONTINUE;
                }

                @Override
                public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
                    System.err.println("访问文件失败: " + file);
                    return FileVisitResult.CONTINUE;
                }
            });

            long endTime = System.nanoTime(); // 记录结束时间
            long duration = (endTime - startTime);  // 计算时间差，单位是纳秒
            double seconds = (double) duration / 1_000_000_000.0; // 将纳秒转换为秒，并输出结果

            System.out.printf("【%d】个图片文件处理完毕，耗时：%.3f秒。\n", fileCount[0], seconds);

        } catch (IOException e) {
            System.err.println("遍历文件树时出错.");
            e.printStackTrace();
        }
    }

    private static void processPngFile(Path file) {
        try {
            BufferedImage image = ImageIO.read(file.toFile());

            int startX = 1230;
            int endX = 1330;
            int startY = 0;
            int endY = 16;

            for (int y = startY; y <= endY; y++) {
                int color = image.getRGB(startX, y); // 获取起始像素颜色
                for (int x = startX + 1; x <= endX; x++) {
                    image.setRGB(x, y, color); // 用起始像素颜色覆盖其他像素
                }
            }

            ImageIO.write(image, "png", file.toFile());
        } catch (IOException e) {
            System.err.println("处理文件时出错: " + file);
            e.printStackTrace();
        }
    }
}
```

测试处理程序运行情况：

```bash
java PngImageProcessor D:\temp\replace\png
开始递归处理【D:\temp\replace\png】目录下的png图片文件...
【150】个图片文件处理完毕，耗时：292.933秒。
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