https://blog.csdn.net/ianly123/article/details/80061918

# Java 后台实现二维码生成（生成 base64或者直接本地保存，可以内嵌 logo）

### 1.生成 base64的二维码，前台展示

#### 1.引入 jar 包

```xml
<!-- 条形码、二维码生成 -->
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>2.2</version>
</dependency>
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>2.2</version>
</dependency>
```

#### 2.编写工具类

QrCodeUtils.java：

```java
public class QrCodeUtils {

    public static String creatRrCode(String contents, int width, int height) {
        String binary = null;
        Hashtable hints = new Hashtable();
        hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
        try {
            BitMatrix bitMatrix = new MultiFormatWriter().encode(
                    contents, BarcodeFormat.QR_CODE, width, height, hints);
            // 1、读取文件转换为字节数组
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            BufferedImage image = toBufferedImage(bitMatrix);
            //转换成png格式的IO流
            ImageIO.write(image, "png", out);
            byte[] bytes = out.toByteArray();

            // 2、将字节数组转为二进制
            BASE64Encoder encoder = new BASE64Encoder();
            binary = encoder.encodeBuffer(bytes).trim();
        } catch (WriterException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return binary;
    }

    /**
     * image流数据处理
     *
     * @author ianly
     */
    public static BufferedImage toBufferedImage(BitMatrix matrix) {
        int width = matrix.getWidth();
        int height = matrix.getHeight();
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < width; x++) {
            for (int y = 0; y < height; y++) {
                image.setRGB(x, y, matrix.get(x, y) ? 0xFF000000 : 0xFFFFFFFF);
            }
        }
        return image;
    }


    public static void main(String[] args) {
        String binary = QrCodeUtils.creatRrCode("https://blog.csdn.net/ianly123", 200,200);
        System.out.println(binary);
    }
}
```

#### 3.controller层获取生成图片二进制代码：

```java
String binary = QrCodeUtils.creatRrCode("二维码内容", 200,200);
```

#### 4.前台展示

```java
<div id="qrcode"><img src="data:image/png;base64,${binary }"></div>
```

### 2.后台生成二维码图片（可以内嵌 logo）文件，保存到本地。

#### 1.*生成二维码(内嵌LOGO)*

```java
package xin.ianly.utils.qrcode;

import com.google.zxing.*;
import com.google.zxing.client.j2se.BufferedImageLuminanceSource;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.common.HybridBinarizer;
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.geom.RoundRectangle2D;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.OutputStream;
import java.util.Hashtable;
import java.util.Random;

/**
 * @author ianly
 * @description
 * @date 2018-05-31
 */
public class QrCodeUtils2 {

    private static final String CHARSET = "utf-8";
    private static final String FORMAT = "JPG";
    // 二维码尺寸  
    private static final int QRCODE_SIZE = 300;
    // LOGO宽度  
    private static final int LOGO_WIDTH = 60;
    // LOGO高度  
    private static final int LOGO_HEIGHT = 60;

    private static BufferedImage createImage(String content, String logoPath, boolean needCompress) throws Exception {
        Hashtable<EncodeHintType, Object> hints = new Hashtable<EncodeHintType, Object>();
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H);
        hints.put(EncodeHintType.CHARACTER_SET, CHARSET);
        hints.put(EncodeHintType.MARGIN, 1);
        BitMatrix bitMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, QRCODE_SIZE, QRCODE_SIZE,
                hints);
        int width = bitMatrix.getWidth();
        int height = bitMatrix.getHeight();
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < width; x++) {
            for (int y = 0; y < height; y++) {
                image.setRGB(x, y, bitMatrix.get(x, y) ? 0xFF000000 : 0xFFFFFFFF);
            }
        }
        if (logoPath == null || "".equals(logoPath)) {
            return image;
        }
        // 插入图片  
        QrCodeUtils2.insertImage(image, logoPath, needCompress);
        return image;
    }

    /**
     * 插入LOGO 
     *
     * @param source
     *            二维码图片 
     * @param logoPath
     *            LOGO图片地址 
     * @param needCompress
     *            是否压缩 
     * @throws Exception
     */
    private static void insertImage(BufferedImage source, String logoPath, boolean needCompress) throws Exception {
        File file = new File(logoPath);
        if (!file.exists()) {
            throw new Exception("logo file not found.");
        }
        Image src = ImageIO.read(new File(logoPath));
        int width = src.getWidth(null);
        int height = src.getHeight(null);
        if (needCompress) { // 压缩LOGO  
            if (width > LOGO_WIDTH) {
                width = LOGO_WIDTH;
            }
            if (height > LOGO_HEIGHT) {
                height = LOGO_HEIGHT;
            }
            Image image = src.getScaledInstance(width, height, Image.SCALE_SMOOTH);
            BufferedImage tag = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
            Graphics g = tag.getGraphics();
            g.drawImage(image, 0, 0, null); // 绘制缩小后的图  
            g.dispose();
            src = image;
        }
        // 插入LOGO  
        Graphics2D graph = source.createGraphics();
        int x = (QRCODE_SIZE - width) / 2;
        int y = (QRCODE_SIZE - height) / 2;
        graph.drawImage(src, x, y, width, height, null);
        Shape shape = new RoundRectangle2D.Float(x, y, width, width, 6, 6);
        graph.setStroke(new BasicStroke(3f));
        graph.draw(shape);
        graph.dispose();
    }

    /**
     * 生成二维码(内嵌LOGO) 
     * 二维码文件名随机，文件名可能会有重复 
     *
     * @param content
     *            内容 
     * @param logoPath
     *            LOGO地址 
     * @param destPath
     *            存放目录 
     * @param needCompress
     *            是否压缩LOGO 
     * @throws Exception
     */
    public static String encode(String content, String logoPath, String destPath, boolean needCompress) throws Exception {
        BufferedImage image = QrCodeUtils2.createImage(content, logoPath, needCompress);
        mkdirs(destPath);
        String fileName = new Random().nextInt(99999999) + "." + FORMAT.toLowerCase();
        ImageIO.write(image, FORMAT, new File(destPath + "/" + fileName));
        return fileName;
    }

    /**
     * 生成二维码(内嵌LOGO) 
     * 调用者指定二维码文件名 
     *
     * @param content
     *            内容 
     * @param logoPath
     *            LOGO地址 
     * @param destPath
     *            存放目录 
     * @param fileName
     *            二维码文件名 
     * @param needCompress
     *            是否压缩LOGO 
     * @throws Exception
     */
    public static String encode(String content, String logoPath, String destPath, String fileName, boolean needCompress) throws Exception {
        BufferedImage image = QrCodeUtils2.createImage(content, logoPath, needCompress);
        mkdirs(destPath);
        fileName = fileName.substring(0, fileName.indexOf(".")>0?fileName.indexOf("."):fileName.length())
                + "." + FORMAT.toLowerCase();
        ImageIO.write(image, FORMAT, new File(destPath + "/" + fileName));
        return fileName;
    }

    /**
     * 当文件夹不存在时，mkdirs会自动创建多层目录，区别于mkdir． 
     * (mkdir如果父目录不存在则会抛出异常) 
     * @param destPath
     *            存放目录 
     */
    public static void mkdirs(String destPath) {
        File file = new File(destPath);
        if (!file.exists() && !file.isDirectory()) {
            file.mkdirs();
        }
    }

    /**
     * 生成二维码(内嵌LOGO) 
     *
     * @param content
     *            内容 
     * @param logoPath
     *            LOGO地址 
     * @param destPath
     *            存储地址 
     * @throws Exception
     */
    public static String encode(String content, String logoPath, String destPath) throws Exception {
        return QrCodeUtils2.encode(content, logoPath, destPath, false);
    }

    /**
     * 生成二维码 
     *
     * @param content
     *            内容 
     * @param destPath
     *            存储地址 
     * @param needCompress
     *            是否压缩LOGO 
     * @throws Exception
     */
    public static String encode(String content, String destPath, boolean needCompress) throws Exception {
        return QrCodeUtils2.encode(content, null, destPath, needCompress);
    }

    /**
     * 生成二维码 
     *
     * @param content
     *            内容 
     * @param destPath
     *            存储地址 
     * @throws Exception
     */
    public static String encode(String content, String destPath) throws Exception {
        return QrCodeUtils2.encode(content, null, destPath, false);
    }

    /**
     * 生成二维码(内嵌LOGO) 
     *
     * @param content
     *            内容 
     * @param logoPath
     *            LOGO地址 
     * @param output
     *            输出流 
     * @param needCompress
     *            是否压缩LOGO 
     * @throws Exception
     */
    public static void encode(String content, String logoPath, OutputStream output, boolean needCompress)
            throws Exception {
        BufferedImage image = QrCodeUtils2.createImage(content, logoPath, needCompress);
        ImageIO.write(image, FORMAT, output);
    }

    /**
     * 生成二维码 
     *
     * @param content
     *            内容 
     * @param output
     *            输出流 
     * @throws Exception
     */
    public static void encode(String content, OutputStream output) throws Exception {
        QrCodeUtils2.encode(content, null, output, false);
    }

    /**
     * 解析二维码 
     *
     * @param file
     *            二维码图片 
     * @return
     * @throws Exception
     */
    public static String decode(File file) throws Exception {
        BufferedImage image;
        image = ImageIO.read(file);
        if (image == null) {
            return null;
        }
        BufferedImageLuminanceSource source = new BufferedImageLuminanceSource(image);
        BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
        Result result;
        Hashtable<DecodeHintType, Object> hints = new Hashtable<DecodeHintType, Object>();
        hints.put(DecodeHintType.CHARACTER_SET, CHARSET);
        result = new MultiFormatReader().decode(bitmap, hints);
        String resultStr = result.getText();
        return resultStr;
    }

    /**
     * 解析二维码 
     *
     * @param path
     *            二维码图片地址 
     * @return
     * @throws Exception
     */
    public static String decode(String path) throws Exception {
        return QrCodeUtils2.decode(new File(path));
    }

    public static void main(String[] args) throws Exception {
        String text = "https://blog.csdn.net/ianly123";
        //不含Logo  
        //QrCodeUtils2.encode(text, null, "/Users/ianly/Documents/picture", true);
        //含Logo，不指定二维码图片名  
        //QrCodeUtils2.encode(text, "/Users/ianly/Documents/picture/google-icon.jpg", "/Users/ianly/Documents/picture/", true);
        //含Logo，指定二维码图片名  
        QrCodeUtils2.encode(text, "/Users/ianly/Documents/picture/google-icon.jpg", "/Users/ianly/Documents/picture", "qrcode", true);
    }
}
```

